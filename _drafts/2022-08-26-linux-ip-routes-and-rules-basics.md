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



## Host 1 configuration

## Host 2 configuration


