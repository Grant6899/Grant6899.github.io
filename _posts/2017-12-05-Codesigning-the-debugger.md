---
layout:		post
title:		Codesigning the Debugger
subtitle:
date:		2017-12-05
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - GDB
    - Linux
---

The Darwin Kernel requires the debugger to have special permissions before it is allowed to control other processes. These permissions are granted by codesigning the GDB executable. Without these permissions, the debugger will report error messages such as:

Starting program: /x/y/foo
Unable to find Mach task port for process-id 28885: (os/kern) failure (0x5).
(please check gdb is codesigned - see taskgated(8))
Codesigning requires a certificate. The following procedure explains how to create one:

* Start the Keychain Access application (in /Applications/Utilities/Keychain Access.app)
* Select the Keychain Access -> Certificate Assistant -> Create a Certificate... menu
* Then:
* Choose a name for the new certificate (this procedure will use "gdb-cert" as an example)
* Set "Identity Type" to "Self Signed Root"
* Set "Certificate Type" to "Code Signing"
* Activate the "Let me override defaults" option
* Click several times on "Continue" until the "Specify a Location For The Certificate" screen appears, then set "Keychain" to "System"
* Click on "Continue" until the certificate is created
* Finally, in the view, double-click on the new certificate, and set "When using this certificate" to "Always Trust"
* Exit the Keychain Access application and restart the computer (this is unfortunately required)
Once a certificate has been created, the debugger can be codesigned as follow. In a Terminal, run the following command:

$ codesign -f -s  "gdb-cert"  <gnat_install_prefix>/bin/gdb
where "gdb-cert" should be replaced by the actual certificate name chosen above, and <gnat_install_prefix> should be replaced by the location where you installed GNAT. Also, be sure that users are in the Unix group _developer.