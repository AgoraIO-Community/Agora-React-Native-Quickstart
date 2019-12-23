# Agora React Native Demo

Quickstart for group video calls on react-native using Agora.io SDK.
Use this guide to quickly start a multiple user group call.


## Prerequisites
* '>= react native 0.55.x'
* iOS SDK 8.0+
* Android 5.0+ x86 arm64 armv7a
* A valid Agora account ([Sign up](https://dashboard.agora.io/) for free)

<div class="alert note">Open the specified ports in <a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a> if your network has a firewall.</div>

## Running this example project

### Structure

```
.
├── android
├── components
│ └── permission.js
│ └── Style.js
│ └── Video.js
├── ios
├── index.js
.
```

### Generate an App ID

In the next step, you need to use the App ID of your project. Follow these steps to [create an Agora project](https://docs.agora.io/en/Agora%20Platform/manage_projects?platform=All%20Platforms) in Console and get an [App ID](https://docs.agora.io/en/Agora%20Platform/terms?platform=All%20Platforms#a-nameappidaapp-id ).

1. Go to [Console](https://dashboard.agora.io/) and click the **[Project Management](https://dashboard.agora.io/projects)** icon on the left navigation panel. 
2. Click **Create** and follow the on-screen instructions to set the project name, choose an authentication mechanism, and Click **Submit**. 
3. On the **Project Management** page, find the **App ID** of your project. 

### Steps to run our example

* Download and extract the zip file from the master branch.
* Run npm install or use yarn to install the app dependencies in the unzipped directory.
* Navigate to `./components/Video.js` and edit line 18 to enter your App ID that we generated as `AppID: 'YourAppIDGoesHere'`
* Open a terminal and execute `react-native link react-native-agora`.
* Connect your device and run `react-native run-android` / `react-native run-ios` to start the app.

The app uses `channel-x` as the channel name.

## Understanding the code

### What we need
![Image of how a call works](flow.jpg?raw=true)
### permission.js
We have permission.js to request for camera and microphone permissions from the OS on Android.

### Style.js
We have are styles for the view stored in Style.js

### Video.js

```javascript
import requestCameraAndAudioPermission from './permission';
import React, { Component } from 'react';
import { View, NativeModules, ScrollView, Text, TouchableOpacity, Platform } from 'react-native';
import { RtcEngine, AgoraView } from 'react-native-agora';
import styles from './Style';

const { Agora } = NativeModules;            //Define Agora object as a native module

const {
  FPS30,
  AudioProfileDefault,
  AudioScenarioDefault,
  Adaptative,
} = Agora;                                  //Set defaults for Stream

const config = {                            //Setting config of the app
  appid: 'ENTER APP ID HERE',               //Enter the App ID generated from the Agora Website
  channelProfile: 0,                        //Set channel profile as 0 for RTC
  videoEncoderConfig: {                     //Set Video feed encoder settings
    width: 720,
    height: 1080,
    bitrate: 1,
    frameRate: FPS30,
    orientationMode: Adaptative,
  },
  audioProfile: AudioProfileDefault,
  audioScenario: AudioScenarioDefault,
};
```
We write the used import statements and define the Agora object as a native module and set the defaults from it. We also define the configuration for our RTC engine with settings for the audio and video stream.
```javascript
...
class Video extends Component {
  constructor(props) {
    super(props);
    this.state = {
      peerIds: [],                                       //Array for storing connected peers
      uid: Math.floor(Math.random() * 100),              //Generate a UID for local user
      appid: config.appid,                               
      channelName: 'channel-x',                        //Channel Name for the current session
      joinSucceed: false,                                //State variable for storing success
    };
    if (Platform.OS === 'android') {                    //Request required permissions from Android
      requestCameraAndAudioPermission().then(_ => {
        console.log('requested!');
      });
    }
  }
  ```
  We define the class based video component. In the constructor, we set our state variables: peerIds is an array that stores the unique ID of connected peers used to display their videofeeds, uid is the local user’s unique id that we transmit our videofeed alongside, appid is the agora app id used to authorize access to the sdk, channelName is used to join a channel (users on the same channel can view each other's feeds) and joinSucceed which is used to check if we've successfully joined a channel and setup our scrolling-view.


  ```javascript
  ...
  componentDidMount() {
    RtcEngine.on('userJoined', (data) => {
      const { peerIds } = this.state;                   //Get currrent peer IDs
      if (peerIds.indexOf(data.uid) === -1) {           //If new user has joined
        this.setState({
          peerIds: [...peerIds, data.uid],              //add peer ID to state array
        });
      }
    });
    RtcEngine.on('userOffline', (data) => {             //If user leaves
      this.setState({
        peerIds: this.state.peerIds.filter(uid => uid !== data.uid), //remove peer ID from state array
      });
    });
    RtcEngine.on('joinChannelSuccess', (data) => {                   //If Local user joins RTC channel
      RtcEngine.startPreview();                                      //Start RTC preview
      this.setState({
        joinSucceed: true,                                           //Set state variable to true
      });
    });
    RtcEngine.init(config);                                         //Initialize the RTC engine
  }
  ```
The RTC Engine fires events on user events, we define functions to handle the logic for maintaing user's on the call. We update the peerIds array to store connected users' uids which is used to show their feeds.
When a new user joins the call, we add their uid to the array. When user leaves the call, we remove their uid from the array; if the local users successfully joins the call channel, we start the stream preview.
We use `RtcEngine.init(config)` to initialise the RTC Engine with our defined configuration. 

```javascript
...
  /**
  * @name startCall
  * @description Function to start the call
  */
  startCall = () => {
    RtcEngine.joinChannel(this.state.channelName, this.state.uid);  //Join Channel
    RtcEngine.enableAudio();                                        //Enable the audio
  }
  /**
  * @name endCall
  * @description Function to end the call
  */
  endCall = () => {
    RtcEngine.leaveChannel();
    this.setState({
      peerIds: [],
      joinSucceed: false,
    });
  }
```
We define functions to start and end the call, which we do by joining and leaving the channel and updating our state variables.
```JSX
...
   /**
  * @name videoView
  * @description Function to return the view for the app
  */
  videoView() {
    return (
      <View style={styles.max}>
        {
          <View style={styles.max}>
            <View style={styles.buttonHolder}>
              <TouchableOpacity title="Start Call" onPress={this.startCall} style={styles.button}>
                <Text style={styles.buttonText}> Start Call </Text>
              </TouchableOpacity>
              <TouchableOpacity title="End Call" onPress={this.endCall} style={styles.button}>
                <Text style={styles.buttonText}> End Call </Text>
              </TouchableOpacity>
            </View>
            {
              !this.state.joinSucceed ?
              <View/>
              :
            <View style={styles.fullView}>
              <ScrollView decelerationRate={0}
                style={styles.fullView}>
                <AgoraView style={styles.fullView} showLocalVideo={true} mode={1} />
                {
                  this.state.peerIds.map((data) => (
                    <AgoraView style={styles.fullView}
                      remoteUid={data} mode={1} key={data} />
                  ))
                }
              </ScrollView>
            </View>
          }
          </View>
        }
      </View>
    );
  }
  render() {
    return this.videoView();
  }
}
export default Video;
```
Next we define the view for our videocall, we have a button holder for our start and end call buttons. We also have a scrolling view that contains the videostreams of all the users. We use an AgoraView component, for viewing remote streams we set `remoteUid={'RemoteUidGoesHere'}`. For viewing the local user's stream we set `showLocalVideo={true}`.