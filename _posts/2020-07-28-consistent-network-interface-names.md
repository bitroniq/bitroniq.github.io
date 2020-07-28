---
layout: post

title: "Consistent Network Device Naming (CNDN) when cloning or converting VMs"

excerpt: "Predictable Network Interface Names are the root problem for cloning
and converting VMs, because every time we clone our VM, we don't know what are
the interface names on the new VM."

header:
  image: "assets/images/blog/2020-07-28-cndn.png"
  teaser: "assets/images/blog/2020-07-28-cndn.png"
  og_image: "assets/images/blog/2020-07-28-cndn.png"

carousel:
  item1: "assets/images/blog/2020-07-28-cndn-item01.png"
  item2: "assets/images/blog/2020-07-28-cndn-item02.png"
  item3: "assets/images/blog/2020-07-28-cndn-item03.png"

tags:

- devops
- linux
- networking
- hypervisors
- VirtualBox
- VMware
- ESXi
- Hyper-V

categories:

- DevOps

---

## The Problem

Modern server platforms support an increasing number of network interface ports
on the motherboard (Lan-on-Motherboard or LOM) in addition to numerous add-in
(single and multiport) adapters. Traditionally, network interfaces are
enumerated as eth[012...], but these names do not necessarily correspond to the
actual labels as seen on the chassis. This new naming convention assigns names
to network interfaces based on their physical location, whether embedded or in
PCI slots. By converting to this naming convention, system administrators will
no longer have to guess at the physical location of a network port, or modify
each system to rename them into some consistent order.

In this classic naming scheme for network interfaces, the kernel simply assigns
the names beginning with `eth0`, `eth1`, ... to all the interfaces as they are
probed by the device drivers during the system boot process. As the driver
probing is generally not predictable, in a multi network interfaces setup, a
given network interface that for example, got a name assignment `eth0` in the
first boot may end up with a different name on the next boot. This is
undesirable and can have serious security implications, for example in firewall
rules which are coded for certain naming schemes and which are hence very
sensitive to unpredictable changing names. Also, this naming scheme gives no
clue whatsoever of the interface's physical location on the system (for
example, whether it is on the system's motherboard or if it is on an add-in
card or if it is on an add-in card with multiple ports and which port on the
card it is located). Hence you need a consistent device naming scheme that can
provide the following benefits:

1. Stable network interface names across reboots
1. Stable network interface names when you add or remove hardware
1. Stable network interface names when you update/change the kernel or device
   drivers
1. Stable network interface names when you replace a broken/defective ethernet
  card for example, with a new one
1. The network interface names automatically get determined without user
  configuration and they just work
1. The network interface names are predictable

## Solution for physical servers

### [Predictable Network Interface Names](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)

> Starting with v197 systemd/udev will automatically assign predictable, stable
> network interface names for all local Ethernet, WLAN and WWAN interfaces.
> This is a departure from the traditional interface naming scheme ("eth0",
> "eth1", "wlan0", ...), but should fix real problems.

#### The Purpose

> The classic naming scheme for network interfaces applied by the kernel is to
> simply assign names beginning with "eth0", "eth1", ... to all interfaces as
> they are probed by the drivers. As the driver probing is generally not
> predictable for modern technology this means that as soon as multiple network
> interfaces are available the assignment of the names "eth0", "eth1" and so on
> is generally not fixed anymore and it might very well happen that "eth0" on
> one boot ends up being "eth1" on the next. This can have serious security
> implications, for example in firewall rules which are coded for certain
> naming schemes, and which are hence very sensitive to unpredictable changing
> names.
>
> To fix this problem multiple solutions have been proposed and implemented.
> For a longer time udev shipped support for assigning permanent "ethX" names
> to certain interfaces based on their MAC addresses. This turned out to have a
> multitude of problems, among them: this required a writable root directory
> which is generally not available; the statelessness of the system is lost as
> booting an OS image on a system will result in changed configuration of the
> image; on many systems MAC addresses are not actually fixed, such as on a lot
> of embedded hardware and particularly on all kinds of virtualization
> solutions. The biggest of all however is that the userspace components trying
> to assign the interface name raced against the kernel assigning new names
> from the same "ethX" namespace, a race condition with all kinds of weird
> effects, among them that assignment of names sometimes failed. As a result
> support for this has been removed from systemd/udev a while back
>
> [...]

## Predictable Network Interface Names break VM migration and cloning

Predictable Network Interface Names are the root problem for cloning and
converting VMs, because every time we clone our VM, we don't know what are
the interface names on the new VM.

### I don't like this, how do I disable this?

You basically have three options:

1. You disable the assignment of fixed names, so that the unpredictable kernel
   names are used again. For this, simply mask udev's .link file for
   the default policy: ln -s /dev/null /etc/systemd/network/99-default.link
1. You create your own manual naming scheme, for example by naming your
   interfaces "internet0", "dmz0" or "lan0". For that create your own .link
   files in /etc/systemd/network/, that choose an explicit name or a better
   naming scheme for one, some, or all of your interfaces. See
   systemd.link(5) for more information.
1. You pass the net.ifnames=0 on the kernel command line

### Example solution for Ubuntu 18.04

#### Solution A

```bash
sudo echo 'GRUB_CMDLINE_LINUX="biosdevname=0 net.ifnames=0"' >> /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

#### Solution B

```bash
sed -i.bak 's/GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 bios.devname=0 quiet"/' /etc/default/grub
update-grub
apt-get remove biosdevname -y || true;
```

#### Solution C

Run this script, and if the output looks good, redirect the output into
/etc/udev/rules.d/70-persistent-net.rules to build new udev rules in case the
kernel ever decides to enumerate the ethernet ports in a different order.

```bash
#!/bin/bash
count=0

# build array of network devices starting with eth? from /proc
for dev in `cat /proc/net/dev | egrep 'eth.*:' | awk '{print $1};' | cut -d':' -f1 | sort`; do
   edev[$count]="$dev"
   let count="$count+1"
done

# use array to find mac address
for d in ${edev[@]}; do
   mac=`ip addr show "$d" | grep ether | awk '{print $2};'`
   if [ -n "$mac" ]; then
      echo "# mac for $d is $mac"
   fi

   printf 'SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="%s", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="%s"\n' $mac $d
done
```

## References

1. [Consistent Network Device Naming (CNDN)](https://docs.vmware.com/en/VMware-Adapter-for-SAP-Landscape-Management/services/Administration-Guide-for-LaMa-Administrators/GUID-3979BFD8-D9DB-4C53-9FB8-AC89E024693B.html)
1. [FreeDesktop.org - systemd/ PredictableNetworkInterfaceNames](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)
  * [systemd.link](https://www.freedesktop.org/software/systemd/man/systemd.link.html)
1. [stackexchange.com - Predictable Network Interface Names break vm migration](https://unix.stackexchange.com/questions/335461/predictable-network-interface-names-break-vm-migration)
1. [NetworkInterfaceNames - "Predictable Names" Migration HOWTO](https://wiki.debian.org/NetworkInterfaceNames)
1. [Solution for Ubuntu 18.04](https://github.com/geerlingguy/packer-boxes/issues/1#issuecomment-213116792)

