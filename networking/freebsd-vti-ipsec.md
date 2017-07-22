---
layout: default
title: Route-based VPN with FreeBSD-11.1's VTI (if_ipsec)
---

# Route-based VPN with FreeBSD-11.1's VTI (if_ipsec)
**NOTE**: This text shows bsd1 configurations only.

## Network configuration
```
                       10.0.0.1      10.0.0.2
192.168.10.0/24 --- [bsd1] ----- /// ----- [bsd2] --- 192.168.20.0/24
172.16.10.0/24                                        172.16.20.0/24
```

## strongSwan setup
### Installation
```
$ sudo pkg install strongswan
```

### /usr/local/etc/ipsec.conf
```
conn route-based
  left = 10.0.0.1
  right = 10.0.0.2
  leftsubnet = 0.0.0.0/0
  rightsubnet = 0.0.0.0/0

  authby = psk
  keyexchange = ikev2
  ike = aes256-sha1-modp1024
  ikelifetime = 28800

  mobike = no
  installpolicy = no
  reqid = 100

  esp = aes256-sha1
  lifetime = 3600

  auto = start
```

### /usr/local/etc/ipsec.secrets
```
10.0.0.2 %any : PSK "xxxxxxxxxxxxxxxx"
```

### /usr/local/etc/strongswan.conf
Just add 'install_routes = no'.
```
charon {
  install_routes = no

  load_modular = yes
  plugins {
    include strongswan.d/charon/*.conf
  }
}

include strongswan.d/*.conf
```

## System setup
### /etc/rc.conf
```
gateway_enable="YES"
cloned_interfaces="ipsec0"
create_args_ipsec0="reqid 100"
ifconfig_ipsec0="inet 169.254.1.1/32 169.254.1.2"
static_routes="remote1 remote2"
route_remote1="192.168.20.0/24 169.254.1.2"
route_remote1="172.16.20.0/24 169.254.1.2"
strongswan_enable="YES"
```

### /etc/rc.local
```
ifconfig ipsec0 inet tunnel 10.0.0.1 10.0.0.2
```
