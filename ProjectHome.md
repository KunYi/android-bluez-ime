This project aims to create a framework for Bluetooth input devices on the Android platform.

The current version supports the [Zeemote JS1](http://www.zeemote.com/), MSI Chainpus BGP100, [Phonejoy](http://phonejoy.weebly.com/) and [iControlPad](http://www.icontrolpad.com).

Wiimote support is ready, but many devices lack the L2CAP ability and thus cannot communicate with the Wiimote. See [issue #7](https://code.google.com/p/android-bluez-ime/issues/detail?id=#7) for more information. If you think your device has L2CAP support, install the [HID Enabler](https://code.google.com/p/android-bluez-ime/downloads/detail?name=BluezIME-HIDEnabler.apk) package to activate extra devices in Bluez-IME. The HID Enabler works with both the [Bluez-IME version on Google Play](https://play.google.com/store/apps/details?id=com.hexad.bluezime), as well as the one on the [download page](https://code.google.com/p/android-bluez-ime/downloads/list).

Wiimote does not work on devices using either Samsung Galaxy, HTC Sense or LG ROMs with Android 2.x (unknown error: 0). On devices with Android 3.x+, the built-in HID support prevents Bluez-IME from working correctly. If you have any questions or comments on that please write in [issue #7](https://code.google.com/p/android-bluez-ime/issues/detail?id=#7).

LG devices cannot create a RFCOMM channel (invalid UUID), if you have any questions or comments on that, please write in [issue #84](https://code.google.com/p/android-bluez-ime/issues/detail?id=#84).

The application comes bundled with an IME (soft keyboard) emulator, allowing all applications to receive input.

If you need help, have questions or ideas, please [create a new issue](http://code.google.com/p/android-bluez-ime/issues/entry).

The framework also supports broadcasting controller events, allowing any application to instantly support controllers, without requiring the user to activate the IME, by simply subscribing to some broadcast events. See the [sample activity](http://code.google.com/p/android-bluez-ime/source/browse/trunk/BluezIMETestApp/src/com/hexad/bluezime/testapp/TestActivity.java) for a source code example.

<a href='https://www.paypal.com/cgi-bin/webscr?cmd=_xclick&business=paypal%40hexad%2edk&item_name=BluezIME%20Donation&no_shipping=2&no_note=1&tax=0&currency_code=EUR&bn=PP%2dDonationsBF&charset=UTF%2d8'><img src='https://www.paypal.com/en_US/i/btn/btn_donateCC_LG.gif' alt='Donate to Bluez IME' title='You can support Bluez IME by donating a small amount of money' /> <br /> Support Bluez IME by donating</a>

<br /><br />
&lt;wiki:gadget url="http://android-bluez-ime.googlecode.com/svn/adsense/adsense-bluez-ime.xml" width="728" height="90" border="0" /&gt;