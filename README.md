# Firebase NodeJS

# Requirements

- NVM
- NodeJS 8.16.0
- Firebase Tools

## Local Setup

### Node Version

Make sure you have the correct version of node installed because firebase only supports specific versions of node and not the latest versions.

```bash
nvm install;
```

### Firebase Tools

Make sure you have the firebase tools installed on your computer with the correct node version.

```bash
yarn add firebase-tools; # npm install firebase-tools
```

### Initial Firebase Setup From Scratch

These are the steps you would take if you were setting your project from scratch. If the project is already setup, please ignore these steps and proceed to `### Existing Firebase Setup`

1. Make sure you are logged into your google account:

```bash
./node_modules/firebase-tools/lib/bin/firebase.js login;
```

2. Init or select your project

```bash
./node_modules/firebase-tools/lib/bin/firebase.js init;
```

3. When the prompt appears, select `Functions`

```
Which Firebase CLI features do you want to set up for this folder? Press Space to select features, then Enter
 to confirm your choices.
 ◯ Database: Deploy Firebase Realtime Database Rules
 ◯ Firestore: Deploy rules and create indexes for Firestore
❯◉ Functions: Configure and deploy Cloud Functions
 ◯ Hosting: Configure and deploy Firebase Hosting sites
 ◯ Storage: Deploy Cloud Storage security rules
```

4. You'll need to select an existing project:

```
Select a default Firebase project for this directory:
  [don't setup a default project]
❯ yourproject (your-project)
  [create a new project]
```

5. Select `JavasScript` for this tutorial:

```
What language would you like to use to write Cloud Functions? (Use arrow keys)
❯ JavaScript
  TypeScript
```

6. Enable `ESLint`:

```
Do you want to use ESLint to catch probable bugs and enforce style? (y/N) y
```

7. Install dependencies:

```
Do you want to install dependencies with npm now? (Y/n) y
```

8. Adding Service Account Key:

**_NOTE_:** Make you sure only perform this step once and just refer to the existing generated file otherwise every time you generate a new key, every one else's code will **NOT WORK**.

A. In your firebase console in the browser, next to `Project Overview`, click the `⚙` icon and go to `Project Settings`.

B. In the top navigation, go to `Service Accounts`

C. Under `Firebase Admin SDK`, make sure `Node.js` is selected and click **Generate new private key**

D. Place the downloaded file and rename it as `functions/config/serviceAccountKey.json`

9. Adding Config File:

A. In your firebase console in the browser, next to `Project Overview`, click the `⚙` icon and go to `Project Settings`.

B. In the top navigation, go to `General`

C. Scroll down to section `Your Apps` > `Web apps` and make sure your project is selected.

D. Under `Firebase SDK snippet` make sure `Config` is selected and copy the scripts:

```javascript
const firebaseConfig = {
  apiKey: '{api-key}',
  authDomain: '{auth-domain}',
  databaseURL: '{database-url}',
  projectId: '{project-id}',
  storageBucket: '{storage-bucket}',
  messagingSenderId: '{messaging-sender-id}',
  appId: '{app-id}'
};
```

E. Create a new file called `functions/config/config.json`, paste and modify the code as:

```json
{
  "apiKey": "{api-key}",
  "authDomain": "{auth-domain}",
  "databaseURL": "{database-url}",
  "projectId": "{project-id}",
  "storageBucket": "{storage-bucket}",
  "messagingSenderId": "{messaging-sender-id}",
  "appId": "{app-id}"
}
```

### Existing Firebase Setup

1. Configure `.firebaserc`

```bash
cp .firebaserc.example .firebaserc;
```

2. Make sure you are logged into your google account:

```bash
./node_modules/firebase-tools/lib/bin/firebase.js login;
```

3. Set project

```json
{
  "projects": {
    "default": "your-project"
  }
}
```

4. Configure your `config.json` file:

Copy and modify accordingly with the right crentials.

```bash
cp functions/config/config.example.json functions/config/config.json;
```

5. Configure your `serviceAccountKey.json` file:

Copy and modify accordingly with the right crentials.

```bash
cp functions/config/serviceAccountKey.example.json functions/serviceAccountKey/config.json;
```

_*NOTE*:_ If you get the following error, make sure to do `nvm install` in the root to install the correct node version:

```bash
error functions@: The engine "node" is incompatible with this module. Expected version "8". Got "10.15.3"
error Found incompatible module.
```

6. Install Root Dependencies

```bash
yarn install; # npm install
```

7. Install Functions Dependencies

```bash
cd functions;
yarn install; # npm install
```

## Creating Local Server Start

Firebase by default uploads the code to be compiled on Google's servers, which takes time. There is a way around this, where we'll add a local development script in our `package.json`:

Add the following: `"local-start": "PORT=5000 node src/index.js",`

`functions/package.json`

```json
  "scripts": {
    "lint": "eslint .",
    "serve": "firebase serve --only functions",
    "shell": "firebase functions:shell",
    "start": "npm run shell",
    "local-start": "PORT=5000 node src/index.js",
    "deploy": "firebase deploy --only functions",
    "logs": "firebase functions:log"
  },
```

## Re-Structuring Our Functions Folder

To keep things organized we're going create a new folder called `functions/src` where most of our code will live.

In there we'll create our base `functions/src/index.js` which will also reference our other resources.

In the `functions/index.js` file, we'll modify it from:

```javascript
const functions = require('firebase-functions');

// // Create and Deploy Your First Cloud Functions
// // https://firebase.google.com/docs/functions/write-firebase-functions
//
// exports.helloWorld = functions.https.onRequest((request, response) => {
//  response.send("Hello from Firebase!");
// });
```

and change it to:

```javascript
// Dependencies
const functions = require('firebase-functions');
const app = require('./src');

// Export
exports.app = functions.https.onRequest(app);
```

Now we can run our server locally on `http://localhost:5000` by running:

```bash
cd functions;
yarn run local-start;
```

When we go to `http://localhost:5000` we should see:

```json
{ "VERSION": "1.0.0", "HOST": "0.0.0.0", "PORT": "5000", "ENV": "development" }
```

## Testing Endpoint

To test one of the new routes, use this curl command in another terminal window:

```bash
curl -X POST \
  http://localhost:5000/api/posts \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
	"comment": "Hello first comment 👋."
}'
```

## Deployment

**_NOTE_**: You will need to be logged in to `firebase-tools` for this to work, and make sure you are set the correct `project` as well.

**_IMPORTANT_**: Make sure you environment keys / variables are set for production before pushing.

**_NODE_VERSION_**: Make sure you are using node the correct node version (run `nvm use`);

**_LINTING_ERRORS_**: Make sure to fix your linting errors, otherwise it will NOT deploy.

**_DEPLOYMENT_VERSIONING_**: Make sure to increment the deployment version in `functions/src/index.js` to make sure you have an idea if the new code has been deployed.

To deploy, run:

```bash
# !!! Make sure you're in the root of the project
./node_modules/firebase-tools/lib/bin/firebase.js deploy --only functions;
```

If successfully deployed, it should be deployed to:

`https://{your-firebase-project}.cloudfunctions.net/app/api`

If it was successfully deployed, but it's NOT showing, check:

`https://console.firebase.google.com/u/0/project/{your-firebase-project}/functions/logs?severity=DEBUG`
