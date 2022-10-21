---
layout:     post
title:      ":construction: How to install VMWare Horizon Client in Fedora 32"
date:       2020-10-06 09:15
toc:        true
comments:   true
img:        fedora-logo.avif
tags: 
- How-to 
- fedora32
- VMWare
- VDI
---

Sometimes in my engagements with customers I have to connect to them using [Virtual Desktop Infrastructure (VDI)](https://en.wikipedia.org/wiki/Desktop_virtualization). Basically VDI is defined as the hosting of desktop environments on a central server. It is a form of desktop
virtualization, as the specific desktop images run within virtual machines (VMs) and are delivered to end clients over a
network. Those endpoints may be PCs or other devices, like tablets or thin client terminals.

These kind of systems normally need to install some specific client software to allow the access to them. Me, as Fedora user, found
sometimes issues or problems to install or use these kind of clients. This blog describes the steps I followed to install one
of these clients: [VMware Horizon Client](https://docs.vmware.com/en/VMware-Horizon-Client-for-Linux)

VMware Horizon Client for Linux makes it easy to access your remote desktops and published applications from a supported
Linux system with the best possible user experience on the Local Area Network (LAN) or across a Wide Area Network (WAN).

VMWare Horizon Client supports [Red Hat Enterprise Linux (RHEL)](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux), so
as Fedora is the upstream project for that product, it should works :grin:. Let's try it! :muscle:

The software is available [here](https://my.vmware.com/web/vmware/downloads/details?downloadGroup=CART21FQ1_LIN64_541&productId=863&rPId=36590).

:point_right: This blog describes the steps to install the latest
version available at the time of writing it [VMWare Horizon Client for 64-bit Linux 5.4.1](https://my.vmware.com/web/vmware/downloads/details?downloadGroup=CART21FQ1_LIN64_541&productId=863&rPId=36590). :point_left: 

:warning: It is needed to review the [requirements](https://docs.vmware.com/es/VMware-Horizon-Client-for-Linux/5.4/horizon-client-linux-installation/GUID-DF3FBF68-3C78-45AA-9503-202BD683408F.html),
as a best practice, before to install anything :wink:. :warning:

The only constraint that applies here is that WMWare Horizon Client only
supports X11, so [Wayland](https://wayland.freedesktop.org/) is not a supported protocol :collision:.

Now, it is time to install it with the following commands:

```bash
❯ sudo sh ./VMware-Horizon-Client-5.4.1-15988340.x64.bundle
Extracting VMware Installer...done.
```

The process will ask for the options to install:

[![](/images/vdi/01-vdi-options.avif "WMWare Horizon Client Installation - Options")]({{site.url}}/images/vdi/01-vdi-options.avif = 640x680)

The summary of options selected:

[![](/images/vdi/02-vdi-summary.avif "WMWare Horizon Client Installation - Summary")]({{site.url}}/images/vdi/02-vdi-summary.avif = 640x680)

When the installation finished, we check "Register and start installed service(s) after the installation" to complete
the final steps of the installation: 

[![](/images/vdi/03-vdi-finish.avif "WMWare Horizon Client Installation - Finished")]({{site.url}}/images/vdi/03-vdi-finish.avif = 640x680)

The final step of the installation is a scan system to identify that everything is right:

[![](/images/vdi/04-vdi-scan-system.avif "WMWare Horizon Client Installation - Scan System")]({{site.url}}/images/vdi/04-vdi-scan-system.avif = 640x680)

Finally, we will have the client installed successfully to connect to the VDI.

[![](/images/vdi/05-vdi-horizon-client.avif "WMWare Horizon Client")]({{site.url}}/images/vdi/05-vdi-horizon-client.avif = 640x680)

Have fun with your remote desktop! :computer:

## References

* [WMWare Horizon Client for Linux](https://docs.vmware.com/en/VMware-Horizon-Client-for-Linux)
