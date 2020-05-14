# OpenVPN Multiple Connections

On Windows OpenVPN by default installs one TAP network interface.
If you want to connect to multiple VPNs simultaneously you need an interface for each VPN.

You can add a additional adapter by a batch file provided by the TAP driver.

Open a command prompt with administrative rights and change to the TAP install folder.

```
Microsoft Windows [Version 10.0.19041.21]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>cd "c:\Program Files\TAP-Windows\bin"

c:\Program Files\TAP-Windows\bin>addtap.bat

c:\Program Files\TAP-Windows\bin>rem Add a new TAP virtual ethernet adapter

c:\Program Files\TAP-Windows\bin>"C:\Program Files\TAP-Windows\bin\tapinstall.exe" install "C:\Program Files\TAP-Windows\driver\OemVista.inf" tap0901
Device node created. Install is complete when drivers are installed...
Updating drivers for tap0901 from C:\Program Files\TAP-Windows\driver\OemVista.inf.
Drivers installed successfully.
```

and check it:

```
Microsoft Windows [Version 10.0.19041.21]
(c) 2019 Microsoft Corporation. All rights reserved.

c:\Program Files\TAP-Windows\bin>cd "c:\Program Files\OpenVPN\bin"

c:\Program Files\OpenVPN\bin>openvpn --show-adapters
Available TAP-WIN32 adapters [name, GUID]:
'Ethernet 2' {8867194D-C0A0-4A00-B23C-3D0FF68FE2EE}
'Ethernet 4' {E58DDA99-0AF6-482D-B8A5-50256FC5391E}
```

# References
https://michlstechblog.info/blog/openvpn-connect-to-multiple-vpns-on-windows/