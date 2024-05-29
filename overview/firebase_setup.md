# <ins>Google Firebase Setup</ins>

## 1. Introduction and Overview

`Google Firebase` is an application development platform that provides various cloud-based services to simplify development of web apps.

The main feature we'll be using is `Firebase Cloud Firestore`, which is a real-time cloud database with a robust set of developer tools.

Cloud Firestore allows us to build web applications in a "serverless" architecture.

"Serverless" may sound like a bit of a misnomer here, as there are still plenty of web servers involved, but the name is based on the fact that web app developers do not need to deploy or configure web servers themselves.

Cloud Firestore essentially serves as a convenient, secure backend database for psyanim-2 users & developers, so we can persist data from our web-based experiments without needing to worry about the configuration or deployment of web servers.

To use Firebase Firestore in your psyanim-2 experiments, there are several tasks related to Firebase setup that you'll need to complete:

- Setup a Firebase Cloud Project
- Setup a Firestore Database in your Firebase Cloud Project
- Add an app that can access your Firebase resources
- Install `Firebase CLI` Tools
- Setup a Google Service Account so any app can access your Google Cloud Resources

Aside from having `Firebase CLI` installed and authenticated with your Google account, here are two files we'll need at the end of this process:

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

In this section, we'll create our `firebase.config.json` file for use in our `psyanim-2` projects by copying the `firebase config` object from the `Firebase Console` page for our app.

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

## 5. Firestore Database Structure

<p align="center">
    <a href="https://www.youtube.com/watch?v=v_hR4K4auoQ" target="_blank">
      <img src="https://img.youtube.com/vi/v_hR4K4auoQ/0.jpg" alt="Firebase NoSQL Video"/>
    </a>
</p>

[`Cloud Firestore`](https://firebase.google.com/docs/firestore/data-model) is a NoSQL, document-oriented database.  Basically, the database is composed of any number of `collections` of `documents`, where each document is a set of key-value pairs.

---

Psyanim-2 adds a *very* thin abstraction layer to this, where a `Psyanim-2 database` is also a collection of `documents`, but each document is a [JSON object](https://en.wikipedia.org/wiki/JSON#Data_types).

The distinction between the `Firestore` database structure and the `Psyanim-2` database structure is very subtle, maybe even trivial in most cases.  So it's mostly OK to think of them as having the same structure.

However, this stricter definition of a `Psyanim-2 document` as a `JSON object` allows us to decouple `Psyanim-2` databases from `Google Firestore`, so they can be easily migrated to flat-files on the local disk, or even other NoSQL database services.

---

In `Cloud Firestore`, a document is uniquely identified within the database by a `path`.  This path is composed of collection / subcollection names and document name related to a particular document.

The Firebase console is a powerful tool for exploring the structure of any Firestore database.

You can view, add, edit, delete, and query data in your database via a graphical user interface.

You can also view / edit the security rules for external clients accessing the database (only recommended for advanced users).

You can also monitor the database usage.

The most common use of the Firebase console for researchers will likely be viewing data and performing queries via the GUI.

Check out these [docs](https://firebase.google.com/docs/firestore/using-console) for more information on managing Cloud Firestore with the Firebase Console.

## 6. Firebase CLI Tools

Firebase comes with a collection of powerful tools, called `Firebase CLI`, that can be used via a command-line interface.

The tools have a broad and robust feature-set, allowing you to manage, view and deploy Firebase projects from the command-line on your development machine.

While the `Firebase-CLI` can be used to run queries against your Firestore database, update security rules, etc., in `Psyanim-2`, the most useful aspect of `Firebase CLI` is the ability to run an instance of a Firebase emulator locally for testing using the [Firebase Local Emulator Suite](https://firebase.google.com/docs/emulator-suite).

For more information about this capability, check out the [docs](https://firebase.google.com/docs/emulator-suite).

---

For now, let's just get the `Firebase CLI` installed so it's available for us to use during development.

The installation process is slightly different depending on what platform you are using.

To install the `Firebase CLI`, follow the instructions [here](https://firebase.google.com/docs/cli#install_the_firebase_cli).

Once installed, you'll need to 'login' with the Firebase CLI in order for it to have access to your Cloud Firebase project.

To do so, follow the steps [here](https://firebase.google.com/docs/cli#sign-in-test-cli).

Basically, running `firebase login` will open a web-page that allows you authenticate from your logged-in Google account, granting the `Firebase CLI` tool access to your Firebase Cloud Project.

## 7. Google Cloud Service Account Setup

> Detailed information on creating `service accounts` can be found in [Google Service Account Creation Docs](https://cloud.google.com/iam/docs/service-accounts-create), but we'll walk through the high-level process here.

In order to give `Psyanim Experiment Viewer` and `Psyanim-CLI` access to your `Firebase Cloud project`, you'll need to setup a `Google Service Account` and obtain a `key` file associated with that account.

An `IAM Service Account` is an account with Google that allows applications (in this case, `psyanim-2` apps such as `Psyanim Experiment Viewer` and `psyanim-CLI`) to use allowed Google web services via API calls with a simple authentication mechanism, all of which can be managed by an admin in the Google Cloud web interface.

You can read more about IAM service accounts [here](https://cloud.google.com/iam/docs/service-account-overview).

The first thing you'll need to do is enable the IAM APIs for your project.

Navigate to the following page [Create Service Accounts](https://cloud.google.com/iam/docs/service-accounts-create) and click the `Enable the API` button under the `Before you begin` section.

On the `Confirm Project` step on that page, make sure you have selected the Firebase project ID we just created, in the dropdown shown in the image below:

<p align="center" style="font-size: 12px;">
    <img src="./imgs/iam_enable_access_page.png"/>
</p>

NOTE: if you attempt to change the project by clicking the drop-down and selecting your new project by ID, and then you get an error on that page, just close it, return to the [Create Service Accounts](https://cloud.google.com/iam/docs/service-accounts-create) page, and click the `Enable the API` button again.

Once you've done this, navigate to [Google Cloud Console](https://console.cloud.google.com), click the `IAM and Admin` link, then click `Service Accounts` on the left, and verify that your project ID is showing in the dropdown on the top-left (just as in the previous image where we enabled the `IAM API`).

In the table showing `Service accounts for project [projectId]`, you should see a service account named `firebase-adminsdk`.

Click the 3 dots under the `Actions` column for this service account and click `Manage Keys` in the drop-down menu.

On the next page, click the `ADD KEY` button and select `Create new key` from the drop-down menu.

In the dialog box that comes up, make sure `JSON` is selcted and click `CREATE`.

You should see a pop-up that says, `Private key saved to your computer` and a file download should start.

This file is your service account key.  Keep it in a safe place, as it can allow anyone to access / modify your Google account via the web APIs.

Should you feel this key has been compromised, you can always return to this `IAM & Admin` web UI and delete any keys you no longer want around.

Make a copy of this key and call it `service-account.json`.  Keep this in a safe place too, for now.

This is what we will use in `Psyanim Experiment Viewer` project and anytime we use `Psyanim-CLI`'s Firebase query features.