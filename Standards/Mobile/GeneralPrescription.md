![Architecture Office](https://github.com/ming98/ming98.github.io/blob/master/Standards/Mobile/_images/AO-Logo.png?raw=true)

#General Prescription
High quality apps generally do the following well:

- Good User Experience (UX) 
	- Platform consistent
	- Simple
	- Engaging
	- Uses device capability
- Fast
- Stable
- Secure

The goal of this section is to provide overall prescriptive recommendations which help teams make high-level choices for projects in order to create an app with those attributes. 

##Native vs MADP/MEAP
In general, mobile apps should be developed using the vendor's native platforms. While the MEAP/MADP platforms may look simpler on paper, that is simply not the case in non-trival apps. Typically the first 70% of a project using one of these tools seems easy and quick. However, the remaining 30% will take at least as long as the first 70% and typically end up costing as much as native apps. If choosing a MEAP/MADP there should be a compelling technical or business reason. MEAP/MADP tools typcially struggle when interacting directly with device functionality (camera, bluetooth, etc) and should be avoided for apps using hardware explicitly. 

Read the [Why Native page](WhyNative.md) for a more detailed description of the problem.

##Emulator/Simulator vs Devices
The emulator (Android) and simulator (iOS) alone are not sufficent for developing and testing mobile apps. A team building apps will need to use actual devices, likely both in person and through a service, in addition to the emulator/simulator. The emulator/simulator cannot mimic all device functionality, so any apps using device hardware will ultimiately need an actual device for development. The Emulator/Simulator is sufficent for many development tasks, and with a good development laptop can do much for a developer.

**Emulator/Simulator Advantages:**

- Inexpensive
- Easy to test different screen sizes
- Easy to test different OS versions

**There are some problems common to both the Android emulator and iOS simulator:**

- Not all device functionality available
- Can't measure performance
- Hard to test in different lighting scenarios
- Can't test push notifications

**Android Emulator Problems:**

- Very Slow on AMD chipsets
- Not a vendor-modiifed version of Android
- Chrome not always available
- Simulated hardware (like camera) hides problems

**iOS Simulator Problems:**

- Hides performance issues
- Webview/browser is desktop Safari, not mobile Safari 
- Not the actual mobile operating system

##Secure Practices
###HTTPS
Securing your web sites and services using HTTPS is something you should be doing no matter what, including development sites or services. As of iOS 10, Apple has been forcing developers to make explicit changes to accept insecure connections. As of iOS 11 Apple plans to completely disallow insecure connections. Developing without a secure connection will soon be problematic for mobile development.

###Certificate Pinning
Users, developers, and applications expect end-to-end security on their secure channels, but some secure channels are not meeting the expectation. Specifically, channels built using well known protocols such as VPN, SSL, and TLS can be vulnerable to a number of attacks. Pinning is the process of associating a host with their expected X509 certificate or public key. Once a certificate or public key is known for a host, the certificate or public key is associated or 'pinned' to the host by the mobile application. Essentially the mobile application is testing the certificate recieved during transmission from the host against a known certificate or hash of that certificate. If a server is using a secure channel, a mobile app should pin the certicate in all instances. 

[How to Pin a Certificate in Android](https://davidtruxall.com/android-certificate-pinning/)

[How to Pin a Certificate in iOS]()

##DevOps
##Analytics
