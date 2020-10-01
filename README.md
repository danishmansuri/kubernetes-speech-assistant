# Talk to an app that can create and manage your Kubernetes instances

>A Kubernetes Assistant that can manage your Kubernetes clusters on IBM Cloud through voice
this is very good performance 

Watch the [video](https://youtu.be/MTG4uqiTpCg) to check out a demo of the application.

Imagine being away from your computer when you realize you need to deploy a back end on a Kubernetes cluster that you have not yet provisioned. Knowing that provisioning a cluster could take several minutes after selecting the right configuration for your task, you visualize sitting in front of your computer staring idly at the screen, waiting for the cluster to be initialized and wishing that you could start work immediately.

Not in my world!

This developer code pattern demonstrates a Kubernetes speech assistant application for an Android mobile device. You can simply talk on your phone in natural language to provision, view, and manage your Kubernetes clusters without having to do it manually on the cloud interface.

The pattern showcases an Android app that mobile device users interact with and a Node.js back-end server that holds the application logic and talks to the IBM Cloud Kubernetes Service. The pattern demonstrates use of the IBM Identity and Access Management using the OpenID Connect specifications for a native application. It also showcases Watson Assistant to understand the natural language spoken by the mobile users, holding the context of the speech and converting the speakers’ intent into executable Kubernetes commands that are run on IBM Cloud.

In this code pattern, you learn the following skills:

* Create an Android application connected with the OpenID Connect specifications for IBM Identity and Access Management.
* Develop a Node.js server using Express.js, which interfaces with the Watson Assistant and the IBM Cloud Kubernetes Service.
* Set up Watson Assistant to create intents, entities, and a dialog flow.
* Convert speech to text and text to speech, natively for an Android application.
* Manage OpenID Connect authorization tokens on an Android application.
* Deploy a Node.js back-end server on the IBM Cloud Kubernetes Service.

> Note: This codebase has two branches:
> * Branch `master` showcases using IAM with API-KEY
> * Branch `using-iam-custom-ui` showcases using IAM with custom UI (This branch is not recommended for non-IBM developers as IAM currently doesn't support creating credentials for non-IBM developers)

# Architecture flow

<p align="center">
  <img src="docs/doc-images/arch-flow.png">
</p>

1. The user triggers a login into their IBM Cloud accounts on the Android App, if not previously setup.
2. They are redirected to the IBM Cloud login page on their phone browsers, using the IBM IAM OpenID Connect Protocol.
3. After successful authentication, a request is initiated to get a generic IAM token.
4. The request sends the Authorization Code retrieved from Step 2 to the IBM IAM OpenID Connect Protocol to get the generic IAM token.
5. A request is initiated to get the list of accounts associated with the cloud login.
6. The request sends the generic IAM access token to the IBM Account Management API to retrieve the list of accounts associated with the user's cloud login. The user selects an account from the list.
7. A request is initiated to get an IAM token for the selected account.
8. The request sends the generic IAM refresh token and the selected account ID to the IBM IAM OpenID Connect Protocol to retrieve an IAM token linked to the selected cloud account.
9. The IAM Authorization Token object is persisted on the user's device for future use, until expiration.
10. A request is initiated to get the list of resource groups for the selected account.
11. The request sends the account specific IAM access token to the IBM Resource Controller API to retrieve the list of resource groups associated with the account. The user selects a resource group from the list.
12. A request is initiated to get a Watson Assistant session.
13. The request talks to the Watson Assistant API SDK through the NodeJS server to get a new assitant session.
14. The Android application setup is completed and the view is changed so that the user can start talking with the applicaton.
15. The user sends a speech command to the application, by clicking the mic button and talking.
16. The speech is converted to text using the native Android speech-2-text converter.
17. A fresh IAM token request is issued using the previous IAM refresh token if the IAM access token has reached expiration.
18. The fresh IAM access token/non-expired IAM access token, along with the user text input is sent to the NodeJS backend server on hosted on a Kubernetes cluster on the IBM cloud.
19. The user text input is sent to the Watson Assistant to extract knowledge around the intent of the text and the different entities present.
20. If the context for the user input is complete and all knowledge required to execute the Kubernetes command is obtained, a request is sent to the Kubernetes service API on IBM Cloud along with the user IAM access token and other parameters, to execute the command.
21. The result of the executed command/a follow up question to get further details around the user request is sent to the Android application.
22. The text received by the application is converted to speech by using the native Android text-2-speech converter.
23. The speech is relayed to the user to continue the conversation.

# Included components
*	[IBM Watson Assistant](https://cloud.ibm.com/catalog/services/watson-assistant) Watson Assistant lets you build conversational interfaces into any application, device, or channel.
*	[IBM Cloud Kubernetes Service](https://www.ibm.com/cloud/container-service) gcreates a cluster of compute hosts and deploys highly available containers. A Kubernetes cluster lets you securely manage the resources that you need to quickly deploy, update, and scale applications.
* [IBM Identity and Access Management](https://www.ibm.com/security/identity-access-management) Explore silent identity and access management solutions for today's hybrid environments.

## Featured technologies
+ [Node.js](https://nodejs.org) is an open source, cross-platform JavaScript run-time environment that executes server-side JavaScript code.
+ [Express.js](https://expressjs.com/) is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.
+ [Axios](https://www.npmjs.com/package/axios) Promise based HTTP client for the browser and node.js.
+ [Android](https://developer.android.com/) Android is a mobile operating system developed by Google.
+ [AppAuth-Android](https://github.com/openid/AppAuth-Android) Android client SDK for communicating with OAuth 2.0 and OpenID Connect providers.
+ [Docker](https://www.docker.com/) independent container platform that enables organizations to seamlessly build, share and run any application, anywhere—from hybrid cloud to the edge.

## Running the application

Follow these steps to set up and run this code pattern. The steps are described in detail below.

### Prerequisites

- [IBM Cloud account](https://cloud.ibm.com/registration/?target=%2Fdashboard%2Fapps)
- [Node v8.x or greater and npm v5.x or greater](https://nodejs.org/en/download/)
- [Android Studio](https://developer.android.com/studio)

### Steps

1. [Clone the repo](#1-clone-the-repo)
2. [Setup IBM IAM](#2-setup-ibm-iam)
3. [Create IBM Cloud services](#3-create-ibm-cloud-services)
4. [Configure Watson Assistant](#4-configure-watson-assistant)
5. [Deploy NodeJS server to Kubernetes](#5-deploy-nodejs-server-to-kubernetes)
6. [Configure Android Application](#6-configure-android-application)
7. [Run the application](#7-run-the-application)


## 1. Clone the repo

Clone this repository in a folder your choice:

```bash
git clone https://github.com/IBM/kubernetes-speech-assistant.git
cd Kubernetes-speech-assistant
```

## 2. Setup IBM IAM

* Login into your `IBM Cloud` account.
* On the main dashboard, click `Manage` and under that `Access (IAM)`.
* On the left pane, click `IBM Cloud API keys`.
* Click on `Create an IBM Cloud API key`.
* GIve your key a name and click `create`.
* Copy the key.

<br>
<p align="center">
  <img src="docs/doc-gifs/iam-create-apikey.gif">
</p>
<br>

* Navigate to the file `app/src/main/java/com/example/kubernetesassistant/AppConfigTemplate.java` and paste the `API_KEY` value with the one copied.

<br>
<p align="center">
  <img src="docs/doc-gifs/iam-replace-apikey.gif">
</p>
<br>
 
* Rename `app/src/main/java/com/example/kubernetesassistant/AppConfigTemplate.java` to `app/src/main/java/com/example/kubernetesassistant/AppConfig.java`.

```bash
cd app/src/main/java/com/example/kubernetesassistant
mv AppConfigTemplate.java AppConfig.java
```

## 3. Create IBM Cloud services

* Create the [IBM Cloud Kubernetes Service](https://cloud.ibm.com/catalog/infrastructure/containers-kubernetes).  You can find the service in the `Catalog`.  For this code pattern, we can use the `Free` cluster, and give it a name.  Note, that the IBM Cloud allows one instance of a free cluster and expires after 30 days.

<br>
<p align="center">
  <img src="docs/doc-gifs/kubernetes-create.gif">
</p>
<br>

* Create the [IBM Watson Assistant Service](https://cloud.ibm.com/catalog/services/watson-assistant) service on the IBM Cloud.  You can find the service in the `Catalog`, and give a name.

<br>
<p align="center">
  <img src="docs/doc-gifs/assistant-create.gif">
</p>
<br>

## 4. Configure Watson Assistant

* Go to your provisioned Watson Assistant service on IBM Cloud. Create new credentials and enter them into the config.

<br>
<p align="center">
  <img src="docs/doc-gifs/assistant-credentials.gif">
</p>
<br>

* Click on `Launch Watson Assistant`.

<br>
<p align="center">
  <img src="docs/doc-gifs/launch-assistant.gif">
</p>
<br>

* Create a skill using the existing json in the repository.

<br>
<p align="center">
  <img src="docs/doc-gifs/create-assistant-skill.gif">
</p>
<br>

* Create an Assistant, and add the skill to the Assistant.

<br>
<p align="center">
  <img src="docs/doc-gifs/add-assistant-skill.gif">
</p>
<br>

* Get the Assistant ID and enter it into the config file.

<br>
<p align="center">
  <img src="docs/doc-gifs/assistant-id.gif">
</p>
<br>

* Rename `server/config-template.json` to `config.json`.

```bash
cd server
mv config-template.json config.json
```

## 5. Deploy NodeJS server to Kubernetes

* Navigate to the `server` directory in the cloned repository.

```bash
cd server
```

* Build the `Dockerfile` and push the image to your account on Dockerhub.

```bash
docker build -t <DOCKERHUB_USERNAME>/kubernetes-assistant .
docker push <DOCKERHUB_USERNAME>/kubernetes-assistant
```

* Modify the `manifest.yml` file (Line 15), to replace `<DOCKERHUB_USERNAME>` to your username.

* Go to your provisioned `Kubernetes` service on IBM Cloud, and run the commands displayed in the `access` tab.

<br>
<p align="center">
  <img src="docs/doc-images/kubernetes-connect.png">
</p>
<br>

* Deploy the `manifest.yml` on to your Kubernetes cluster.

```bash
kubectl apply -f manifest.yml
```

* Run the command `kubectl get svc`, and copy the external IP address.

## 6. Configure Android Application

* Open the codebase in `Android Studio` or an IDE of your choice.
* Navigate to the file `app/src/main/java/com/example/kubernetesassistant/AppConfig.java` and paste the `SERVER_URL` value with the IP address copied.

## 7. Run the application

Use `Android Studio` or the `CammandLine` to generate an APK and install the application on your Android Mobile device.
Run the application.

## Links
* [IBM Kubernetes REST Api](https://containers.cloud.ibm.com/swagger-api/#/)
* [IBM Code Patterns for Kubernetes](https://developer.ibm.com/patterns/category/containers/)

## License
This code pattern is licensed under the Apache Software License, Version 2. Separate third-party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](https://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](https://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
