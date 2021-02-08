### Introduction
Infobip RTC is a React Native SDK which enables you to take advantage of Infobip platform, giving you the ability to enrich your applications with real-time communications in minimum time, while you focus on your application's user experience and business logic. We currently support audio and video calls between two web or app users, and phone calls between web or app user and actual phone device. 

Here you will find an overview and a quick guide on how to connect to Infobip platform. There is also in-depth reference documentation available [here](https://github.com/infobip/infobip-rtc-react-native/wiki). 


### First-time setup
In order to use Infobip RTC, you need to have Web and In-app Calls enabled on your account and that's it! You are ready to make Web and In-app calls. To learn how to enable them see [the documentation](https://www.infobip.com/docs/voice-and-video/web-and-in-app-calls#set-up-web-and-in-app-calls).

### Getting SDK
InfobipRTC React Native SDK is published on NPM and you can add it as dependency running the following:

```
npm install infobip-rtc-react-native --save
```

And then you use it in your project like this:

```
import InfobipRTC from 'infobip-rtc-react-native';
```

### Authentication
Since Infobip RTC is an SDK, it means you develop your own application, and you only use Infobip RTC as a dependency. Your application has your own users, which we will call subscribers throughout this guide. So, in order to use Infobip RTC, you need to register your subscribers on our platform. The credentials your subscribers use to connect to your application are irrelevant to Infobip. We only need the identity they will use to present themselves. When we have the subscriber's identity, we can generate a token assigned to that specific subscriber. With that token, your subscribers can connect to our platform (using Infobip RTC SDK).

To generate these tokens for your subscribers, you need to call our [`/webrtc/1/token`](https://dev.infobip.com/webrtc/generate-token) HTTP API method using proper parameters. There you authenticate yourself against Infobip platform, so we can relate the subscriber's token to you. Typically, generating a token occurs after your subscribers are authenticated inside your application.
You will receive the token in a response that you will use to instantiate [`InfobipRTC`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC) client in your application.


### Permissions
Audio calls require `RECORD_AUDIO` permission and video calls require `CAMERA` permission.
```
if (Platform.OS === 'android') {
    const audioPermissionGranted = await PermissionsAndroid.request(PermissionsAndroid.PERMISSIONS.RECORD_AUDIO);
    if (audioPermissionGranted === PermissionsAndroid.RESULTS.GRANTED) {
      console.log('Granted permission for audio.');
    } else {
      console.warn('Denied permission for audio.');
    }

    const cameraPermissionGranted = await PermissionsAndroid.request(PermissionsAndroid.PERMISSIONS.CAMERA);
    if (cameraPermissionGranted === PermissionsAndroid.RESULTS.GRANTED) {
      console.log('Granted permission for video.');
    } else {
      console.warn('Denied permission for video.');
    }
}
```

### Making a call
You can call another subscriber if you know their identity. It is done via the [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call) method:

```
try {
    let token = await getToken();
    if (!token) {
      return showError('Error occurred while obtaining access token!');
    }
    let outgoingCall = await InfobipRTC.call(token, 'Alice');
  } catch (e) {
    console.log(e);
}
```

Or if you want to initiate video call use CallOptions: 

```
let outgoingCall = await InfobipRTC.call(token, 'Alice', CallOptions.builder().setVideo(true).build());
```
As you can see, the [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call) method returns an instance of [`Call`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call) as the result. With it you can track the status of your call and respond to events, for example when the called subscriber answers the call, rejects it, the call is ended, etc. You set up event handlers with the following code:

```
outgoingCall.on('ringing', () => {
  console.log('Call is ringing on Alice\'s device!');
});
outgoingCall.on('established', (event) => {
  console.log('Alice answered call!');
});
outgoingCall.on('hangup', (event) => {
  console.log('Call is done! Status: ' + event.name);
});
outgoingCall.on('error', (event) => {
  console.log('Oops, something went very wrong! Message: ' + event.name);
});
```
The most important part of the call is definitely the media that travels between subscribers. It starts after the established event is received. In case of audio call, you don’t have to do anything to enable the media flow (receive caller audio and send your audio to the caller), it is done automatically. In case of video call, you need to add local and / or remote video view UI element. You can use `InfobipRTCVideoView` class to display local and remote video on UI:

```
<View>
  <InfobipRTCVideoView streamId="local"/>
</View>
<View>
  <InfobipRTCVideoView streamId="remote"/>
</View>
```

When event handlers are set up and the call is established, there are a few things that you can do with the actual call. One of them, of course, is to hang up. That can be done via the [`hangup`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call#hangup) method on the call, and after that, both parties will receive the `hangup` event upon hang up completion.

```
outgoingCall.hangup();
```

During the call, you can also mute (and unmute) your audio:

```
outgoingCall.mute(true);
```

### Calling phone number
It is similar to calling the regular WebRTC user, you just use the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#callPhoneNumber) method instead of [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call). The result of the [`callPhoneNumber`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#callPhoneNumber) is the [`Call`](https://github.com/infobip/infobip-rtc-react-native/wiki/Call) with which you can do everything as when using the [`call`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#call) method:

* Example of calling phone number: 

```
let token = await obtainToken();
let phoneCall = await InfobipRTC.callPhoneNumber(token, '41793026727');
```
\\
#### Receiving a call via active connection
The second way to receive a call is to connect once via WebSocket connection to our Infobip WebRTC platform, keep it active, and receive calls via that connection. All this is implemented in our SDK, you just need to call the [`registerForActiveConnection`](https://github.com/infobip/infobip-rtc-react-native/wiki/InfobipRTC#registerForActiveConnection) method to actually start listening for incoming calls. The second parameter is the listener that is fired upon the incoming call. 

The downside of this approach is that your app will consume a significant amount of battery, because it persists the connection. Use it only when running on simulator (Dev enviroment).

```
let token = getToken();
InfobipRTC.registerForActiveConnection(token, (incomingCall) => {
    console.log('Received incoming call from: %s', incomingCall.source());
    incomingCall.on('established', (e) => console.log('Call established.'));
    incomingCall.accept();
});
```

