# Introduction #

You can get support for gamepads in your application, simply by listening for keypresses:
http://developer.android.com/reference/android/view/KeyEvent.html

The end-user can then install Bluez-IME on their device and map the gamepad buttons as they please. They then activate the IME and then your application, and Bluez IME will send keypresses to your application.

Internally Bluez IME runs a service that responds to certain messages, and broadcasts messages about controller events. The IME part reads the data from the service, applies the user key-mapping and sends the keypresses to your application.

If you want more control over the process, such as reading the real analog values, or detecting disconnect, supporting multiple controllers, etc, you can hook up directly to the Bluez IME service. This is what the rest of this document is about.

# Bluez IME service Layout #

The Bluez IME service is implemented as a standard Android service, and the implementation details can be seen in [BluezService.java](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java).

Upon request, BluezService will start a thread that handles a Bluetooth connection. When an event occurs (such as a button press or analog value change), the thread will broadcast information about this event.

The [BluezIME.java](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezIME.java) class uses this system to produce an [IME](http://developer.android.com/resources/articles/on-screen-inputs.html) that can send keypresses.

# Calling BluezService #

Any process can call [BluezService.java](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java) and send messages. The defined set of messages can be seen in the top of the [BluezService.java](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java) file.

All calls should have the field SESSION\_ID set. The purpose of this field is to ensure that multiple applications can work independently. You should set this value to something that is unique for your application, i.e. prefix it with your package name, like "com.company.superapp.controller#1". BluezService is constructed so that it can handle multiple connections, and you can thus easily ask BluezService to connect to multiple controllers, just give them a unique SESSION\_ID. All messages broadcasted from BluezService will contain the session id, so you can easily figure out if you should handle the message, and what controller produced the message.

# Normal workflow #

The normal application usage would be something like this:
  * Choose a session id
  * Listen to broadcast events from BluezService
  * Ask BluezService to connect to controller with session id
  * Wait for connection or error message with session id
  * Run application as normal, respond to broadcast messages with session id
  * Ask BluezService to disconnect from the controller with session id

There is an [example activity](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIMETestApp/src/com/hexad/bluezime/testapp/TestActivity.java) that shows how to do this.

# Connecting to a controller #

Any application can request a connection, and are expected to provide the MAC adress of the gamepad and the name of the driver. If no MAC or driver is supplied, BluezService will use the one set up in the Bluez IME Settings window. The purpose of this fallback is that the common case is a user with a single gamepad, and they have likely set up the gamepad (or can be asked to do so), and the third party app can be kept free of Bluetooth permissions and dialogs.

If the thirdparty application wants to support multiple controllers (or just a custom setup), they need to scan for Bluetooth devices manually. They also need to [request the supported drivers](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#102). The call does not require a session id, and will return the Bluez IME version as well as a list of supported controllers.

Using this information, an application can now request a connection.

# Data produced by BluezService #

The BluezService produces data in three main categories [keypress](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#54), [analog](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#59) and [accelerometer](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#63).

An application can listen for each of these data types separately, so there is no need to subscribe for analog or accelerometer data if you are only interested in digital button presses.

## Digital data, buttons ##

Digital button presses are sent with a keycode specific to the device, so that a button named "A" gets KEYCODE\_A, and similar for B,C,X,Y,Z.
The DPAD generally sends DPAD_(UP|DOWN|LEFT|RIGHT), and the first analog stick sends W,S,A,D._

There are deviations, such as the Zeemote that sends DPAD\_x values for the analog controller, and sends BUTTON\_X for the button labeled "D".

Since the controllers all have different physical layouts (the Zeemote has 4 buttons and 1 analog stick), there can be no universal setup for all controllers, so most appliations should provide a way for the user to configure buttons.

As most apps do not have any way of reading the analog data, BluezIME will also send emulated keypresses based on analog data. There is [a value broadcasted](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#57) that signals if the keypress is emulated or not so an app can filter those easily. The threshold is currently fixed to "half-range" meaning 64.

## Analog data ##

Analog data uses the convention that each analog value is a signed byte in range (-127, 127) with 0 being center.
Each direction is reported independently, "direction 0" is left-thumbstick X axis, "direction 1" is left-thumbstick Y axis.
"Left and up" are negative values, "down and right" are positive.

Analog trigger buttons (found on the Wiimote) report in the range 0-127 (no negative) and use "direction 5" and "direction 6".

## Accelerometer data ##

Only the Wiimote is capable of sending accelerometer data. It uses the same general approach as the analog data, by sending in range (-127, 127).


# Wiimote specific #

There are a number of [extra control commands](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#97) intended to be used with the Wiimote, that can toggle rumble etc.

## Supporting the Wiimote ##

The Wiimote uses a subset of the HID protocol to communicate over Bluetooth. Currently there are almost no Android devices that supports HID, and even if they do, the Wiimote does not send standard HID signals. Instead, Bluez IME supports Wiimote by using the L2CAP protocol, which is a layer lower than HID. Unfortunately the majority of Android devices are built without support for HID and L2CAP, and adding this support requires a kernel module. Some users have installed custom ROMs which usually have the required L2CAP support, because it is actually part of the regular Android source code.

This means that the majority of users cannot use the Wiimote, and for that reason the version on Market does not report Wiimote support. I tried briefly to enable it and put in a warning, but apparently nobody read it and just rated it 1-star with the message "Does not work". The version on this site has Wiimote support, because there is no rating attached, and people that install software this way are expected to search for an explanation when it does not work. I have not found a reliable way of detecting L2CAP support, which would allow me to detect this at runtime.

If you want Wiimote support in your application, you can just pretend it is there. The version on Market is built with lines [37](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#37) and [46](https://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIME/src/com/hexad/bluezime/BluezService.java#46) commented out, nothing else. When you get the configuration response, you can just add a "wiimote" driver string if one is not already there. This will work on all supported devices, regardless of where they installed Bluez IME from.