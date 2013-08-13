# Game Closure DevKit Plugin: Local Notifications

This plugin supports local notifications on iOS and Android platforms.

## Overview

Local notifications allow you to add messages to the phone's status area while
your app is running.  They can appear either immediately or you may schedule the
notifications to appear in the future.

Immediate notifications may be useful to present information to a player while
they are in the middle of gameplay.  For instance, if the player got an
achievement while in the game this is one way to immediately present that
information without disturbing them.

When delayed by a number of days, notifications are often used by games as a
re-engagement strategy.

Notifications appear in the status area with a number of features:

+ action: Name of action for accepting notification. [iOS only]
+ title: Title for alert is Android status area. [Android only]
+ text: Text describing the notification to the user.
+ delay: How far in the future to deliver the nofication (or immediately).
+ icon: Major icon/photo to display for notification.
+ sound: A ringtone-like notification sound that alerts the user.
+ number: Count of notifications represented by that one line.

For more overview see the
[Android notification design pattern](http://developer.android.com/design/patterns/notifications.html)
and the [iOS Local Notifications Guide](http://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Introduction.html#//apple_ref/doc/uid/TP40008194-CH1-SW1).

## Installation

Install the Local Notifications plugin by running `basil install localnotify`.

Include it in the `manifest.json` file under the "addons" section for your game:

~~~
"addons": [
	"localnotify"
],
~~~

You can import the localNotify object anywhere in your application:

~~~
import plugins.localnotify.localNotify as localNotify;
~~~

## Scheduling Notifications

Notification objects have the following schema:

+ `name {string}` (REQUIRED) : Name of the notification.
+ `text {string}` (REQUIRED) : Text describing the notification to the user.
+ `number {integer}` (optional) : Number attached to notification (see below).
+ `sound {string}` (optional) : One of the following options:

	+ undefined : No sound. (default)
	+ true : Use default notification sound.
	+ "resources/sounds/sound.wav" : Custom short .WAV audio file.

+ `action {string}` (optional, iOS-only) : Displayed at phone unlock screen as "Unlock to -action-".
+ `title {string}` (optional, Android-only) : Displayed as a title for the status line in the status area.
+ `icon {string}` (optional) : Path to a .PNG resource to use as an icon for the event.
+ `userDefined {object}` (optional) : User-defined object to store with the notification data.
+ `date {Date}` (optional) : Date when notification should trigger, or:
+ `delay {object}` (optional) : Convenience delay, a sum of:

	+ `seconds {integer}` : Seconds to add to delay (may be fractional).
	+ `minutes {integer}` : Minutes to add to delay (may be fractional).
	+ `hours {integer}` : Hours to add to delay (may be fractional).
	+ `days {integer}` : Days to add to delay (may be fractional).


For example:

~~~
var myNotification = {
	name: "unlock",
	number: 0,
	sound: "notify.wav",
	action: "Unlock",
	title: "Unlocked a New Level",
	body: "You have blasted your way into the Carnage Kingdom!",
	delay: {
		seconds: 32,
		minutes: 1.5,
		hours: 2,
		days: 1
	},
	icon: "resources/images/carnageNotify.png",
	userDefined: {
		bought: false
	}
};
~~~

Notifications can be scheduled by passing them to the `localNotify.add` function.

### Notification: Name

Adding a `name` field to your notification object is required.
When a notification is added it will over-write any existing notification with
the same name.

The name is used to reference the notification in the `remove` and `get` functions,
and it is included in the information returned from the `list` function.

### Notification: Number

Adding the `number` field to your notification object will modify how the
notification appears:

~~~
localNotify.add({
	name: "HeartAlert",
	number: 2,
	body: "2 friends have sent you Hearts!"
});
~~~

On iPhone/iPad, the notification counter on your app icon will be set to `number`.
When your app is launched, the counter will be reset to zero.

On Android, this number will be shown next to the notification in the status area list.

### Combining Notifications

You may choose to group gifts received from other players into a single
notification to avoid overwhelming the player by using the following code:

~~~
function addHeartNotification(fromPlayer, gameName) {
	// Check if single-heart notification exists
	localNotify.get("heart", function(heart) {
		if (heart) {
			var heartCount = heart.count + 1;

			// This will overwrite any existing "hearts" notification
			localNotify.add({
				name: "hearts",
				action: "Accept All",
				title: "Received Gifts: Hearts",
				body: "You have received " + heartCount + " Hearts from friends in " + gameName + "!",
				number: heartCount
			});

			localNotify.remove("heart");
		} else {
			localNotify.add({
				name: "heart",
				action: "Accept",
				title: "Received Gift: Heart",
				body: "You have received a Heart from " + fromPlayer + " in " + gameName + "!",
				number: 1
			});
		}
	});
}
~~~

# localNotify object

## Members:

### localNotify.onNotify (evt)

+ `callback {function}` ---Set to your callback function.
			The first argument will be the object for the triggered notification.

Called whenever a local notification is accepted by the user.  This is the case
for notifications that trigger re-engagement while the app is closed.  In this
event the event(s) that triggered re-engagement will be delivered as soon as the
`onNotify` callback is set.

~~~
localNotify.onNotify = function(evt) {
	logger.log("Got event:", evt.name);
});
~~~

## Methods:

### localNotify.list (callback {function})

Parameters
:	1. `callback {function}` ---Callback function that will receive the list asynchronously.
			The first argument will be an array of pending notifications.

Returns
:    1. `void`

This function enables you to list any pending notifications that are scheduled
for delivery sometime in the future.

Any notifications that have triggered already will be delivered to the `onNotify`
member.

~~~
localNotify.list(function(notifications) {
	for (var ii = 0; ii < notifications.length; ++ii) {
		logger.log("Pending notification:", notifications[ii].name);
	}
});
~~~

### localNotify.get (name, callback {function})

Parameters
:	1. `name {string}` ---The notification name.
	2. `callback {function}` ---Callback function that will receive the info asynchronously.
			The first argument will be the notification info.

Returns
:    1. `void`

This function enables you to get information about a notification by name.

~~~
localNotify.get("name", function(notification) {
	if (notification) {
		logger.log("Notification body:", notification.body);
	} else {
		logger.log("Notification DNE");
	}
});
~~~

### localNotify.clear

Parameters
:	1. `void`

Returns
:    1. `void`

Clears any pending notifications scheduled to be delivered in the future.

~~~
// Remove all notifications
localNotify.clear();
~~~

### localNotify.remove (name)

Parameters
:	1. `name {string}` ---Name of the notification to remove.

Returns
:    1. `void`

This function removes a scheduled notification by name.

~~~
localNotify.remove("name");
~~~

### localNotify.add (object)

Parameters
:	1. `event {object}` ---Notification event to deliver.

Returns
:    1. `void`

This function adds a notification to be delivered.  See above for the object schema.

~~~
localNotify.add({
	name: "heart",
	action: "Accept",
	title: "Received Gift: Heart",
	body: "You have received a Heart from " + fromPlayer + "!"
});
~~~
