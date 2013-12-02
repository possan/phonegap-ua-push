# Urban Airship PhoneGap/Cordova Plugin modified for AppGyver Steroids

### Platform Support

This plugin supports apps running on both iOS and Android.

### Version Requirements

This plugin is meant to work with AppGyver Steroids 3.1.0+.

### Differences to the original plugin

This plugin has been forked from the [official Urban Airship plugin](https://github.com/urbanairship/phonegap-ua-push).

Differences to the original include:

* takeOff() must be explicitly called by the application.

* Plugin will be initialized only in those WebViews which have called takeOff(). The major difference is that Steroids has multiple WebViews where Cordova has only one.

* Push notifications are not enabled by default. This will allow the application to define when the user will be asked whether he wants to allow push notifications. The dialog will be presented when enablePush() is called the first time.


### Installation

Please see the AppGyver guide for Push Notifications.

1. Install this plugin to AppGyver build service and build a custom scanner.

2. Modify the www/config.{ios,android}.xml file to contain (replacing with your configuration settings):

        <preference name="com.urbanairship.production_app_key" value="Your production app key" />
        <preference name="com.urbanairship.production_app_secret" value="Your production app secret" />
        <preference name="com.urbanairship.in_production" value="true" />
        <preference name="com.urbanairship.gcm_sender" value="Android only: Your GCM sender id" />

  Note: Your application will always be in production mode, since AppGyver Build Service will give an adhoc build or a custom scanner.

3. If your app supports Android API < 14, then you have to manually instrument any Android Activities to
have proper analytics.
See [Instrumenting Android Analytics](http://docs.urbanairship.com/build/android_features.html#setting-up-analytics-minor-assembly-required).

### Usage:

A ```PushNotification``` object will be available in your application's global JavaScript namespace. See below for API documentation.

Please follow the AppGyver Push Notification Guide (#TODO: Link here) for information how to create relevant certificates and configure the AppGyver Build Service.

A thing to remember: All certificates must be created for Apple's production push notification service! Developer certificates are only for Xcode. Since Build Service will provide you with an ad hoc build, you will need to set up everything as if you were running a production app. Likewise with Urban Airship: You need to configure your application to act in production mode.


## Data objects

The Urban Airship javascript API provides standard instances for some of our data. This allows us to clearly explain what kind of data we're working with when we pass it around throughout the API.

#### Push

    Push = {
        message: "Your team just scored!",
        extras: {
            "url": "/game/5555"
        }
    }

#### Quiet Time

    // Quiet time set to 10PM - 6AM
    QuietTime = {
        startHour: 22,
        startMinute: 0,
        endHour: 6,
        endMinute: 0
    }

A push is an object that contains the data associated with a Push. The extras dictionary can contain arbitrary key and value data, that you can use inside your application.

## API

**All methods without a return value return undefined**

### Top-level calls

#### takeOff()

Initialize plugin in the context of the calling WebView.
This is different to Cordova, where the application only has one WebView.

#### enablePush()

Enable push on the device. This sends a registration to the backend server.

#### disablePush()

Disable push on the device. You will no longer be able to recieve push notifications.

#### enableLocation()

Enable location updates on the device.

#### disableLocation()

Disable location updates on the device.

#### enableBackgroundLocation()

Enable background location updates on the device.

#### disableBackgroundLocation()

Disable background location updates on the device.

#### registerForNotificationTypes(bitmask)
**Note::** iOS Only

On iOS, registration for push requires specifying what combination of badges, sound and
alerts are desired.  This function must be explicitly called in order to begin the
registration process.  For example:

    push.registerForNotificationTypes(push.notificationType.sound | push.notificationType.alert)

*Available notification types:*

* notificationType.sound
* notificationType.alert
* notificationType.badge

### Status Functions

*Callback arguments:* (Boolean status)

All status callbacks are passed a boolean indicating the result of the request:

    push.isPushEnabled(function (has_push) {
        if (has_push) {
            $('#pushEnabled').prop("checked", true)
        }
    })

#### isPushEnabled(callback)

*Callback arguments* (Boolean enabled)

Indicates whether push is enabled.

#### isSoundEnabled(callback)
**Note:** Android Only

*Callback arguments:* (Boolean enabled)

Indicates whether sound is enabled.

#### isVibrateEnabled(callback)
**Note:** Android Only

*Callback arguments:* (Boolean enabled)

Indicates whether vibration is enabled.

#### isQuietTimeEnabled(callback)

*Callback arguments:* (Boolean enabled)

Indicates whether Quiet Time is enabled.

#### isLocationEnabled(callback)

*Callback arguments:* (Boolean enabled)

Indicates whether location is enabled.

#### isBackgroundLocationEnabled(callback)

*Callback arguments:* (Boolean enabled)

Indicates whether background location updates are enabled.

#### isInQuietTime(callback)

*Callback arguments:* (Boolean inQuietTime)

Indicates whether Quiet Time is currently in effect.

### Getters

#### getIncoming(callback)

*Callback arguments:* (Push incomingPush)

Get information about the push that caused the application to be launched. When a user clicks on a push to launch your app, this functions callback will be passed a Push object consisting of the alert message, and an object containing extra key/value pairs.  Otherwise the incoming message and extras will be an empty string and an empty object, respectively.

    push.getIncoming(function (incoming) {
        if (incoming.message) {
            alert("Incoming push message: " + incoming.message;
        }

        if (incoming.extras.url) {
            showPage(incoming.extras.url);
        }
    })

#### getPushID(callback)

*Callback arguments:* (String id)

Get the push identifier for the device. The push ID is used to send messages to the device for testing, and is the canoncial identifer for the device in Urban Airship.

**Note:** iOS will always have a push identifier. Android will always have one once the application has had a successful registration.

#### getQuietTime(callback)

*Callback arguments:* (QuietTime currentQuietTime)

Get the current quiet time.

#### getTags(callback)

*Callback arguments:* (Array currentTags)

Get the current tags.

#### getAlias(callback)

*Callback arguments:* (String currentAlias)

Get the current tags.

### Setters

#### setTags(Array tags, callback)

Set tags for the device.

#### setAlias(String alias, callback)

Set alias for the device.

#### setSoundEnabled(Boolean enabled, callback)
**Note:** Android Only, iOS sound settings come in the push

Set whether the device makes sound on push.

#### setVibrateEnabled(Boolean enabled, callback)
**Note:** Android Only

Set whether the device vibrates on push.

#### setQuietTimeEnabled(Boolean enabled, callback)

Set whether quiet time is on.

#### setQuietTime(QuietTime newQuietTime, callback)

Set the quiet time for the device.

#### setAutobadgeEnabled(Boolean enabled, callback)
**Note:** iOS only

Set whether the UA Autobadge feature is enabled.

#### setBadgeNumber(Int badge, callback)
**Note:** iOS only

Set the current application badge number

#### resetBadge(callback)
**Note:** iOS only

Reset the badge number to zero

### Location

#### recordCurrentLocation(callback)

Report the location of the device.

### Events

**Note:** If your application supports Android and it listens to any of the events, you should
start listening for events on both 'deviceReady' and 'resume' and stop listening for events on 'pause'.
This will prevent the events from being handled in the background.

### Incoming Push

Event:

    {
        message: <Alert Message>,
        extras: <Extras Dictionary>
    }

This event is triggered when a push notification is received.

    document.addEventListener('urbanairship.push', function(event) {
        alert(event.message);
    });


### Registration

Event:

    {
        error: <Error message when registration failed>,
        pushID: <Push address>
    }

This event is triggered when your application receives a registration response from Urban Airship.

    document.addEventListener('urbanairship.registration', function(event) {
        if (event.error) {
            console.log('There was an error registering for push notifications.');
        } else {
            console.log("Registered with ID: " + event.pushID);
        }
    });

