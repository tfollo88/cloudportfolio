# cloudportfolio
Real-time Chat App with Firebase and Google Cloud
Codelab: Build a real-time chat web app using Firebase and Cloud Run

🔧 Tech Stack:
Firebase Authentication

Firestore

Cloud Run (optional backend logic)

Cloud Build (CI/CD for deployments)

💡 What You’ll Learn:
Build modern web apps with Firebase

Add auth and database in minutes

Optional backend on Cloud Run

Serve dynamic content and host microservices with Cloud Run 

bookmark_border

Firebase Hosting currently does not support integration with Cloud Run for Anthos.
Pair Cloud Run with Firebase Hosting to generate and serve your dynamic content or build REST APIs as microservices.

Using Cloud Run, you can deploy an application packaged in a container image. Then, using Firebase Hosting, you can direct HTTPS requests to trigger your containerized app.

Cloud Run supports several languages (including Go, Node.js, Python, and Java), giving you the flexibility to use the programming language and framework of your choice.
Cloud Run automatically and horizontally scales your container image to handle the received requests, then scales down when demand decreases.
You only pay for the CPU, memory, and networking consumed during request handling.
For example use cases and samples for Cloud Run integrated with Firebase Hosting, visit our serverless overview.


This guide shows you how to:

Write a simple Hello World application
Containerize an app and upload it to Artifact Registry
Deploy the container image to Cloud Run
Direct Hosting requests to your containerized app
Note that to improve the performance of serving dynamic content, you can optionally tune your cache settings.

Before you begin
Before using Cloud Run, you need to complete some initial tasks, including setting up a Cloud Billing account, enabling the Cloud Run API, and installing the gcloud command line tool.

Set up billing for your project
Cloud Run offers free usage quota, but you still must have a Cloud Billing account associated with your Firebase project to use or try out Cloud Run.

If your Firebase project is on the Spark pricing plan, and you associate your Firebase project with a Cloud Billing account, then your Firebase project is automatically upgraded to the Blaze pricing plan. Review the Firebase pricing page for a comparison of the Spark and Blaze plans. Also, make sure to review Cloud Run pricing and its quotas and limits.
Note: Every Firebase project is also a Google Cloud project.
Visit Understand Firebase Projects to learn more about the Firebase and Google Cloud project relationship.
Enable the API and install the SDK
Enable the Cloud Run API in the Google APIs console:

Open the Cloud Run API page in the Google APIs console.

When prompted, select your Firebase project.

Click Enable on the Cloud Run API page.

Install and initialize the Cloud SDK.

Check that the gcloud tool is configured for the correct project:


gcloud config list
Step 1: Write the sample application
Note that Cloud Run supports many other languages in addition to the languages shown in the following sample.

Go
Node.js
Python
Java
Create a new directory named helloworld-go, then change directory into it:


mkdir helloworld-go

cd helloworld-go
Create a new file named helloworld.go, then add the following code:


package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func handler(w http.ResponseWriter, r *http.Request) {
	log.Print("helloworld: received a request")
	target := os.Getenv("TARGET")
	if target == "" {
		target = "World"
	}
	fmt.Fprintf(w, "Hello %s!\n", target)
}

func main() {
	log.Print("helloworld: starting server...")

	http.HandleFunc("/", handler)

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}

	log.Printf("helloworld: listening on port %s", port)
	log.Fatal(http.ListenAndServe(fmt.Sprintf(":%s", port), nil))
}
This code creates a basic web server that listens on the port defined by the PORT environment variable.

Your app is finished and ready to be containerized and uploaded to Artifact Registry.

Step 2: Containerize an app and upload it to Artifact Registry
Containerize the sample app by creating a new file named Dockerfile in the same directory as the source files. Copy the following content into your file.

Go
Node.js
Python
Java


# Use the official Golang image to create a build artifact.
# This is based on Debian and sets the GOPATH to /go.
FROM golang:latest AS builder

ARG TARGETOS
ARG TARGETARCH

# Create and change to the app directory.
WORKDIR /app

# Copy local code to the container image.
COPY . ./

# Install dependencies and tidy up the go.mod and go.sum files.
RUN go mod tidy

# Build the binary.
# -mod=readonly ensures immutable go.mod and go.sum in container builds.
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -mod=readonly -v -o server

# Use the official Alpine image for a lean production container.
# https://hub.docker.com/_/alpine
# https://docs.docker.com/develop/develop-images/multistage-build/#use-multi-stage-builds
FROM alpine:3
RUN apk add --no-cache ca-certificates

# Copy the binary to the production image from the builder stage.
COPY --from=builder /app/server /server

# Run the web service on container startup.
CMD ["/server"]
Build your container image using Cloud Build by running the following command from the directory containing your Dockerfile:


gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
Upon success, you will see a SUCCESS message containing the image name
(gcr.io/PROJECT_ID/helloworld).

The container image is now stored in Artifact Registry and can be re-used if desired.

Note that, instead of Cloud Build, you can use a locally installed version of Docker to build your container locally.

Step 3: Deploy the container image to Cloud Run
Allowed Cloud Run regions

Deploy using the following command:


gcloud run deploy --image gcr.io/PROJECT_ID/helloworld
When prompted:

Select a region (for example us-central1)
Confirm the service name (for example, helloworld)
Respond Y to allow unauthenticated invocations
Important: Remember the region and the service name so that you can use these values when defining the region and serviceID, respectively, in the firebase.json rewrite rule later.
Wait a few moments for the deploy to complete. On success, the command line displays the service URL. For example: https://helloworld-RANDOM_HASH-us-central1.a.run.app

Visit your deployed container by opening the service URL in a web browser.

Note: Consider applying a maximum instance limit to prevent unexpected scaling of the Cloud Run service.
The next step walks you through how to access this containerized app from a Firebase Hosting URL so that it can generate dynamic content for your Firebase-hosted site.

Step 4: Direct hosting requests to your containerized app
With rewrite rules, you can direct requests that match specific patterns to a single destination.

The following example shows how to direct all requests from the page /helloworld on your Hosting site to trigger the startup and running of your helloworld container instance.

Make sure that:

You have the latest version of the Firebase CLI.

You have initialized Firebase Hosting.

For detailed instructions about installing the CLI and initializing Hosting, see the Get Started guide for Hosting.

Open your firebase.json file.

Add the following rewrite configuration under the hosting section:


"hosting": {
  // ...

  // Add the "rewrites" attribute within "hosting"
  "rewrites": [ {
    "source": "/helloworld",
    "run": {
      "serviceId": "helloworld",  // "service name" (from when you deployed the container image)
      "region": "us-central1",    // optional (if omitted, default is us-central1)
      "pinTag": true              // optional (see note below)
    }
  } ]
}
Deploy your hosting configuration to your site by running the following command from the root of your project directory:


firebase deploy --only hosting
How pinTag works within the run block

Your container is now reachable via the following URLs:

Your Firebase subdomains:
PROJECT_ID.web.app/ and PROJECT_ID.firebaseapp.com/

Any connected custom domains:
CUSTOM_DOMAIN/

Note: Firebase Hosting is subject to a 60-second request timeout. If your app requires more than 60 seconds to run, you'll receive an HTTPS status code 504 (request timeout). To support dynamic content that requires longer compute time, consider using an App Engine flexible environment.
Visit the Hosting configuration page for more details about rewrite rules. You can also learn about the priority order of responses for various Hosting configurations.

Test locally
During development, you can run and test your container image locally. For detailed instructions, visit the Cloud Run documentation.

Next steps
Set up caching of your dynamic content on a global CDN.

Interact with other Firebase services using the Firebase Admin SDK.

Learn more about Cloud Run, including detailed how-to guides for setting up, managing, and configuring containers.

Review the pricing and the quotas and limits for Cloud Run.

Was this helpful?

