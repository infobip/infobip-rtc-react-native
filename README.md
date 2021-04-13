### Introduction
Infobip RTC is a React Native SDK which enables you to take advantage of Infobip platform, giving you the ability to 
enrich your applications with real-time communications in minimum time, while you focus on your application's user 
experience and business logic. We currently support audio and video calls between two web or app users, and phone calls 
between web or app user and actual phone device. 

Here you will find an overview and a quick guide on how to connect to Infobip platform. There is also in-depth reference
documentation available [here](https://github.com/infobip/infobip-rtc-react-native/wiki). 

### First-time setup
In order to use Infobip RTC, you need to have Web and In-app Calls enabled on your account and that's it! 
You are ready to make Web and In-app calls. To learn how to enable them see 
[the documentation](https://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#set-up-web-and-in-app-calls).

### Getting SDK
InfobipRTC React Native SDK is published on NPM and you can add it as dependency by running the following:

```
npm install infobip-rtc-react-native --save
```

And then you can simply import it in your project:

```javascript
import InfobipRTC from 'infobip-rtc-react-native';
```

### Authentication
Since Infobip RTC is an SDK, it means you develop your own application, and you only use Infobip RTC as a dependency. 
Your application has its own users, which we will refer to as subscribers throughout this guide. 
So, in order to use Infobip RTC, you need to register your subscribers on our platform. 
The credentials your subscribers use to connect to your application are irrelevant to Infobip. 
We only need the identity they will use to present themselves on our platform. 
When we have the subscriber's identity, we can generate a token assigned to that specific subscriber. 
Using that token, your subscribers are able to connect to our platform (using Infobip RTC SDK).

To generate these tokens for your subscribers, you need to call our 
[`/webrtc/1/token`](https://www.infobip.com/docs/api#channels/webrtc/generate-webrtc-token) HTTP API endpoint using proper parameters. 
After you successfully authenticated your subscribers against Infobip platform, we can relate their token to your application.
Typically, generating a token occurs after your subscribers are authenticated inside your application.
You will receive the token in the response that you will use to make and receive calls via  
[`InfobipRTC`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC) client in your mobile application.

### Permissions

#### Android
Audio calls require `RECORD_AUDIO` permission and video calls require `CAMERA` permission.

You can get these permissions using the following code:

```javascript
PermissionsAndroid.request(PermissionsAndroid.PERMISSIONS.RECORD_AUDIO).then((permissionStatus) => {
    if (permissionStatus === PermissionsAndroid.RESULTS.GRANTED) {
        console.log('Granted permission for audio.');
    } else {
        console.warn('Denied permission for audio.');
    }
});

PermissionsAndroid.request(PermissionsAndroid.PERMISSIONS.CAMERA).then((permissionStatus) => {
    if (permissionStatus === PermissionsAndroid.RESULTS.GRANTED) {
        console.log('Granted permission for video.');
    } else {
        console.warn('Denied permission for video.');
    }
});
```

#### iOS
Recording audio or video always requires explicit permission from the user.
iOS requires that your app provide static messages to display to the user when the system asks for camera or microphone permission:
   * If your app uses device cameras, include the `NSCameraUsageDescription` key in your app’s Info.plist file.
   * If your app uses device microphones, include the `NSMicrophoneUsageDescription` key in your app’s Info.plist file.

For each key, provide a message that explains to the user why your app needs to capture media, so that the user can feel confident granting permission to your app.
Please check the [official documentation](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1624584-requestaccess?language=swift) for additional details. 

### Making a call
You can call another subscriber if you know their identity. It is done via the [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call) method:

```javascript
try {
    let token = await getToken();
    let outgoingCall = await InfobipRTC.call(token, 'alice');
  } catch (e) {
    console.log(e);
}
```

Or if you want to initiate a video call use [`CallOptions`](https://github.com/infobip/infobip-rtc-react-native/wiki/CallOptions): 

```javascript
try {
    let token = await getToken();
    let options = CallOptions.builder().setVideo(true).build();
    let outgoingCall = await InfobipRTC.call(token, 'alice', options);
} catch (e) {
    console.log(e);
}
```

As you can see, the [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call) method returns a 
promise that resolves to an instance of [`Call`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call) as the result. 
With it you can track the status of your call and respond to events, such as:

* called subscriber answered the call
* called subscriber rejected the call
* the call has ended

These event handlers can be set up using the following code:

```javascript
outgoingCall.on('ringing', () => console.log('Call is ringing on Alice\'s device!'));
outgoingCall.on('established', (event) => console.log('Alice answered call!'));   
outgoingCall.on('hangup', (event) => console.log(`Call finished with status ${event.name}`));  
outgoingCall.on('error', (event) => console.log(`Something went wrong. Message: ${event.name}`));
```

The most important part of the call is definitely the media that travels between subscribers. 
It starts after the `established` event is received. 

In case of an audio call, you don’t have to do anything to enable the media flow (e.g. to exchange audio with the peer),
as it is done automatically.

In case of a video call, you need to display local and/or remote video using the 
[`InfobipRTCVideoView`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTCVideoView) view component.

```html
<View>
  <InfobipRTCVideoView streamId="local"/>
</View>
<View>
  <InfobipRTCVideoView streamId="remote"/>
</View>
```

When event handlers are set up and the call is established, there are a few things that you can do with the actual call. 
One of them is to hang up the call, which can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call#hangup) 
method. Upon completion, both parties will receive the `hangup` event.

```javascript
outgoingCall.hangup();
```

During the call, you can also mute (and unmute) your audio:

```javascript
outgoingCall.mute(true);
```

To check if the audio is muted, you can call the [`muted`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call#muted) method in the following way:

```javascript
let audioMuted = outgoingCall.muted();
```

### Receiving a call via push notifications
> Note: In order for push notifications to work, they have to be enabled for your application, as explained in 
>[`the documentation`](https://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#create-and-configure-application).

This is the recommended approach since it doesn't use much battery, as the connection is not kept alive, it only listens for incoming push notifications.  

#### iOS
In order to be able to receive incoming calls, your application needs to support several things:

* VoIP Background mode enabled - `Xcode Project` > `Capabilites`> `Background Modes` and make sure the following options are checked:
    + `Voice over IP`
    + `Background fetch`
    + `Remote notifications`
* Push Notifications enabled - `Xcode Project` > `Capabilites` > `Push Notifications`
* Voip Services Certificate - Log into your Apple developer account, find your app under `Identifiers` option, 
enable Push Notifications and generate new certificate following the instructions from Apple. 
Go back to your MacBook and import the generated certificate in your Keychain and then export it as `.p12` file, 
which will be used later to send push notifications.

Once the configuration is done, you need to register PushKit delegate to be able to receive the latest PushKit Push credentials.
After you successfully registered with PushKit and received the latest push credentials, you need to enable push notifications using 
[`enablePushNotification`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#enablePushNotification) method.

```javascript
let token = getToken();
let debug = isDebug();

InfobipRTC.enablePushNotification(token, deviceToken, debug)
    .then(console.log(`Enabled push notifications for deviceToken: ${deviceToken}`))
    .catch((error) => {
        console.error('Error occurred while enabling push notifications.', error);
    });
```

With these steps completed, you should be able to receive VoIP Push Notifications and handle them properly. 
The main thing you need to do upon receiving VoIP Push Notification is to report the new incoming call to CallKit as soon as possible.
After you retrieved information from its payload, in order to get the incoming call from our SDK, you need to handle the received push notification via 
[`handleIncomingCall`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#handleIncomingCall) method.
This is where you define the handler of the incoming call. One of the most common things to do here is to show a prompt for answering or rejecting the call.
For the purpose of this guide, let's look at an example that answers the incoming call as soon as it arrives:

```javascript
InfobipRTC.handleIncomingCall(payload)
    .then((incomingCall) => {
        console.log(`Received incoming call from ${incomingCall.source()}`);
        incomingCall.accept();
    })
    .catch((e) => {
        console.error('Error occurred while handling incoming call.', e);
    });
```

#### Android
[`Here`](https://firebase.google.com/docs/android/setup) you can find a complete tutorial on how to integrate Firebase with your app.
In order to enable push notifications on Android, all you need to do is use the
[`enablePushNotification`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#enablePushNotification) method.
In this method we will collect your device's token and associate it with your identity on our Infobip WebRTC platform.

```javascript
let token = getToken();

InfobipRTC.enablePushNotification(token)
    .then(console.log('Enabled push notifications'))
    .catch((e) => {
        console.error('Error occurred while enabling push notifications.', e);
    });
```

Upon receiving an FCM payload, you can extract message details and call our 
[`handleIncomingCall`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#handleIncomingCall) method to get the incoming call from our SDK. 
This is where you define the handler of the incoming call. One of the most common things to do here is to show a prompt for answering or rejecting the call.
For the purpose of this guide, let's look at an example that answers the incoming call as soon as it arrives:

```javascript
InfobipRTC.handleIncomingCall(payload)
    .then((incomingCall) => {
        console.log(`Received incoming call from ${incomingCall.source()}`);
        incomingCall.accept();
    })
    .catch((e) => {
        console.error('Error occurred while handling incoming call.', e);
    });
```

#### Receiving a call via active connection
Another way to receive a call is to connect once via WebSocket connection to our Infobip WebRTC platform, keep it alive, and receive calls via that connection. 
All this is implemented in our SDK, you just need to call the [`registerForActiveConnection`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#registerForActiveConnection) 
method to actually start listening for incoming calls. The second parameter is the listener that is fired upon receiving the incoming call. 

The downside of this approach is that your app will consume a significant amount of battery, because it persists the connection. 
Use it only when running on a simulator (dev environment).

```javascript
let token = getToken();

InfobipRTC.registerForActiveConnection(token, (incomingCall) => {
    console.log(`Received incoming call from: ${incomingCall.source()}`);
    incomingCall.accept();
});
```

### Calling phone number
It is similar to calling the regular WebRTC user, just use the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#callPhoneNumber) 
method instead of [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call). 
The result of the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#callPhoneNumber) 
is a promise that also resolves to an instance of [`Call`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call) 
with which you can do everything we described earlier.

* Example of calling phone number: 

```javascript
try {
    let token = await getToken();
    let phoneCall = await InfobipRTC.callPhoneNumber(token, '41793026727');
  } catch (e) {
    console.log(e);
}
```

