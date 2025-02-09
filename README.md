<p align="center">
    <img src="./assets/logo.png" alt="logo" />
</p>

# Nest Camera SDK
![npm](https://img.shields.io/npm/dm/nest-cam)
![build](https://img.shields.io/travis/com/cbartram/Nest)
![npm bundle size](https://img.shields.io/bundlephobia/min/nest-cam)
[![SonarCloud Coverage](https://sonarcloud.io/api/project_badges/measure?project=cbartram_Nest-Cam&metric=coverage)](https://sonarcloud.io/component_measures/metric/coverage/list?id=cbartram_Nest-Cam)
![npm](https://img.shields.io/npm/v/nest-cam)


Access & Stream the Nest API Camera data through a Reactive Node.JS interface. 

In 2019 Google deprecated the Works With Nest program and shut down the Nest API's for new developers and pigeon hole'd the existing Nest developers
to migrate their Nest account's to Google. Google integrated the Nest products under the "Works With Google" umbrella and essentially made data access to your own
Nest devices a one way street.

You can give your Nest data to Google but you can't get it back out of the cloud. This Nest Camera SDK aims
to give developers the freedom and ability to access their own camera feed data in order to continue building great applications.

![nest denial letter](./assets/nest_denied.png)

## Quick Start

To get started with the Unofficial Nest Camera SDK you can install the package from [NPM](https://npmjs.com): 

```shell script
$ npm install nest-cam
```

### Examples

You can do a whole lot more with the Nest Camera SDK but here is a quick example to get you started:

```javascript
const Nest = require('nest-cam');

const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    clientId: "YOUR_CLIENT_ID",
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
});

nest.init().then(() => {
    nest.subscribe((eventStream) => {
        console.log('There was some motion or sound caught on the camera!', eventStream);
    });
    // ...or
    nest.getLatestSnapshot().then(image => {
        image.pipe(fs.createWriteStream('/My/Path/to/save/image_3.jpg'));
    });
    // ... or
    nest.subscribeSnapshot((image) => {
       image.pipe(fs.createWriteStream('/My/Path/to/save/image_3.jpg'));
    }, (error) => {
        console.log('Oh no there was an error!')
    })
});
```

### Prerequisites

Before you go hooking into your camera data you will need some security credentials. Check out the table below for a description
of each credential that is required.

| **Name**       | **Data Type** | **Description**                                                                                                                                                                                                                                                   |
|----------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `nestId`       | String        | A unique string which identifies your Nest Camera to the Google Nest API. This value will not change.                                                                                                                                                             |
| `refreshToken` | String        | Nest uses OAuth 2.0 to securely authenticate users. A refresh token allows an application to obtain a new OAuth access token without prompting the user for any information. This is a long lived token which will expire but not for an estimated several weeks. |
| `apiKey`       | String        | This value comes with special privileges to tell the Nest servers how often the client wielding the api key can call the backend. This value will not change.                                                                                                     |
| `clientId`     | String        | The Client Id is a unique string which is used by the Nest API to identify which client or application is calling the backend resources.                                                                                                                          |

Retrieving these values is a one time process than can be somewhat cumbersome. Below is an in depth tutorial of each step
with helpful images to guide you to the security credentials you need. The whole process should take less than 10 minutes! 

## Obtaining Nest Security Credentials

To get the security credentials you will have to setup a proxy on the same WiFi network as your Nest Camera. We will proxy requests from your Android or iPhone 
through the Nest app (and our proxy) to intercept the security credentials before they get to the Nest servers. Don't worry this is **way** easier than it sounds.
This tutorial is written for the Mac & iOS but works in a similar fashion on Windows and Android. Please consult the [Charles Documentation](https://www.charlesproxy.com/documentation/using-charles/ssl-certificates/) to
learn how to trust the SSL certificates on a Windows machine. 

### Installing Charles Proxy

First head to the [Charles Proxy Website](https://www.charlesproxy.com/) and download and install the proxy software. Next we are going to configure
Charles to trust all SSL certificates it proxies so that we can see the request's and response's that Nest uses to communicate with its API's (The credentials you are after are in these requests and responses). 

### Trusting the Root Charles SSL Certificate

Charles generates its own certificates for sites, which it signs using a Charles Root Certificate. The root certificate is uniquely generated for your installation of Charles (as of v3.10). In order
to avoid having to trust each website you visit individually you can simply trust the Charles Root Certificate one time and it will take care of everything else for you.

You need to trust the same certificate on your Mac/Windows and iPhone/Android

#### Mac 

In Charles go to the Help menu and choose "SSL Proxying > Install Charles Root Certificate".

 - Keychain Access will open. 
 - Find the "Charles Proxy..." entry, and double-click to get info on it. 
 - Expand the "Trust" section, and beside "When using this certificate" change it from "Use System Defaults" to "Always Trust". 
 - Then close the certificate info window, and you will be prompted for your Administrator password to update the system trust settings. 

![charles trust](./assets/charles_trust.png) 

#### iOS

In order to install the Charles Trust Certificate on iOS there are a couple simple steps.

Set your iOS device to use Charles as its HTTP proxy in the Settings app > Wifi settings. For the IP Address set the IP
address for the device that is running the charles proxy. This is usually something like: `192.168.1.153`. You can view your Mac's network
settings in System Preferences to locate the correct IP address. The port number will always be `8888`
and there is no Authentication.

- Open Settings
- Open Wifi
- Select the informational "I" next to the Wifi network that your Nest camera is connected to.
- Select Proxy Settings

<img alt="ios_proxy_one" src="./assets/ios_proxy_one.png" width="200" height="400" />

**Update your proxy**

<img alt="ios_proxy_two" src="./assets/ios_proxy_two.png" width="200" height="400" />


Open Safari and browse to [https://chls.pro/ssl](https://chls.pro/ssl). Safari will prompt you to install the SSL certificate.
If you are on iOS 10.3 or later, open the Settings.app and navigate to General > About > Certificate Trust Settings, and find the Charles Proxy certificate, 
and switch it on to enable full trust for it (More information about this change in iOS 10).

<img alt="ios_proxy_three" src="./assets/ios_cert_trust.png" width="200" height="400" />

Done!

### Capturing Nest Traffic

Now that you have your iOS/Android device configured to route traffic through the Charles Proxy you are ready to start intercepting Nest camera requests to
locate your credentials!

#### Nest ID

The Nest Id is easy to find. Fire up your Nest App on your phone and watch Charles Proxy capture the traffic! 

Make sure you:

- Set your Charles Proxy to use "Sequence". This ensures you see requests in the order they were sent
- Filter for the word "nexus". This will get rid of requests you dont care about (there will be a lot of them)

![nest_id_proxy](./assets/nest_id_proxy.png)

Your nest ID is highlighted in the picture above.

#### Refresh Token & Client Id

In Charles Proxy:

- Filter for "google"
- Look for the request from the host: `oauth2.googleapis.com`
- Select "Contents" (right beneath the filter bar) to view the contents of the Http Request

![refresh_client_id_charles](./assets/refresh_client_id_charles.png)

Your client id and refresh token are highlighted in the picture above. **Note:** If you cannot find this request
in Charles Proxy try signing out of Nest on your iPhone and then signing in again.

If your refresh token isn't displayed check the response body of the request and you should find it there!

#### API Key

Finally your API key can be located by:

- Filtering for "nestauthproxyservice"
- It is located in the "x-goog-api-key" Header value

![goog_api_key_nest](./assets/api_key_nest.png)

That's it your done!

## About the SDK

This section describes how to use the Nest SDK to retrieve images, fetch events, and stream data
from your Nest camera.

### Initializing the SDK

You **must** call the `init()` method after initializing the Nest SDK. The init method will fetch your OAuth access_token and JWT token
used to retrieve data from the Nest API.

### Events

The Nest SDK works around events. An event is logged by your Nest camera whenever it detects motion or sound. You can periodically
fetch Nest events through the API or subscribe to the event stream and be notified when a new event arrives.

The event object looks like this:

```javascript
 {
    "playback_time": 1586551361730,
    "start_time": 1586551361730,
    "camera_uuid": "<Your Nest Id>",
    "face_id": "",
    "is_important": false,
    "face_category": "UNSET",
    "end_time": 1586551371619,
    "importance": 0,
    "face_name": "",
    "in_progress": false,
    "id": "1586551361-labs",
    "zone_ids": [],
    "types": ["motion"]
}
```

### Streaming Events

The streaming interface for the Nest SDK is built on top of Observables using [RxJS](https://rxjs-dev.firebaseapp.com/). It uses [Observables](https://rxjs-dev.firebaseapp.com/guide/observable) as the fundamental unit to 
stream Nest camera data directly to your applications. Events are streamed using the `nest.subscribe()` method. You must specify the event type you wish to subscribe to either "event" or "snapshot".

You can poll for two different types of data:

- Snapshots - The latest snapshot image from the Nest camera
- Events - The latest JSON event from the Nest API 

Under the hood the streaming Observables will poll the Nest API at regular intervals for updates on new events (using you guessed it `getEvents()`)! You can configure how often
the observables poll the Nest API by passing values into the Nest constructor at start up. The values default to five seconds for snapshots and three seconds for events.

## Nest SDK Configuration

The Nest constructor takes one parameter. The parameter is an object which contains configuration parameters to tweak the Nest SDK's behavior. 
If no argument is provided the default options will be used. There are **four required properties** to the Nest options: Nest Id, Refresh Token, API Key, and client Id.
Please reference the documentation above if you need help finding your nest credentials. 

All other configuration options are documented below: 

| **Name**       | **Data Type** | **Description**                                                                                                                                                                                                                                                   |
|----------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `nestId`       | String        | A unique string which identifies your Nest Camera to the Google Nest API. This value will not change.                                                                                                                                                             |
| `refreshToken` | String        | Nest uses OAuth 2.0 to securely authenticate users. A refresh token allows an application to obtain a new OAuth access token without prompting the user for any information. This is a long lived token which will expire but not for an estimated several weeks. |
| `apiKey`       | String        | This value comes with special privileges to tell the Nest servers how often the client wielding the api key can call the backend. This value will not change.                                                                                                     |
| `clientId`     | String        | The Client Id is a unique string which is used by the Nest API to identify which client or application is calling the backend resources.                                                                                                                          |  
| `eventInterval`     | Integer       | The amount of time between polls to the Nest API when subscribing to events. A lower number will poll the API more often. Default is 3000. All values are in milliseconds                                                                                                                          |  
| `snapshotInterval`     | Integer       | The amount of time between polls to the Nest API when subscribing to snapshots. A lower number will poll the API more often. Default is 5000. All values are in milliseconds                                                                                                                         |  

```javascript
const Nest = require('nest-cam');

const options = {
    nestId: '...',
    clientId: '...',
    refreshToken: '...',
    apiKey: '...',
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
    eventInterval: 9000,
    snapshotInterval: 3000
};

let nest = new Nest(options);

// nest.init() etc...
```

## Examples

Here are some helpful examples to get you started.

### Fetching Camera Events

You can fetch all the latest events for your camera using the snippet below. You can also specify a start and end
time as parameters to the `getEvents()` method. For example `nest.getEvents('1586551371619', '1586551374320')`.

```javascript
const Nest = require('nest-cam');

const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    clientId: "YOUR_CLIENT_ID",
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
});

nest.init().then(() => {
    nest.getEvents().then(events => {
        ...
    }).catch(err => console.log('Failed to fetch events: ', err));
});
```

### Streaming Camera Events

```javascript
const Nest = require('nest-cam');

const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    clientId: "YOUR_CLIENT_ID",
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
});

nest.init().then(() => {
    nest.subscribe((event) => {
        ...
    })
});
```

You can also configure how often the Nest API is polled under the hood to publish new events to your subscription. All values 
are in milliseconds. A lower value will poll the Nest API more often and return new events faster but will also consume more computing
resources. We have found 3 seconds between each poll to be sufficient for most application use cases but feel free to tweak it to your needs.

```javascript
const Nest = require('nest-cam');

// This will poll subscriptions to snapshots every 2 seconds (instead of 5) and subscriptions to events
// every second (instead of 3 seconds).
const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
    clientId: "YOUR_CLIENT_ID",
    eventInterval: 1000,
    snapshotInterval: 2000
});

// ... 
```

### Fetching Latest Image

This method is not subscription based rather it simply finds the latest snapshot image from your nest cam and 
returns the response back to you in the form of a promise. You can do whatever you would like with the image 
such as: Uploading to Amazon S3 for processing, saving to disk, etc...

```javascript
const Nest = require('nest-cam');

const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    clientId: "YOUR_CLIENT_ID",
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
});

nest.init().then(() => {
    // Finds the latest snapshot
    nest.getLatestSnapshot().then((data) => {
        console.log('Found the latest snapshot!');
        data.pipe(fs.createWriteStream('/My/Path/to/save/image_3.jpg'));
    });
});
```

### Fetching Event Image

```javascript
const Nest = require('nest-cam');

const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    clientId: "YOUR_CLIENT_ID"
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
});

nest.init().then(() => {
    // 1586551361-labs comes from an event.id property found from getEvents() or subscribe()
    nest.getSnapshot('1586551361-labs', (data) => {
        console.log('Fetched a specific snapshot image: ', data);
    });
});
```

### Streaming Live Camera Images

This is the moment you all have been waiting for! With the Nest package you can stream live images from your Nest camera
as a reactive stream.

This method will subscribe to image data polling the Nest API on a specified cadence (through the nest options) and 
return the image data to your code via promise based callback. You can configure where a new image is fetched 
through the `snapshotInterval` option when you create your `Nest` object.

```javascript
const Nest = require('nest-cam');
const fs = require('fs');

const nest = new Nest({
    nestId: "YOUR_NEST_ID",
    refreshToken: "YOUR_REFRESH_TOKEN",
    apiKey: "YOUR_API_KEY",
    clientId: "YOUR_CLIENT_ID",
    nexusHost: "https://nexusapi-us1.dropcam.com" // or https://nexusapi-eu1.dropcam.com
    snapshotInterval: 20000 // Fetch a new image every 20 seconds
});

nest.init().then(() => {
    nest.subscribeSnapshot((imageData) => {
        console.log('Writing latest snapshot image to disk!');
        data.pipe(fs.createWriteStream('/My/Path/to/save/image_3.jpg'));
    })
});
```

## Running the tests

Unit tests for this project are managed with Mocha and can be executed with:

```shell script
$ npm test
```

### Coding Style Tests

This project uses [ESLint](https://eslint.org/) to manage the coding style. To run the eslint tests and ensure the code is formatted correctly
according to the style guide run:

```shell script
$ sh ./scripts/coding_style_tests.sh
```

### Code Coverage

A new code coverage report can be generated with the following NPM command:

```shell script
$ npm run coverage
```

Code coverage output is stored in lcov format within the `coverage` directory.

## Deployment

This project uses Travis CI to test and deploy the built artifact to NPM. Travis CI configuration settings are managed
in the `.travis.yml` file. 

Deployments are done on tagged commits to the master branch when all the unit and coding style tests pass. Make sure
to bump the version in the `package.json` accordingly before deploying!

## Built With

* [Node.JS](http://www.dropwizard.io/1.0.2/docs/) - The server side library used
* [Javascript](https://maven.apache.org/) - Programming Language Used
* [NPM](https://npmjs.com) - Dependency Management
* [RxJS](https://rometools.github.io/rome/) - Reactive Javascript library for working with streaming data and observables

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Christian Bartram** - *Initial work* - [cbartram](https://github.com/cbartram)

See also the list of [contributors](https://github.com/cbartram/Nest/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE) file for details

## Acknowledgments

* Nest for making a cool camera
* Google for taking down the nest API's without which this project would not be possible
* Charles Proxy for helping me decode the Nest API calls
