# ng-conf 2019 workshop

## Things you need

* Node.js
* Google account
* Firebase Tools
* Google Cloud SDK

Note: If you use the Google Cloud Shell for this tutorial then you don't need to install anything locally.

## Getting Started

1.  Clone the repository:

        git clone git@github.com:jamesdaniels/onSnapshot.git \
          && cd onSnapshot

1.  Read about Firebase:

    https://firebase.google.com

1.  Read about Google Cloud Run:

    https://cloud.google.com/run

1.  Take a look at our demo Angular Universal application:

    https://github.com/jamesdaniels/onSnapshot

1.  Do the Cloud Run quickstart:

    https://cloud.google.com/run/docs/quickstarts/prebuilt-deploy

## Running our app

1.  Do the **Before you Begin** in the Google Cloud Source Repositories
    quickstart:

    https://cloud.google.com/source-repositories/docs/quickstart#before-you-begin

1.  Go to https://console.cloud.google.com and open the Cloud Shell.

1.  Create a new repository:

        gcloud source repos create onSnapshot

1.  Clone the new repository:

        gcloud source repos clone onSnapshot && cd onSnapshot

1.  Add https://github.com/jamesdaniels/onSnapshot as a remote:

        git remote add upstream https://github.com/jamesdaniels/onSnapshot

1.  Pull from `upstream`:

        git pull upstream

1.  Checkout `master`:

        git checkout master

1.  Push `master` to `origin`:

        git push origin master

1.  Install app dependencies:

        npm install

1.  Build the app:

        npm run build:all

1.  Run the app:

        npm run serve:ssr

1.  Change Web Preview port to 4200.

1.  Open the Web Preview.

    Note: The Service Worker takes over and we don't get server-side rendering.

1.  Try curling the preview from Mac (hint: it doesn't work).

1.  Open a 2nd Cloud Shell to curl the app:

        curl localhost:4200

    Note: We get a fully server-side rendered response.

1.  Stop the app.

## Deploying our app

1.  Do the 2nd Cloud Run quickstart, except use the onSnapshot code:

    https://cloud.google.com/run/docs/quickstarts/build-and-deploy

    Add this `.gcloudignore` file:

        dist
        node_modules
        public

        functions/node_modules
        functions/lib

    Add this `Dockerfile`:

        # Use the official Node.js 10 image.
        # https://hub.docker.com/_/node
        FROM node:10

        # Create and change to the app directory.
        WORKDIR /usr/src/app

        # Copy application dependency manifests to the container image.
        # A wildcard is used to ensure both package.json AND package-lock.json are copied.
        # Copying this separately prevents re-running npm install on every code change.
        COPY package.json package*.json ./

        # Install dependencies.
        RUN npm i

        # Copy local code to the container image.
        COPY . .

        # Build the application
        RUN npm run build:all

        # # Run the web service on container startup.
        CMD [ "npm", "run", "serve:ssr" ]

    Use these commands to deploy:

        gcloud builds submit --tag gcr.io/[PROJECT_ID]/universal
        gcloud beta run deploy --image gcr.io/[PROJECT-ID]/universal

    where `[PROJECT_ID]` is your Cloud project ID. Choose to allow
    unauthenticated requests.

1.  Open the URL printed to the terminal to see the app.

    Note: Again notice that the Service worker takes over and we don't get
    the server-side rendered HTML.

1.  Curl the app:

        curl [YOUR_APP_URL]

    Note: Notice that when curling you _do_ get the server-side rendered HTML.

## Deploying our app using Cloud Build

1.  Read https://cloud.google.com/run/docs/continuous-deployment.

1.  Add a `cloudbuild.yaml` file:

        steps:
        # build the base image using Kaniko for caching
        - name: 'gcr.io/kaniko-project/executor:latest'
          waitFor: ['-']
          id: DOCKER_BUILD
          args:
          - --destination=gcr.io/$PROJECT_ID/universal:$BUILD_ID
          - --cache=true

        # deploy the container to Cloud Run
        - name: 'gcr.io/cloud-builders/gcloud'
          args:
          - beta
          - run
          - deploy
          - universal
          - --image=gcr.io/$PROJECT_ID/universal:$BUILD_ID
          - --region=us-central1
          id: DEPLOY_CLOUD_RUN
          waitFor: [DOCKER_BUILD]

        images: ['gcr.io/$PROJECT_ID/universal']


    Deploy with the following command:

        gcloud builds submit --config cloudbuild.yaml .

1.  Setup a Build Trigger for automatic deployments:

    https://cloud.google.com/cloud-build/docs/running-builds/automate-builds

## Debugging the app

1.  Install the Google Stackdriver agents:

        npm i --save @google-cloud/trace-agent @google-cloud/debug-agent @google-cloud/profiler

1.  Add them to the top of `server.ts`:

        import * as TraceAgent from '@google-cloud/trace-agent';
        TraceAgent.start();

        import * as Profiler from '@google-cloud/profiler';
        Profiler.start();

        import * as DebugAgent from '@google-cloud/debug-agent';
        DebugAgent.start();

## Appendix

1.  Enable the Stackdriver Profiler API:

    https://console.cloud.google.com/apis/library/cloudprofiler.googleapis.com

1.  Enable the Stackdriver Error Reporting API:

    https://console.cloud.google.com/apis/library/clouderrorreporting.googleapis.com

1.  Enable the Cloud Build API

    https://console.cloud.google.com/apis/library/cloudbuild.googleapis.com
