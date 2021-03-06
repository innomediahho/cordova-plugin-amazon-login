# cordova-plugin-amazon-login

A Cordova Plugin for Login with Amazon. Use your Amazon account to authenticate with the app.
Also supports Companion App APIs for registration of AVS devices.
This plugin is a wrapper around native android and iOS libraries developed by Amazon.
It is advised that you are aware that without having the Amazon Shopping App on the device, on Android, the Amazon libraries will invoke a session of the default browser. Typically this is Chrome which would be fine... however for users with other browsers, such as Firefox, this plugin would break. This is a known issue and stems from how Amazon's libraries work.

Originally from:
``
https://github.com/edu-com/cordova-plugin-amazon-login
``

Changed the build to use dotenv and also extended the plugin to support the Alexa Voice Service 
[Companion App](https://developer.amazon.com/docs/alexa-voice-service/authorize-companion-app.html)
APIs for device registration. 
Support for hosted splash screen via the new ALEXA_PRE_AUTH flag is also included.

Extra Android only helper function to check for various Amazon App presence.
This is to provide support to launch the Alexa App if it is present instead of linking to the Play Store as per Amazon's UX requirements. 

iOS works but is still kind of experimental.
 
## Prerequisites

Last tested to work with 3.0 libraries from the Amazon Mobile App SDK (Jan 2018).

### Android

- Download the [Amazon Mobile App SDK](https://developer.amazon.com/public/resources/development-tools/sdk)  and extract the files to a directory on your hard drive.
- You should see a `login-with-amazon.jar` file in the LoginWithAmazon parent directory.
- Copy `login-with-amazon.jar` file to `platforms/android/libs/login-with-amazon.jar` of your Cordova project.
- Create configuration file for Amazon API keys `config\project.json`
- If you do not have your API Key yet, see [Android App Signatures and API Keys](https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/register_android.html#Android%20App%20Signatures%20and%20API%20Keys) and follow the instructions under "Retrieving an Android API Key".
- Set your Amazon API keys in `build.json`

For example, your `build.json` might look like this:

```
"android": 
{
  "debug": 
  {
    "AMAZON_API_KEY" : "hdgGHHDG......8yRM=="
  },
  "release": 
  {
    "AMAZON_API_KEY" : "dh8HGuDQ......rt5H=="
  }
},
```

By default, the value of API key from `debug` section will be used.
This plugin uses [dotenv](https://www.npmjs.com/package/dotenv) to determine if you are building for `release` or `debug`.
In order to use API key from `release` section set your environment to build `release`.

### iOS

- If you have not installed Xcode, you can get it from [Apple's Xcode](https://developer.apple.com/xcode).
- Download the [Amazon Mobile App SDK](https://developer.amazon.com/public/resources/development-tools/sdk) and extract the files to a directory on your hard drive.
- You should see a LoginWithAmazon.framework directory in the LoginWithAmazon parent directory. This contains the Login with Amazon library.
- Extra Tip for developers on Windows using TACO: Because NTFS doesn't handle symlinks well, you are advised to manually recreate the folders in the archive as directories and extracting the actual files out into those folders.
- With your project open in Xcode, select the Frameworks folder, then click File from the main menu and select Add Files to "project". In the dialog, select LoginWithAmazon.framework and click Add. 
- Select Build Settings. Click All to view all settings.
- Under Search Paths, ensure that the LoginWithAmazon.framework directory is in the Framework Search Paths. For example:
- In the main menu, click Product and select Build. The build should complete successfully.
- [Add a URL Scheme to Your App Property List](https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/create_ios_project.html#add_url_scheme) 
- Get Amazon API key for your iOS app. There is only one key for iOS.
 
 
## Installation

```
cordova plugin add cordova-plugin-amazon-login --variable IOS_API_KEY="your-key-here"
```

Or

```
cordova plugin add https://github.com/innomediahho/cordova-plugin-amazon-login --variable IOS_API_KEY="your-key-here"
```

## API Reference

### Authorize

Perform an OAuth with Amazon based on the invocation profile desired.
Some profile scopes cannot be mixed.

| Type of Profiles              |   Enum  |
| :---------------------------- | ------: |
| None                          | 0x00000 |
| Alexa Pre-Auth                | 0x00001 |
| UserId                        | 0x00010 |
| Profile                       | 0x00020 |
| Postal code                   | 0x00040 |
| Alexa Skills Read             | 0x00100 |
| Alexa Skills Read/Write       | 0x00200 |
| Alexa Skills Test             | 0x00800 |
| Alexa Models Read             | 0x01000 |
| Alexa Models Read/Write       | 0x02000 |
| Dash replenish (experimental) | 0x10000 |

**Alexa Pre-Auth flag will invoke the hosted splash screen during the Companion App registration which is the new Amazon requirement.**

Invoke with the following options:

```
var USERID = 0x00010;
var PROFILE = 0x00020;
var POSTAL_CODE = 0x00040;
var options = {
  scopeFlag: USERID | PROFILE | POSTAL_CODE,
}
```

`window.AmazonLoginPlugin.authorize(Object options, Function success, Function failure)`

Success function returns an object like:

```
{
  accessToken: "...",
  user: 
  {
    name: "Full Name",
    email: "email@example.com",
    user_id: "63456543...",
    postal_code: "123..."
  }
}
```

**Some of these fields may not appear depending on the scope of the profile requested.**

Failure function returns an error String.


### Fetch User Profile

Retrieve previously auth'ed profile information.

`window.AmazonLoginPlugin.fetchUserProfile(Function success, Function failure)`

Success function returns an object like:

```
{
  accessToken: "...",
  user: 
  {
    name: "Full Name",
    email: "email@example.com",
    user_id: "63456543...",
    postal_code: "123..."
  }
}
```

**Some of these fields may not appear depending on the scope of the profile requested.**

Failure function returns an error String.


### Get Token

Invoke with the following options:

```
var USERID = 0x00010;
var options = {
  scopeFlag: USERID,
}
```

`window.AmazonLoginPlugin.getToken(Object options, Function success, Function failure)`

Success function returns an object like:

```
{
  accessToken: "..."
}
```

**Some of these fields may not appear depending on the scope of the profile requested.**

Failure function returns an error String.


### Sign Out

`window.AmazonLoginPlugin.signOut(Function success, Function failure)`

**Android devices without the Amazon Shopping App will leave a session of the Chrome browser still lingering and it would be advised to manage that if you are concerned about user security.**

### Authorize Device

Invoke with the following options:

```
var ALEXA_PRE_AUTH = 0x00001;
var options = {
  scopeFlag: ALEXA_PRE_AUTH,
  productID: "MyDeviceModel",
  productDSN: "SerialNum232...",
  codeChallenge: "2B34E2F342..."
}
```

`window.AmazonLoginPlugin.authorizeDevice(Object options, Function success, Function failure)`

Success function returns an object like:

```
{
  accessToken: "...",
  authorizationCode: "...",
  clientId: "...",
  redirectURI: "https://...",
  user: {
    name: "Full Name",
    email: "email@example.com",
    user_id: "63456543...",
    postal_code: "123..."
  }
}
```

**Some of these fields may not appear depending on the scope of the profile requested.**


Failure function returns an error String.


### (Check if an Amazon) App Exists
**Only supported on Android!**

Helper function to check if such an Amazon App exists on the phone and launch it if found.

| Type of Amazon App   |   Enum  |
| :------------------- | ------: |
| None                 | 0x00000 |
| Amazon Shopping App  | 0x00001 |
| Amazon Alexa App     | 0x00010 |

Please only detect for one app at a time.

Invoke with these options:

```
var APPS_SHOPPING = 0x00001;
var options = {
  appsFlag: APPS_SHOPPING,
  appLaunch: true
}
```

The appLaunch flag set to `true` will launch the Amazon app if found and do nothing if set to `false`.
The className of the Amazon app would be returned regardless on if appLaunch was set `true` or `false` if the requested app was found.
If the app was not detected a null object would be returned. 

`window.AmazonLoginPlugin.appExists(Object options, Function success, Function failure)`

Success function returns an object like:

```
{
  appName: "com.amazon..."
}
```

Failure function returns an error String.

## Extras

For those who are using VisualStudio 2017 TACO and building on a remote Mac with their Amazon libraries on their Windows computers, you might like to add the following script and Cordova hook to help bring the necessary bits across to the other end.

In your config.xml add a reference to the before-build script:

```
    <platform name="ios">
        <hook src="path/to/script/below/before-build.js" type="before_build" />
        ...
    </platform>
```

And the `before-build.js` script for your before-build hook above:

```
#!/usr/bin/env node
'use strict';
const fs = require('fs-extra');

module.exports = function (context) {
    // Ensure we are in an iOS build
    if (context.opts.cordova.platforms.indexOf('ios') !== -1) {        
        var cordovaUtil = context.requireCordovaModule('cordova-lib/src/cordova/util');
        var ConfigParser = context.requireCordovaModule('cordova-lib/src/configparser/ConfigParser');
        var projectRoot = cordovaUtil.isCordova();
        var xml = cordovaUtil.projectConfig(projectRoot);
        var cfg = new ConfigParser(xml);
        var appName = cfg.name();

        // Customize with the location of your LoginWithAmazon.framework path here:
        var pluginPath = 'plugins/cordova-plugin-amazon-login/libs/LoginWithAmazon.framework';

        // The location of where Xcode wants to have (capital P for Plugins) the LoginWithAmazon.framework:
        var iosPath = 'platforms/ios/' + appName + '/Plugins/cordova-plugin-amazon-login/LoginWithAmazon.framework';

        // Copy source folder to where Xcode wants the framework to be
        fs.copy(pluginPath, iosPath, function (err) {
            if (err) {
                console.log('An error occured while copying the framework.');
                return console.error(err);
            }
            console.log('LoginWithAmazon.framework copy completed!');
        });
    }
};
```
