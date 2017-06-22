<h2 id="section-title">OpenVidu Documentation</h2>
<hr>


What is OpenVidu?
========

OpenVidu is a platform to facilitate the addition of video calls in your web or mobile 
application, either group or one-to-one calls. In fact, any combination you come up with is easy to implement with OpenVidu.

It is based on [Kurento](http://www.kurento.org), the WebRTC platform for multimedia applications. Openvidu was forked from [KurentoRoom project](https://github.com/Kurento/kurento-room).

OpenVidu and Kurento are licensed under Apache License v2.

----------


Table of contents
========

* [Running a videocall demo](#running-a-videocall-demo-application)
* [Building a simple app](#building-a-simple-app-with-openvidu)
* [Securization](#securization)
* [Sharing data between users](#sharing-data-between-users)
* [Acknowledgments](#acknowledgments)

----------


Running a videocall demo application
====================================
We have implemented a very basic demo application to see OpenVidu in action. To ease the installation, we have packaged it as a docker image. 

 - Please be sure that you have [docker CE installed](https://store.docker.com/search?type=edition&offering=community)
 - Run this Docker container
   
```
docker run -p 5000:5000 -p 4040:4040 -e KMS_STUN_IP=193.147.51.12 -e KMS_STUN_PORT=3478 -e openvidu.security=false openvidu/openvidu-plainjs-demo
```
   
 - Wait until you see a public URL ended with `.ngrok.io`. You can connect locally in [`localhost:5000`](http://localhost:5000) or by using the ngrok public URL. You can also share this URL with anyone you want to test the app over the Internet!
 
----------

Building a simple app with OpenVidu
===================

<p align="center">
  <img src="https://docs.google.com/uc?id=0B61cQ4sbhmWSNF9ZWHREUXo3QlE">
</p>

OpenVidu has a traditional **Client - Server** architecture built on three modules that are shown in the image above. To run **openvidu-server** and **Kurento Media Server** you can execute the following container: 

```
docker run -p 8443:8443 --rm -e KMS_STUN_IP=193.147.51.12 -e KMS_STUN_PORT=3478 -e openvidu.security=false openvidu/openvidu-server-kms
```
 
 
Then, you have to use the library **openvidu-browser** in your JavaScript browser application (frontend). This library is packaged in [OpenVidu.js] file that you can download from https://github.com/OpenVidu/openvidu/blob/master/openvidu-browser/src/main/resources/static/js/OpenVidu.js. Then add the file in your HTML with `<script src="OpenVidu.js"></script>`.

With the **openvidu-browser** library you can handle all available operations straight away from your client, as creating video calls, joining users to them or publishing/unpublishing video and audio


## Sample application


Once you have up and running Kurento Media Server and openvidu-server, you just need to add a few lines of code in your frontend to make your first video call with OpenVidu. You can take a look to the simplest sample application in GitHub https://github.com/OpenVidu/openvidu-tutorials/tree/master/openvidu-insecure-js.

You can clone the repo and serve the app locally with your favourite tool (we recommend http-server: `npm install -g http-server`)

```
git clone https://github.com/OpenVidu/openvidu-tutorials.git
cd openvidu-tutorials/openvidu-insecure-js/web
http-server
```
You can now start editing HTML, JS and CSS files. Just reload your browser to see your changes (mind the browser's cache!).

### Code description


1. Get an *OpenVidu* object and initialize a session with a *sessionId*. Have in mind that this is the parameter that defines which video call to connect.

```javascript
var OV = new OpenVidu("wss://" + OPENVIDU_SERVER_IP + ":8443/");
var session = OV.initSession(sessionId);
```
	
2. Set the events to be listened by your session. For example, this snippet below will automatically append the new participants videos to HTML element with 'subscriber' id. Available events for the Session object are detailed in [API section](#session).

```javascript
session.on('streamCreated', function (event) {
    session.subscribe(event.stream, 'subscriber');
});
```
3. Connect to the session. For a non-secure approach, the value of *token* parameter is irrelevant. You can pass as second parameter a callback to be executed after connection is stablished. A common use-case for users that want to stream their own video is the following one: if the connection to the session has been succesful, get a Publisher object (appended to HTML element with id 'publisher') and publish it. The rest of participants will receive the stream.

```javascript
session.connect(token, function (error) {
    // If connection successful, get a publisher and publish to the session
    if (!error) {
        var publisher = OV.initPublisher('publisher', {
            audio: true,
            video: true,
            quality: 'MEDIUM' //'LOW','MEDIUM','HIGH'
        });
        session.publish(publisher);
    } else {
        console.log('Error while connecting to the session');
    }
});
```
4. Finally, whenever you want to leave the video call...

```javascript
session.disconnect();
```

With these few lines of code you will already have a functional video-call capability in your app. Check [Securization](#securization) section to learn how to easily make your app ready for production.

If you prefer, there's an Angular version of the sample app that uses _openvidu-browser_ npm package. Check it out [here](https://github.com/OpenVidu/openvidu-tutorials/tree/master/openvidu-insecure-angular).

----------

Securization
===================


## Why?


In a production environment probably you don't want unauthorized users swamping your video calls. It's not possible to control access to them with the first approach we have seen in the sections above: anyone who knows the _sessionId_ could connect to your video call, and if it turns out that the _sessionId_ doesn't belong to an existing session, a new one would be created.

In addition, a secure version also means you can choose the role each user has in your video calls (see [OpenViduRole](#openvidurole) section).

Thus, a non-secure version of OpenVidu is only intended for development environments. Don't worry, adding securization is not a difficult task.

## How?


<p align="center">
  <img src="https://docs.google.com/uc?id=0B61cQ4sbhmWSeDNIekd5R2ZhQUE">
</p>

In the image above you can see the main difference with the non-secure version of OpenVidu. Your backend will now have to call two HTTP REST operations in openvidu-server to get the two parameters needed in the securization process:

 - ***sessionId***: just as in the non-secure version, it identifies each specific video-call
 - ***token***: any user joining a specific video call will need to pass a valid token as a parameter

You have three different options available for getting sessionIds and tokens from openvidu-server:

 - [REST API](https://github.com/OpenVidu)
 - [openvidu-java-client](https://github.com/OpenVidu)
 - [openvidu-node-client](https://github.com/OpenVidu)

## A sequence diagram to sum up



<p align="center">
  <img src="http://www.plantuml.com/plantuml/png/ZP9TIyCm58QlcrznY3SJNFsus8KNmgPJAcKPRWWYkyYQT8HXKfBiu-URkcn4cxhUji_xdFSSOjP2LbJJBnYf_PGo9kGARcyGSX-jA4H5fGNyeJOQdhMI5l_-eIekju9j-akjTePhZ11QgZtW7vXBXk623ygxiaH9gp4vetGQSD9Ofn4jrcsLN7ORDAhHKw6I3sA53hhaVz-fJhW4z1z21zp3PqvUiWcGwVXjEC_8P76EVoKEVy-UnWGUXtc-G6cQ7eSSe3hpjuvBNg-udN5ZX98PGqsYCGgR8uqx-INVpPKxNYUthSccDzpTBHjKkFAHo84QRy55NHdms_O2ooMAqCt1Fj1jb8VJGad92zlpHLj7nMxdizy0">
</p>

 1. Identify your user and listen to a request for joining a video call (represented by [LOGIN OPERATION] and [JOIN VIDEO CALL] in the diagram). This process is entirely up to you.
 2. You must get a _sessionId_: a new one if the video call is being created or an existing one for an active video call. In the first case you need to ask openvidu-server for it (as shown in the diagram), in the second case you must retrieve it from wherever you stored it when it was created (a data-base or maybe your backend itself).
 3. You also need a new valid _token_ for this session. Ask openvidu-server for it passing the _sessionId_.
 4. Finally return both parameters to your frontend, where using openvidu-browser you may initilize your session with _sessionId_ and then connect to it with _token_. Good news: **the code is exactly the same as explained before in [Code description](#code-description) section**

> Communication between _Your Back_ and _openvidu-server_ modules is outlined in the diagram, but it does not correspond to the real methods. Remember you can handle this from your backend by consuming the [REST API](#rest-api) or by using [openvidu-backend-client](#openvidu-backend-client) package.


## Running a secure videocall application
We have implemented a very basic [demo application](https://github.com/OpenVidu/openvidu-tutorials/tree/master/openvidu-js-java) to see the secure version of OpenVidu in action. It has a Java backend to manage the user sessions and the securization process with OpenVidu Server.

 - Please be sure that you have [docker CE installed](https://store.docker.com/search?type=edition&offering=community)
 - Run this Docker container
   
```
docker run -p 5000:5000 -p 3000:3000 -p 4040:4040 -e KMS_STUN_IP=193.147.51.12 -e KMS_STUN_PORT=3478 openvidu/openvidu-sample-secure
```
 
 - Wait until you see a public URL ended with `.ngrok.io`. You can connect locally in [`localhost:3000`](http://localhost:3000) or by using the ngrok public URL. You can also share this URL with anyone you want to test the app over the Internet!
 


## Running a sample advanced app
Wanna try a [real sample application](https://github.com/OpenVidu/openvidu/tree/master/openvidu-sample-app) that makes use of everything we have talked about? Take a look at this app. It wraps a frontend built with Angular, a backend built with Spring and a MySQL database:

 - Please be sure that you have docker-compose (`sudo apt-get install docker-compose`)
 - Download the `docker-compose.yml` file and run it:
   
```
wget -O docker-compose.yml https://raw.githubusercontent.com/OpenVidu/openvidu-docker/master/openvidu-sample-app/docker-compose.yml
docker-compose up
```
 - Wait until you see an output like `Started App in XXX seconds (JVM running for XXX)`
   
 - Go to [`https://localhost:5000`](https://localhost:5000) and accept the self-signed certificate. Here you have a couple registered users (use a standard window and an incognito window to test both of them at the same time):

	| user                             | password |
	| -------------------------- | ----------- |
	| teacher@<span></span>gmail.com   | pass         |
	| student1@<span></span>gmail.com | pass         |

----------


Sharing data between users
===================
Whatever app you are developing, chances are you will need to pass some data for each user, at least a nickname. You can do it in two different places:

- **openvidu-browser**: when calling `session.connect` method
```
session.connect(token, DATA, function (error) { ... });
```

- **API REST**: when asking for a token to */api/tokens*, you can pass data as third parameter in the BODY of the POST request 
```
{“session”: “sessionId”, “role”: “role”, “data”: "DATA"}
```

> Java and Node clients (_openvidu-java-client_ and _openvidu-node-client_) allow you to pass data when creating a Token object: </br>
> `tokenOptions = new TokenOptions.Builder().data("DATA").build();`

The result will be that in all clients, *Connection* objects will have in their *data* property the pertinent value you have provided for each user. So, an easy way to get the data associated to any user would be:

```javascript
session.on('streamCreated', function (event) {
    session.subscribe(event.stream, 'subscriber');
    console.log('USER DATA: ' + event.stream.connection.data);
});
```

Some clarifications:

- *Connection.data* will be a simple string if you have provided data only with one of the methods, and will be a string with the following format if you provide data both from openvidu-browser and your backend: "OPENVIDUBROWSER_DATA%/%APIREST_DATA" 

- Using only first option is not secure, as clients could modify the value of the second parameter. It is intended only in development environments. If you want total control over shared data, please use the second way.
- You can choose whatever format you like for the data string, but if you are planning to share more than a simple field, maybe a standard format as JSON would be a wise choice.


Acknowledgments
===============
OpenVidu platform has been supported under project LERNIM (RTC-2016-4674-7) confunded by the _Ministry of Economy, Finance and Competitiveness_ of Spain, as well as by the _European Union_ FEDER, whose main goal with this funds is to promote technological development, innovation and high-quality research.

<p align="center">
  <img width="400px" src="https://docs.google.com/uc?id=0B61cQ4sbhmWSQzNLQnF4SnhFLWc">
</p>

<p align="center">
  <img width="400px" src="https://docs.google.com/uc?id=0B61cQ4sbhmWSa205YXNkSW9VNUE">
</p>

