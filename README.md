# Firebase in Python

Firebase is a back end platform that offers several services to aid in the development of apps and games, specially the ones that rely on server side infrastructure.

Some of its services can be accessed by using RESTful approaches. This repository contains detailed guides and [examples](./examples) explaining how to use those services in your `Python ` projects.

You won't need to download additional modules for these guides, all of them work only using the PYthon Standard Library.

## Firebase Auth
*Main guide: [Firebase Auth](./auth)*

This service allows you to securely authenticate users into your app. It uses Google Identity Toolkit to provide this service. Some of its key features are:

* Leverages the use of OAuth, saving time and effort.
* Authenticate with `Facebook`, `Google`, `Twitter`, `Email`, `Anonymous` and more.
* Generates an `auth_token` that can be used for secure operations against Firebase Storage and Firebase Database.

## Firebase Database
*Main guide: [Firebase Database](./database)*

This service allows you to save and retrieve text based data. Some of its key features are:

* Securely save and retrieve data using rules and Firebase Auth.
* Data is generated in JSON, making it lightweight and fast to load.
* Easy to model and understand data structures.
* Filter, organize and query the data.

## Firebase Storage
*Main guide: [Firebase Storage](./storage)*

This service allows you to upload and maintain all kinds of files, including images, sounds, videos and binaries. It uses Google Cloud Storage to provide this service. Some of its key features are:

* Securely save, retrieve and delete files using rules and Firebase Auth.
* Load end edit metadata from files.

## Getting Started

This guide assumes you want to use the 3 services in the same application, you will be able to use them with a free account.

Before you start coding you need to follow these steps to prepare your application for Firebase:

1. Create or open a project in the [Firebase Console](https://firebase.google.com)
2. You will be presented with 3 options for adding your app to `iOS`, `Android` or `Web`.
3. Click `Web`, a popup will appear with information about your project. Copy down your `apiKey` and `authDomain`.

From the `authDomain` we only need the id of the project, an example of an id is: `my-app-12345`.

You can read the guides in any order but it is recommended to start with the [Firebase Auth guide](./auth).

## FAQs

### **Which are the benefits of using these guides?**

You won't need to download any additional modules, everything works only using the Python Standard Library. This is useful for developers that are in restricted environments.

They are also a great way to understand how Firebase works behind the scenes.

Free and open source!

### **Why only Database, Auth and Storage, what about the other Firebase features?**

These guides are based on the JavaScript SDK and therefore have their same limitation of being web based only.
