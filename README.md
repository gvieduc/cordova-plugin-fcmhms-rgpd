
# cordova-plugin-fcmhms-rgpd
This plugin brings push notifications, event tracking, crash reporting and more from Google Firebase to your Cordova project!
Android and iOS supported.

## Supported Cordova Versions
- cordova: `>= 6`
- cordova-android: `>= 6.3`
- cordova-ios: `>= 4`

## Installation
Install the plugin by adding it your project's config.xml:
```
<plugin name="cordova-plugin-fcmhms-rgpd" spec="^1.0.0" />
```
or by running:
```
cordova plugin add cordova-plugin-fcmhms-rgpd
```

### Guides

### Setup
Download your Firebase configuration files, GoogleService-Info.plist for ios and google-services.json for android, and place them in the root folder of your cordova project.  Check out this [firebase article](https://support.google.com/firebase/answer/7015592) for details on how to download the files.

```
- My Project/
    platforms/
    plugins/
    www/
    config.xml
    google-services.json       <--
    GoogleService-Info.plist   <--
    ...
```

#### IMPORTANT NOTES
- This plugin uses a hook (after prepare) that copies the configuration files to the right place, namely `platforms/ios/\<My Project\>/Resources` for ios and `platforms/android` for android.
- Firebase SDK requires the configuration files to be present and valid, otherwise your app will crash on boot or Firebase features won't work.

### PhoneGap Build
Hooks do not work with PhoneGap Build. This means you will have to manually make sure the configuration files are included. One way to do that is to make a private fork of this plugin and replace the placeholder config files (see `src/ios` and `src/android`) with your actual ones, as well as hard coding your app id and api key in `plugin.xml`.

### Google Play Services
Your build may fail if you are installing multiple plugins that use Google Play Services.  This is caused by the plugins installing different versions of the Google Play Services library.  This can be resolved by installing [cordova-android-play-services-gradle-release](https://github.com/dpa99c/cordova-android-play-services-gradle-release).

## Changing Notification Icon
The plugin will use notification_icon from drawable resources if it exists, otherwise the default app icon is used.
To set a big icon and small icon for notifications, define them through drawable nodes.
Create the required `styles.xml` files and add the icons to the
`<projectroot>/res/native/android/res/<drawable-DPI>` folders.

The example below uses a png named `ic_silhouette.png`, the app Icon (@mipmap/icon) and sets a base theme.
From android version 21 (Lollipop) notifications were changed, needing a separate setting.
If you only target Lollipop and above, you don't need to setup both.
Thankfully using the version dependant asset selections, we can make one build/apk supporting all target platforms.
`<projectroot>/res/native/android/res/values/styles.xml`
```
<?xml version="1.0" encoding="utf-8" ?>
<resources>
    <!-- inherit from the holo theme -->
    <style name="AppTheme" parent="android:Theme.Light">
        <item name="android:windowDisablePreview">true</item>
    </style>
    <drawable name="notification_big">@mipmap/icon</drawable>
    <drawable name="notification_icon">@mipmap/icon</drawable>
</resources>
```
and
`<projectroot>/res/native/android/res/values-v21/styles.xml`
```
<?xml version="1.0" encoding="utf-8" ?>
<resources>
    <!-- inherit from the material theme -->
    <style name="AppTheme" parent="android:Theme.Material">
        <item name="android:windowDisablePreview">true</item>
    </style>
    <drawable name="notification_big">@mipmap/icon</drawable>
    <drawable name="notification_icon">@drawable/ic_silhouette</drawable>
</resources>
```

## Notification Colors

On Android Lollipop and above you can also set the accent color for the notification by adding a color setting.

`<projectroot>/res/native/android/res/values/colors.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="primary">#FFFFFF00</color>
    <color name="primary_dark">#FF220022</color>
    <color name="accent">#FF00FFFF</color>
</resources>
```


## Methods

### getToken

Get the device token (id):
```
window.FCMHMSPlugin.getToken(function(token) {
    // save this server-side and use it to push notifications to this device
    console.log(token);
}, function(error) {
    console.error(error);
});
```
Note that token will be null if it has not been established yet

### onTokenRefresh

Register for token changes:
```
window.FCMHMSPlugin.onTokenRefresh(function(token) {
    // save this server-side and use it to push notifications to this device
    console.log(token);
}, function(error) {
    console.error(error);
});
```
This is the best way to get a valid token for the device as soon as the token is established

### onNotificationOpen

Register notification callback:
```
window.FCMHMSPlugin.onNotificationOpen(function(notification) {
    console.log(notification);
}, function(error) {
    console.error(error);
});
```
Notification flow:

1. App is in foreground:
    1. User receives the notification data in the JavaScript callback without any notification on the device itself (this is the normal behaviour of push notifications, it is up to you, the developer, to notify the user)
2. App is in background:
    1. User receives the notification message in its device notification bar
    2. User taps the notification and the app opens
    3. User receives the notification data in the JavaScript callback

Notification icon on Android:

[Changing notification icon](#changing-notification-icon)

### grantPermission (iOS only)

Grant permission to receive push notifications (will trigger prompt):
```
window.FCMHMSPlugin.grantPermission();
```
### hasPermission

Check permission to receive push notifications:
```
window.FCMHMSPlugin.hasPermission(function(data){
    console.log(data.isEnabled);
});
```

### setBadgeNumber

Set a number on the icon badge:
```
window.FCMHMSPlugin.setBadgeNumber(3);
```

Set 0 to clear the badge
```
window.FCMHMSPlugin.setBadgeNumber(0);
```

### getBadgeNumber

Get icon badge number:
```
window.FCMHMSPlugin.getBadgeNumber(function(n) {
    console.log(n);
});
```

### clearAllNotifications

Clear all pending notifications from the drawer:
```
window.FCMHMSPlugin.clearAllNotifications();
```

### subscribe

Subscribe to a topic:
```
window.FCMHMSPlugin.subscribe("example");
```

### unsubscribe

Unsubscribe from a topic:
```
window.FCMHMSPlugin.unsubscribe("example");
```

### unregister

Unregister from firebase, used to stop receiving push notifications. Call this when you logout user from your app. :
```
window.FCMHMSPlugin.unregister();
```

### verifyPhoneNumber

Request a verification ID and send a SMS with a verification code. Use them to construct a credential to sign in the user (in your app).
- https://firebase.google.com/docs/auth/android/phone-auth
- https://firebase.google.com/docs/reference/js/firebase.auth.Auth#signInWithCredential
- https://firebase.google.com/docs/reference/js/firebase.User#linkWithCredential

**NOTE: This will only work on physical devices.**

iOS will return: credential (string)
Android will return:
credential.verificationId (object and with key verificationId)
credential.instantVerification (boolean)

You need to use device plugin in order to access the right key.

IMPORTANT NOTE: Android supports auto-verify and instant device verification. Therefore in that case it doesn't make sense to ask for an sms code as you won't receive one. Also, **verificationId** will be *false* in this case. In order to sign the user in you need to check **credential.instantVerification**, if it's true, skip the SMS Code entry, call your backend server (sorry, it is the only way to succeed with this plugin) and pass the phone number as param to identify the user (via ajax for example, using any endpoint to your backend).

When using node.js Firebase Admin-SDK, follow this tutorial:
- https://firebase.google.com/docs/auth/admin/create-custom-tokens

Pass back your custom generated token and call
```js
firebase.auth().signInWithCustomToken(customTokenFromYourServer);
```
instead of
```
firebase.auth().signInWithCredential(credential)
```
**YOU HAVE TO COVER THIS PROCESS, OR YOU WILL HAVE ABOUT 5% OF USERS STICKING ON YOUR SCREEN, NOT RECEIVING ANYTHING**
If this process is too complex for you, use this awesome plugin
- https://github.com/chemerisuk/cordova-plugin-firebase-authentication

It's not perfect but it fits for the most use cases and doesn't require calling your endpoint, as it has native phone auth support.

```
window.FCMHMSPlugin.verifyPhoneNumber(number, timeOutDuration, function(credential) {
    console.log(credential);

    // ask user to input verificationCode:
    var code = inputField.value.toString();

    var verificationId = credential.verificationId;

    var credential = firebase.auth.PhoneAuthProvider.credential(verificationId, code);

    // sign in with the credential
    firebase.auth().signInWithCredential(credential);

    // call if credential.instantVerification was true (android only)
    firebase.auth().signInWithCustomToken(customTokenFromYourServer);

    // OR link to an account
    firebase.auth().currentUser.linkWithCredential(credential)
}, function(error) {
    console.error(error);
});
```


#### Android
To use this auth you need to configure your app SHA hash in the android app configuration in the firebase console.
See https://developers.google.com/android/guides/client-auth to know how to get SHA app hash.

#### iOS
Setup your push notifications first, and verify that they are arriving on your physical device before you test this method. Use the APNs auth key to generate the .p8 file and upload it to firebase.  When you call this method, FCM sends a silent push to the device to verify it.

### fetch

Fetch Remote Config parameter values for your app:
```
window.FCMHMSPlugin.fetch(function () {
    // success callback
}, function () {
    // error callback
});
// or, specify the cacheExpirationSeconds
window.FCMHMSPlugin.fetch(600, function () {
    // success callback
}, function () {
    // error callback
});
```

### activateFetched

Activate the Remote Config fetched config:
```
window.FCMHMSPlugin.activateFetched(function(activated) {
    // activated will be true if there was a fetched config activated,
    // or false if no fetched config was found, or the fetched config was already activated.
    console.log(activated);
}, function(error) {
    console.error(error);
});
```

### getValue

Retrieve a Remote Config value:
```
window.FCMHMSPlugin.getValue("key", function(value) {
    console.log(value);
}, function(error) {
    console.error(error);
});
// or, specify a namespace for the config value
window.FCMHMSPlugin.getValue("key", "namespace", function(value) {
    console.log(value);
}, function(error) {
    console.error(error);
});
```

### getByteArray (Android only)
**NOTE: byte array is only available for SDK 19+**
Retrieve a Remote Config byte array:
```
window.FCMHMSPlugin.getByteArray("key", function(bytes) {
    // a Base64 encoded string that represents the value for "key"
    console.log(bytes.base64);
    // a numeric array containing the values of the byte array (i.e. [0xFF, 0x00])
    console.log(bytes.array);
}, function(error) {
    console.error(error);
});
// or, specify a namespace for the byte array
window.FCMHMSPlugin.getByteArray("key", "namespace", function(bytes) {
    // a Base64 encoded string that represents the value for "key"
    console.log(bytes.base64);
    // a numeric array containing the values of the byte array (i.e. [0xFF, 0x00])
    console.log(bytes.array);
}, function(error) {
    console.error(error);
});
```

### getInfo (Android only)

Get the current state of the FirebaseRemoteConfig singleton object:
```
window.FCMHMSPlugin.getInfo(function(info) {
    // the status of the developer mode setting (true/false)
    console.log(info.configSettings.developerModeEnabled);
    // the timestamp (milliseconds since epoch) of the last successful fetch
    console.log(info.fetchTimeMillis);
    // the status of the most recent fetch attempt (int)
    // 0 = Config has never been fetched.
    // 1 = Config fetch succeeded.
    // 2 = Config fetch failed.
    // 3 = Config fetch was throttled.
    console.log(info.lastFetchStatus);
}, function(error) {
    console.error(error);
});
```

### setConfigSettings (Android only)

Change the settings for the FirebaseRemoteConfig object's operations:
```
var settings = {
    developerModeEnabled: true
}
window.FCMHMSPlugin.setConfigSettings(settings);
```

### setDefaults (Android only)

Set defaults in the Remote Config:
```
// define defaults
var defaults = {
    // map property name to value in Remote Config defaults
    mLong: 1000,
    mString: 'hello world',
    mDouble: 3.14,
    mBoolean: true,
    // map "mBase64" to a Remote Config byte array represented by a Base64 string
    // Note: the Base64 string is in an array in order to differentiate from a string config value
    mBase64: ["SGVsbG8gV29ybGQ="],
    // map "mBytes" to a Remote Config byte array represented by a numeric array
    mBytes: [0xFF, 0x00]
}
// set defaults
window.FCMHMSPlugin.setDefaults(defaults);
// or, specify a namespace
window.FCMHMSPlugin.setDefaults(defaults, "namespace");
```

### startTrace

Start a trace.

```
window.FCMHMSPlugin.startTrace("test trace", success, error);
```

### incrementCounter

To count the performance-related events that occur in your app (such as cache hits or retries), add a line of code similar to the following whenever the event occurs, using a string other than retry to name that event if you are counting a different type of event:

```
window.FCMHMSPlugin.incrementCounter("test trace", "retry", success, error);
```

### stopTrace

Stop the trace

```
window.FCMHMSPlugin.stopTrace("test trace");
```
