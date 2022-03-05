<h1 align="center">Adb Wifi</h1>



Someone I had the pleasure of working with recently showed me something extremely cool for Android development. It is simply that it is possible to use the ADB (Android Debug Bridge) over TCP/IP or more practically speaking over WiFi. This means that you don't have to tether you'r Android device(s) to you'r computer when you are developing, you can just connect it to a charger or someone elses computer. This is quite handy if you have more Android devices then USB ports and very very handy if you are working with multiple people on the same LAN that are testing you'r latest builds. Here is a quick tutorial (requires Android SDK and ADB installed):

**Step 1.**
Connect you'r Android device to you'r computer and have USB debugging enabled. Also you have to have added the Android tools to you'r path or else you have to go to the SDK folder first. Open up you'r terminal/cmd and type:

> adb tcpip 5555

This should give the output:

> restarting in TCP mode port: 5555


You have now enabled ADB over TCP/IP using the port 5555 (use another port if that one is taken)

**Step 2.**
Now knowing the IP of you'r Android device (Settings->About->Status) type the following into the terminal:

> adb connect <you'r devices IP adress>

you should now get the output:

> connected to <you'r devices IP adress>

**Step 3.**
You should now be able to debug against you'r device as usual as long as it is connected to the same WiFi as you'r computer. A simple test:

> adb devices

Should give the output:

> List of devices attached:
> <you'r devices IP adress>   device

Or something similar. Go ahead and try to read the Logcat using DDMS or simply deploying to you'r device using you'r favourite IDE IntelliJ IDEA. I have tested the following to work over WiFi ADB:



- Logcat output
- Deploy dev app
- Debug dev app
- Screenshots over ADB

I have been using my Nexus S so this may not work on all Android devices, but do give it a try and post here if you are sucessfull (with the name of you'r device). A huge thanks to Håvard for showing me this!

**Update**
Sorry for not checking the comments here, a minor typo is probably causing problems as the first line was with 555 and then the following are with 5555, fixed now, sorry guys.



## 参考

* [Android Quick Tip - ADB over WiFi | Tech And Stuff (stuffandtech.blogspot.com)](http://stuffandtech.blogspot.com/2012/03/android-quick-tip-adb-over-wifi.html)
* [debugging - Run/install/debug Android applications over Wi-Fi? - Stack Overflow](https://stackoverflow.com/questions/4893953/run-install-debug-android-applications-over-wi-fi)
* 

