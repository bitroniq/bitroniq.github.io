# Linux ip route and ip rule Basics

The goal for this setup is to configure routing based on `ip rule`(s) and dedicated route tables, so that traffic between the `Host1` and `Host2` always goes through the Bastion Host.

Bastion Host is connected with Gateway Host using IPsec tunnel.

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
     |IPsec tunnel
     |
     v
 Router with NAT
 172.16.4.1/24
     |
  172.16.4.5/24
---------------
|Gateway Host |
|             |- 172.16.5.5/24 <-> 172.16.5.6/24 [Host1]
|             |
|             |- 172.16.6.5/24 <-> 172.16.6.6/24 [Host2]
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


## Summary and traceroute

Checking from `host1` - `172.16.5.6` to host `172.16.6.6`.

```bash
root@host1:~# mtr -r -n 172.16.6.6
Start: 2022-08-26T18:32:54+0000
HOST: vmubuntu2004srv             Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 172.16.5.5                30.0%    10    0.8   1.2   0.8   1.6   0.4
  2.|-- 100.65.0.30               80.0%    10   98.7  99.3  98.7  99.9   0.8
  3.|-- 100.65.0.1                 0.0%    10   99.9  99.6  98.3 101.7   0.9
  4.|-- 100.65.0.30               20.0%    10   99.4  99.8  98.9 100.5   0.6
  5.|-- 172.16.5.5                70.0%    10  100.0  99.9  99.3 100.4   0.5
  6.|-- 172.16.6.6                 0.0%    10  199.6 199.3 198.1 200.6   0.8
```
Checking from `host2` - `172.16.6.6` to host `172.16.5.6`.

```bash
root@vmubuntu2004srv:~# mtr -r -n 172.16.5.6
Start: 2022-08-26T18:33:58+0000
HOST: vmubuntu2004srv             Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 172.16.6.5                60.0%    10    1.0   1.2   1.0   1.4   0.2
  2.|-- 100.65.0.30               100.0    10    0.0   0.0   0.0   0.0   0.0
  3.|-- 100.65.0.1                30.0%    10  101.5 101.6 101.0 102.6   0.5
  4.|-- 100.65.0.30               70.0%    10  100.7 101.4 100.7 102.3   0.8
  5.|-- 172.16.6.5                100.0    10    0.0   0.0   0.0   0.0   0.0
  6.|-- 172.16.5.6                 0.0%    10  202.4 202.4 201.9 203.1   0.5
```
