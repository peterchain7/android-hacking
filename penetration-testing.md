
# Android Application penetration Testing

#### Why do we need burp Certificate in `Browser` or `Android Phone emulator`?

```bash
When the app tries to establish a connection to a server, it doesn’t determine which certificates to trust and which not to. The app relies entirely on the certificates that the iOS Trust Store provides or Android CA’s provide by Google.

This method has a weakness, however: An attacker can generate a self-signed certificate and include it in the iOS/Android Trust Store or hack a root CA certificate. This allows such an attacker to set up a man-in-the-middle attack and capture the transmitted data moving to and from your app.
```

- So how burp cert comes into effect?
```bash
As a result,if you try and access an HTTPS URL while Burp is running, your browser/app will detect that it is not communicating directly with the authentic web server and will show a security warning. To prevent this issue, Burp generates its own TLS certificate for each host, signed by its own Certificate Authority (CA) and saved as trusted Certificate in a specific Browser or Android device. (It works perfect when devices is not SSL pinned)
```

#### Installation of `Burp certificate` in Android phone (Genymotion)

```bash
# on rooted devices we have to install burp certificate as SYSTEM like this:
1. Export burp certificate and convert it to pem
    a. openssl x509 -inform DER -in cacert.der -out cacert.pem
2. output the subject_hash_old and rename the file:
    a. openssl x509 -inform PEM -subject_hash_old -in cacert.pem \head -1
       ## Output
       9a5ba575<br>----- BEGIN CERTIFICATE-----<br>MIIDqDCCApCgAwIBAgIFANd0C+8wDQYJKoZIhvcNAQELBQAwgYoxFDASBgNVBAYT<br>C1BvcnRTd2lnZ2VyMRQwEgYDVQQIEwtQb3J0U3dpZ2dlcjEUMBIGA1UEBxMLUG9y<br>dFN3aWdnZXIxFDASBgNVBAoTC1BvcnRTd2lnZ2VyMRcwFQYDVQQLEw5Qb3J0U3dp<br>nnuYKm3uGQ+KWti/NVPZn4ty5CHX9J6d1dxlb4keBGLmfycOOWMY1mVAKpsJQ1xR<br>Ualu7kGgI6nOb06813vjIPRlr/dpZpk938mMgKx0xxkLYs+mOAErmNFP6D3cEK45<br>U5f1yl5EZVzQtAdhnAJhk2QQQO6rxXr2mBASLSC3JUJFhCicIAA+VTnzmHt0ThUG<br>lJ7kWVhQMAbGvckgk2NMiu4AMo90oqwOLd1vGw==
       ----- END CERTIFICATE-----|
       
       But name of certificate should be`md5sum`of subject which can be seen as the fist line with these characters`9a5ba575`. Now to rename certificate use the following command

    b.   mv cacert.pem <hash>.0
    c.   mv cacert.pem 9a5ba575.0

3. Copy the certificate to the device
a. adb push <hash>.0 /sdcard/
b. Adb shell
c. Su
d. mount -o rw,remount /system
e. mv /sdcard/<hash>.0 /system/etc/security/cacerts/
f. chmod 644 /system/etc/security/cacerts/<hash>.0
g. adb reboot

4.  Setting proxy in virtual phone to `host`
     a.  adb shell settings put global http_proxy <Host_Ip>
          - [x] Set Proxy hostname to the IP of the computer running Burp Suite {Professional}.

     b. To unset it:
          adb shell settings put global http_proxy :0
```

#### Intercepting `apk` traffic

-  The above methods will work only when intercepting android device browser traffic, it may be no for apk traffics.


#### Important directories
```bash
	- The directories listed below are the most important directories in an Android device and are worth being aware of.

	- /data/data: Contains all the applications that are installed by the user.

	- /data/user/0: Contains data that only the app can access.

	- /data/app: Contains the APKs of the applications that are installed by the user.

	- /system/app: Contains the pre-installed applications of the device.

	- /system/bin: Contains binary files.

	- /data/local/tmp: A world-writable directory.

	- /data/system: Contains system configuration files.

	- /etc/apns-conf.xml: Contains the default Access Point Name (APN) configurations. APN is used in order for the device to connect with our current carrier’s network.

	- /data/misc/wifi: Contains WiFi configuration files.

	- /data/misc/user/0/cacerts-added: User certificate store. It contains certificates added by the user.

	- /etc/security/cacerts/: System certificate store. Permission to non-root users is not permitted.

	- /sdcard: Contains a symbolic link to the directories DCIM, Downloads, Music, Pictures, etc.
```

#### Using frida in windows OS 

- Frida

```bash
pip3 install Frida-tools
python -m pip install Frida-tools
```

- Pushing the proxy’s CA Certificate

```bash
In order to be able to intercept the traffic, frida needs to have access to your proxy’s CA certificate. We can use the same certificate we downloaded before.Push the certificate into the device and into the same location as the frida-server, name it cert-der.crt (below we will see why this is required).

adb push cacert.der /data/local/tmp/cert-der.crt
```

- Running
```bash
frida --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -f
```


#### Tools installation in window

- Frida

```bash
pip3 install Frida-tools
python -m pip install Frida-tools
```

##### ADB - download platforms tools

######  adb commands

- [-] adb devices: Lists all connected Android devices and their status.
- [-] adb shell: Opens a shell on the Android device, allowing the user to execute commands directly on the device.
- [-] adb install [path to APK]: Installs an Android application on the connected device.
- [-] adb uninstall [package name]: Uninstalls an application from the connected device.
- [-] adb pull [remote file path] [local file path]: Copies a file from the Android device to the local computer.
- [-] adb push [local file path] [remote file path]: Copies a file from the local computer to the Android device.
- [-] adb bugreport: Generates a detailed bug report of the Android device, including system logs, application data, and device information.
- [-] adb screenrecord: Records the Android device’s screen in real-time and saves it as a video file on the local computer.
      Start/ Stop ADB server: If a device is connected start the adb server to be able to interact with the device.
- [-] adb start-server
- [-] adb kill-server
- [-] adb logcat : Displays the Android device’s system log in real-time.
- [-] adb logcatPrint the current device log to the console.
- [-] adb logcat -d > [path_to_file]Save the logcat output to a file on the local system.
- [-] adb logcat -c: The parameter -c will clear the current logs on the device.
      - To capture logs of a specific app:
- [-] adb shell pidof com.example.app (gives you pid of specific app)
- [-] adb logcat --pid 15236 displays log of that app\'s pid only) you can also append -f <filename> to adb logcat command
- [-] adb logcat packagename:[priority level: V, D, I, W, E, F, S]Filter log files by priority e.g. adb logcat com.myapp:E which prints all error logs
- [-] adb shell Allow you to interact with an Android device’s command-line interface directly from your computer
- [-] adb shell pm list packages: Lists all installed packages on the Android device.
- [-] adb shell cmd package list packages: Lists all installed packages on the Android device.
- [-] adb shell am start [intent]: Starts an activity on the Android device using an intent.
- [-] adb shell am force-stop com.android.settings: Stops an activity on the Android device using an intent.
- [-] adb shell input text [text]: Simulates typing text on the Android device’s keyboard.
- [-] adb shell pm path com.example.myapp # Getting the full path of apk file


###### Open Source Frameworks
```bash
- [x] Drozer is an Android security testing framework that allows security researchers to find vulnerabilities and potential exploits in Android applications.

- [x] Ghidra  A software reverse engineering (SRE) suite of tools developed by NSA\'s Research Directorate in support of the Cybersecurity mission.
- [x] Androbugs  is a static code analysis tool that identifies security issues and potential vulnerabilities in Android applications.

- [x] QARK is a dynamic analysis tool that automatically scans Android applications for security vulnerabilities.

- [x] MobSF is a dynamic and static analysis tool that provides an all-in-one solution for mobile application security testing on Android and iOS platforms.
- [x] Android Debug Bridge (ADB): A versatile command-line tool that lets you communicate with a device.

- [x] Dex2jar: Converts .dex files to .class files, zipped as a jar file.

- [x] JD-GUI: A standalone graphical utility that displays Java sources from CLASS files.

- [x] JADX: Command line and GUI tools for producing Java source code from Android Dex and APK files.

- [x] APKTOOL:  A tool for reverse engineering 3rd party, closed, binary Android apps.

- [x] Burp Suite: A set of tools used for web applications penetration testing.

- [x] Frida: A dynamic instrumentation toolkit for developers, reverse engineers, and security researchers.

- [x] Objection: A runtime mobile exploration toolkit, powered by Frida, built to help you assess the security posture of your mobile applications, without needing a jailbreak.

```

- [-] Reference

- [https://medium.com/@srkasthuri/android-pentesting-101-a-novices-handbook-to-getting-started-8f56f877f418](https://medium.com/@srkasthuri/android-pentesting-101-a-novices-handbook-to-getting-started-8f56f877f418)

- [https://www.hackthebox.com/blog/intro-to-mobile-pentesting?source=post_page-----8f56f877f418--------------------------------](https://www.hackthebox.com/blog/intro-to-mobile-pentesting?source=post_page-----8f56f877f418--------------------------------)