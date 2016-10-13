# react-native-gcm-android

GCM for React Native Android

## Demo

https://github.com/oney/TestGcm

## Installation

- Run `npm install react-native-gcm-android --save`

- In `android/build.gradle`
```gradle
dependencies {
    classpath 'com.android.tools.build:gradle:1.3.1'
    classpath 'com.google.gms:google-services:1.5.0-beta3' // <- Add this line
```

- In `android/settings.gradle`, add
```gradle
include ':RNGcmAndroid', ':app'
project(':RNGcmAndroid').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-gcm-android/android')

include ':react-native-system-notification'
project(':react-native-system-notification').projectDir = new File(settingsDir, '../node_modules/react-native-system-notification/android')
```

- In `android/app/build.gradle`
```gradle
apply plugin: "com.android.application"
apply plugin: 'com.google.gms.google-services'           // <- Add this line
...
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:0.16.+"
    compile 'com.google.android.gms:play-services-gcm:8.3.0' // <- Add this line
    compile project(':RNGcmAndroid')                         // <- Add this line
    compile project(':react-native-system-notification')     // <- Add this line
}
```

- In `android/app/src/main/AndroidManifest.xml`, add these lines, be sure to change `com.xxx.yyy` to your package
```xml
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="com.google.android.c2dm.permission.SEND" />
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
<uses-permission android:name="android.permission.GET_TASKS" /> 
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

<permission
  android:name="com.xxx.yyy.permission.C2D_MESSAGE"
  android:protectionLevel="signature" />
<uses-permission android:name="com.xxx.yyy.permission.C2D_MESSAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.VIBRATE"></uses-permission>

...

<application
  android:theme="@style/AppTheme">

  ...
  <meta-data
    android:name="com.google.android.gms.version"
    android:value="@integer/google_play_services_version" />

  <receiver
    android:name="com.google.android.gms.gcm.GcmReceiver"
    android:exported="true"
    android:permission="com.google.android.c2dm.permission.SEND" >
    <intent-filter>
      <action android:name="com.google.android.c2dm.intent.RECEIVE" />
      <category android:name="com.xxx.yyy" />
    </intent-filter>
  </receiver>
  <service android:name="com.oney.gcm.GcmRegistrationService"/>
  <service android:name="com.oney.gcm.BackgroundService"></service>

  <service
    android:name="com.oney.gcm.RNGcmListenerService"
    android:exported="false" >
    <intent-filter>
      <action android:name="com.google.android.c2dm.intent.RECEIVE" />
    </intent-filter>
  </service>
  <receiver
    android:exported="false"
    android:name="com.oney.gcm.GcmBroadcastReceiver">
    <intent-filter>
      <action android:name="com.oney.gcm.GCMReceiveNotification" />
      </intent-filter>
  </receiver>

  <receiver android:name="io.neson.react.notification.NotificationEventReceiver" />
  <receiver android:name="io.neson.react.notification.NotificationPublisher" />
  <receiver android:name="io.neson.react.notification.SystemBootEventReceiver">
    <intent-filter>
      <action android:name="android.intent.action.BOOT_COMPLETED"></action>
    </intent-filter>
  </receiver>
  ...
```
- In `android/app/src/main/java/com/testoe/MainActivity.java`
```java
import com.oney.gcm.GcmPackage;                             // <- Add this line
import io.neson.react.notification.NotificationPackage;     // <- Add this line
    ...
        .addPackage(new MainReactPackage())
        .addPackage(new GcmPackage())                       // <- Add this line
        .addPackage(new NotificationPackage(this))          // <- Add this line
```

### GCM API KEY
By following [Cloud messaging](https://developers.google.com/cloud-messaging/android/client), you can get `google-services.json` file and place it in `android/app` directory

### Usage

```javascript
'use strict';

import React, { Component } from 'react';
import { AppRegistry, View, DeviceEventEmitter } from 'react-native';
import GcmAndroid from 'react-native-gcm-android';
import Notification from 'react-native-system-notification';
import { Router, Route, Schema, Animations, TabBar } from 'react-native-router-flux';

if (GcmAndroid.launchNotification) {
  const { subject, message } = JSON.parse(GcmAndroid.launchNotification.info);
  Notification.create({ subject, message });
  GcmAndroid.stopService();
} else {
  AppRegistry.registerComponent('YourApp', () => YourApp);
}

class YourApp extends Component {
  componentDidMount () {
    GcmAndroid.addEventListener('register', token => {
      console.log('send gcm token to server', token);
    });
    GcmAndroid.addEventListener('registerError', error => {
      console.log('registerError', error.message);
    });
    GcmAndroid.addEventListener('notification', notification => {
      console.log('receive gcm notification', notification);
      if (GcmAndroid.isInForeground) {
        return;
      }
      const { subject, message } = JSON.parse(notification.data.info);
      Notification.create({ subject, message });
    });

    DeviceEventEmitter.addListener('sysNotificationClick', function(e) {
      console.log('sysNotificationClick', e);
    });

    GcmAndroid.requestPermissions();
  },
  render() {
    return (
      ...
    );
  }
};
```

* There are two situations.
##### The app is running on the foreground or background.
`GcmAndroid.launchNotification` is `null`, you can get notification in `GcmAndroid.addEventListener('notification'` listenter.
##### The app is killed/closed
`GcmAndroid.launchNotification` is your GCM data. You can create notification with resolving the data by using [react-native-system-notification module](https://github.com/Neson/react-native-system-notification).

* You can get info when clicking notification in `DeviceEventEmitter.addListener('sysNotificationClick'`. See [react-native-system-notification](https://github.com/Neson/react-native-system-notification) to get more informations about how to create notification 

## Troubleshoot

- Do not add `multiDexEnabled true` in `android/app/build.gradle` even encounter `com.android.dex.DexException: Multiple dex files...` failure.
- Make sure to install Google Play service in Genymotion simulator before testing.
