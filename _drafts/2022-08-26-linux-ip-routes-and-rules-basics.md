# Linux ip route and ip rule Basics

## Prerequisites

1. Bastion host on the cloud with public IP
2. Gateway host in the lab in network `172.16.4.0/24`
3. Host 1 in network `172.16.5.0/24``
4. Host 2 in network `172.16.6.0/24`
5. IPsec tunnel between Gateway and Bastion

## Network Diagram

```
Bastion Host
194.204.152.34
     ^
     |
     |
     |
     v
 Router with NAT
 172.16.4.1/24
     |
  172.16.4.5/24
---------------
|Gateway Host |
|             |- 172.16.5.5/24
|             |
|             |- 172.16.6.5/24
---------------
```

## Bastion configuration

## Gateway configuration

### Configure interfaces using netplan

```bash
root@vmubuntu2004srv:~# vim /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    enp0s3:
      addresses: [172.16.4.5/24]
      nameservers:
        addresses: [1.1.1.1]
      gateway4: 172.16.4.1
    enp0s8:
      addresses:
        - 172.16.5.5/24
    enp0s9:
      addresses:
        - 172.16.6.5/24
  version: 2
```

Finally apply the changes:

```bash
netplan --debug try
```

### Adding ip route tables

We will add two new route tables.

- Choose an available value for your new table.
- It must be between `1` and `252` and not in use by any other table.
- Choose an alphanumeric name with no spaces.
- The order of your tables in the master routing table doesn't matter; rules control the flow.
- These entries in the master routing table just tell the kernel where to find the route information.
- Do not allow any empty lines in the master routing table file.
- Remember, the order in which the indexes (table numbers) are presented in the master routing table doesn't matter.
- The corresponding table numbers do.
- Route tables are processed in chronological order, beginning with number 0.

On Gateway host switch to root with `sudo -i` and edit with `vim` the `/etc/iproute2/rt_tables`

```bash
root@gateway-host:~# vim /etc/iproute2/rt_tables
```

Verify if it looks ok:

```bash
root@vmubuntu2004srv:~# cat /etc/iproute2/rt_tables
255     local
254     main
253     default
252     six
251     five
0       unspec
```

### Adding ip rules

On Gateway host switch to root with `sudo -i`

Add ip rules:

```bash
root@gateway-host:~# ip rule add from all iif enp0s8 lookup five
root@gateway-host:~# ip rule add from all iif enp0s9 lookup six
```

### Final verification

Checking the main route table:

```bash
root@gateway-host:~# ip route show
default dev vti-cde0b1c29f scope link
172.16.4.0/24 dev enp0s3 proto kernel scope link src 172.16.4.5
172.16.5.0/24 dev enp0s8 proto kernel scope link src 172.16.5.5
172.16.6.0/24 dev enp0s9 proto kernel scope link src 172.16.6.5
209.237.128.211 via 172.16.4.1 dev enp0s3
```

Checking route table `five`
```bash
root@gateway-host:~# ip route show table five
default dev vti-cde0b1c29f scope link
```

Checking route table `six`

```bash
root@gateway-host:~# ip route show table six
default dev vti-cde0b1c29f scope link
```

Checking ip rules:

```bash
root@gateway-host:~#  ip rule list
0:      from all lookup local
216:    from all iif enp0s9 lookup six
219:    from all iif enp0s8 lookup five
220:    from all lookup 220
32766:  from all lookup main
32767:  from all lookup default
```

## Host 1 configuration

```bash
root@host1:~# cat /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    enp0s3:
      addresses: [172.16.5.6/24]
      nameservers:
        addresses: [1.1.1.1]
      gateway4: 172.16.5.5
  version: 2
```

## Host 2 configuration

```bash
root@host2:~# cat /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    enp0s3:
      addresses: [172.16.6.6/24]
      nameservers:
        addresses: [1.1.1.1]
      gateway4: 172.16.6.5
  version: 2
```

