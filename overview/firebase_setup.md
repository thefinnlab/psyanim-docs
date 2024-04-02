# <ins>Google Firebase Setup</ins>

## 1. Introduction and Overview

`Google Firebase` is an application development platform that provides various cloud-based services to simplify development of web apps.

The main feature we'll be using is `Firebase Cloud Firestore`, which is a real-time cloud database with a robust set of developer tools.

Cloud Firestore allows us to build web applications in a "serverless" architecture.

"Serverless" may sound like a bit of a misnomer here, as there are still plenty of web servers involved, but the name is based on the fact that web app developers do not need to deploy or configure web servers themselves.

Cloud Firestore essentially serves as a convenient, secure backend database for psyanim-2 users & developers, so we can persist data from our web-based experiments without needing to worry about the configuration or deployment of web servers.

To use Firebase Firestore in your psyanim-2 experiments, there are several tasks related to Firebase setup that you'll need to complete:

- Setup a Firebase Cloud Project
- Add an app that can access your Firebase resources
- Download a Firebase Config File for use in your experiment project
- Setup a Google Service Account
- Download a service account key file for use with psyanim-CLI and Experiment Viewer

There are two files we'll need at the end of this process:

- `firebase.config.json`: A file that contains the necessary info for your experiment app to access Firebase resources
- `service-account.json`: This file contains a key necessary for psyanim-CLI and Experiment Viewer to access Firebase resource

This tutorial will walk you through the basic setup of Firebase for your psyanim-2 experiments, but for more in-depth Firebase docs, visit: [Firebase Docs](https://firebase.google.com/docs).

## 2. Firebase project setup

To start, let's setup our Firebase Cloud Project.  Open a Chrome browser and navigate to the [Firebase Console](https://console.firebase.google.com).

If you are not already signed into your Google account, you'll have to do so first.

Next, you'll need to click the `Add Project` button.

Type in a name for your project and click `Continue`.

You'll be asked if you want to 'Enable Google Analytics for this project'.  I normally disable this, but feel free to leave it enabled it you prefer.

Click `Continue` and your Cloud Firebase project will be created (may take a minute or so).

From now on, when you go to the [Firebase Console](https://console.firebase.google.com), you'll see your project listed and can select it from there.

## 3. Firestore Database Setup

Navigate back to your [`Firebase Console`](https://console.firebase.google.com) page.

Once you select your project in `Firebase Console`, you should be taken to the `Project Overview` page.

On the left side-bar, under `Product Categories`, expand the `Build` tab and click on `Firebase Database`.

Click the `Create Database` button on the `Cloud Firestore` page.

If you are located in North America, the default settings should be fine ('Location' should be `nam5 (United States)`).  Click `Next` to proceed.

On the next page, `Secure Rules`, leave `Start in production mode` checked and click `Create`.

It will take a minute or so for your project's Firestore database to be setup and then you should be greeted by the `Panel View` of your database.

At the moment it is empty, so you won't see any data in the table view.

## 4. Adding an App to Firebase so your experiment can access Firestore

Navigate back to your [`Firebase Console`](https://console.firebase.google.com) page and select your project.

On the left side-bar, click on the `Project Overview` and you should see the words 'Get started by adding Firebase to your app' on that page.

Click on the `Web` button (graphic symbol is `</>`) above the words 'Add an app to get started'.

You'll be taken to a page to register your app for use with Firebase.

For the app nickname, pick any name you want, but I suggest keeping it simple.

Go ahead and check `Also setup Firebase Hosting for this app`, as this will be a useful way to test your experiment online before deployment and share it with others for testing.

Click `Register App` when you're ready.

In the next section on that page, `Add Firebase SDK`, we can ignore most of what's there except for 1 key object, the `firebaseConfig` object.

This is the object we need to convert to JSON and capture in a file named `firebase.config.json` for use in `psyanim-2`.

In the code block presented in the `Add Firebase SDK` section, you should see a `firebaseConfig` object with the following basic structure:

```js
// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyD1PuJUWvLpQMNMTpsenPflpC6oCXlRje8",
  authDomain: "myProject-470a0.firebaseapp.com",
  projectId: "myProject-470a0",
  storageBucket: "myProject-470a0.appspot.com",
  messagingSenderId: "9105152923176",
  appId: "1:9105152923176:web:b3fb722083f35115565440"
};
```

We will take this javascript object on the right-hand side of the assignment statement in the above code block and copy into a file named `firebase.config.json`, wrapping the keys in double-quotes, like the JSON below:

```js
{
  "apiKey": "AIzaSyD1PuJUWvLpQMNMTpsenPflpC6oCXlRje8",
  "authDomain": "myProject-470a0.firebaseapp.com",
  "projectId": "myProject-470a0",
  "storageBucket": "myProject-470a0.appspot.com",
  "messagingSenderId": "9105152923176",
  "appId": "1:9105152923176:web:b3fb722083f35115565440"
}
```

Keep this `firebase.config.json` file somewhere safe.  You will copy it into any psyanim-2 experiment projects you create to give those projects access to your Firebase Firestore database.

## TODO: walk through firebase firestore database structure (collections & docs)

[Managing Cloud Firestore with Firebase Console](https://firebase.google.com/docs/firestore/using-console)

## TODO: Firebase cli installation and login process

[Firebase CLI installation](https://firebase.google.com/docs/cli#install_the_firebase_cli)

## TODO: Firebase service-account.json setup

[Google Service Account Creation Docs](https://cloud.google.com/iam/docs/service-accounts-create)
