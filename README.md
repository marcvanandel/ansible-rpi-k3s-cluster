# Ansible Raspberry Pi k3s Cluster

This repo contains all ingredients and recipies to set up a cluster of Raspberry Pi (v3).
It starts with the set up of the cluster based on Ansible which is copied from [mrlesmithjr/ansible-rpi-k8s-cluster](https://github.com/mrlesmithjr/ansible-rpi-k8s-cluster). Credits to *mrlesmithjr* for this!

## Set Up Dev Env

### Cloning Repo

Because we use submodules for many components within this project, we need to
ensure that we get them as part of the cloning process.

```bash
git clone https://github.com/mrlesmithjr/ansible-rpi-k8s-cluster.git --recurse-submodules
```

### Ansible

> Ansible in [WSL]()


### SD Card Writing


## Raspberry Pi Cluster

First SD card

Install [Rasbian OS]() with [Rufus]()

```
sudo mount -t drvfs D: /mnt/d && touch /mnt/d/ssh && touch /mnt/d/wpa_supplicant.conf && sudo umount /mnt/d
```

`wpa_supplicant.conf`:

```bash
country=NL
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="OpenWrt"
    scan_ssid=1
    psk="k8s-rpis"
    key_mgmt=WPA-PSK
}
```

Other SD cards:

```
sudo mount -t drvfs D: /mnt/d && touch /mnt/d/ssh && sudo umount /mnt/d
```

Check ip-address of the first Pi (via [LuCi](http://192.168.1.1/cgi-bin/luci/admin/status/overview) active leases) and set this in `inventory/hosts.inv`.

In PowerShell with Administrator elivation:

```
route -p add 192.168.100.0/24 192.168.1.115
```

Deploy networking with Ansible (in WSL):

```
cd /mnt/c/.work/repos/ansible-rpi-k8s-cluster
virtualenv ansible
ansible-playbook -i inventory playbooks/bootstrap.yml
```

Deploy [k3s]() with ansible contrib:

```
ansible-playbook -i hosts.ini site.yml
```

Copy credentials to localhost and edit the `server` url (to `https://192.168.1.115:6443`)

```
scp pi@192.168.1.115:~/.kube/config ~/.kube/config
nano ~/.kube/config
```

List resources ... which should be nothing ;-)

```
kubectl get pods
```

Deploy [blinkt controller](https://github.com/apprenda/blinkt-k8s-controller) (works _almost_ out of the box! see [issue #4](https://github.com/apprenda/blinkt-k8s-controller/issues/4#issuecomment-555208803))

Deploy busyboxes... :D

```
kubectl apply -f deployments/busybox1.yaml                    blue
kubectl apply -f deployments/busybox2.yaml                    red
kubectl apply -f deployments/busybox3.yaml                    green
kubectl apply -f deployments/busybox4.yaml                    yellow
```

## Play

```
kubectl get nodes -o wide                                           list nodes
kubectl get pods -o wide                                            list pods
kubectl scale --replicas=8 -f deployments/busybox1.yaml             scale pods
```

## Repo Set Up

- Repo created and cloned
- Submodules added:

```
git submodule add -- https://github.com/mrlesmithjr/ansible-change-hostname.git playbooks/roles/ansible-change-hostname
git submodule add -- https://github.com/mrlesmithjr/ansible-dnsmasq.git playbooks/roles/ansible-dnsmasq
git submodule add -- https://github.com/mrlesmithjr/ansible-isc-dhcp.git playbooks/roles/ansible-isc-dhcp

git submodule add -- https://github.com/mrlesmithjr/ansible-ntp.git playbooks/roles/ansible-ntp
git submodule add -- https://github.com/mrlesmithjr/ansible-apt-cacher-ng.git playbooks/roles/ansible-apt-cacher-ng
```

## WiFi

_TP-Link TL-WR1043ND_

Original firmware:
```
Firmware Version:	3.13.4 Build 110429 Rel.36959n
Hardware Version:	WR1043ND v1 00000000
```

Firmware upgrade with [OpenWRT v18.06.1 - tl-wr1043nd v1.x](https://openwrt.org/toh/tp-link/tl-wr1043nd) (v18.06.4 werkte niet goed) (renamed downloaded file 'cause filename was too long)

Connect to hotspot via [LuCi](http://192.168.1.1/cgi-bin/luci/admin/network/wireless)

`ssh root@192.168.1.1`

Install Wifi WPA2-Enterprise Security:

```
opkg update
opkg remove wpad-mini
opkg install wpad
```

Connect to Kadaster Wifi network via [LuCi](http://192.168.1.1/cgi-bin/luci/admin/network/wireless) (see [forum #1](https://forum.openwrt.org/t/solved-connect-to-wpa2-enterprise/33223/3))

```
SSID          :  Kadaster_Wifi
Encryption:      WPA2-EAP
Cipher    :      auto
EAP-Method:      PEAP
Authentication:  EAP-MSCHAPV2
Identity      :  <kadaster-account-name>
Password      :  <kadaster-account-pass>
```


Create new wifi network via [LuCi](http://192.168.1.1/cgi-bin/luci/admin/network/wireless)

```
SSID          :  OpenWrt
WPA2-PSK      :  k8s-rpis
```
