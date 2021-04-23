# tailscale-balena-rpi

This is an example of using the [Tailscale](https://tailscale.com/) mesh VPN on a Raspberry Pi 3 powered by [Balena](https://www.balena.io/).

## Description

This repository contains an example for running [Tailscale](https://tailscale.com/) on a Raspberry Pi 3.
Tailscale is a system that makes it easy to manage the configuration of many devices in a WireGuard VPN setup.
WireGuard is a modern mesh VPN implementation that requires tunnel configurations to be configured on each device.
This is what Tailscale makes easy and reliable on a global scale using a centralized control plane.

The Tailscale node is managed by Balena, an IoT platform for managing and deploying applications to IoT devices.
Balena is built on top of Docker, the Yocto project and other open source technologies.

The [Go clone](https://github.com/librespeed/speedtest-go) of the Librespeed [speedtest](https://github.com/librespeed/speedtest) application has been included as a demo application of what is possible when setting up a WireGuard tunnel configured by Tailscale. 
The `speedtest` service can be used to test the performance of your Tailscale (well, WireGuard) VPN connection.

## Running

Requirements:

* A [Balena](https://www.balena.io/cloud/) account
* A [Tailscale](https://tailscale.com/) account
* A Raspberry Pi (this project has been tested with RPi 3)
* [Another device](https://tailscale.com/download) with Tailscale installed

You need to create a new Balena application or use an existing one and prepare an SD Card with balenaOS.
The example has currently been tested successfully with a Raspberry Pi 3 (using balenaOS versions `2.60.1+rev1.prod` and `balenaOS 2.56.0+rev2.prod`) using the current Dockerfile.template file.
In case a different balenaOS version is used, the `tailscale` Dockerfile.template _may_ have to be updated to use a different version also, otherwise there may be issues when retrieving the kernel headers or compiling the WireGuard kernel module. 
For example, when using a Raspberry Pi 2 with balenaOS `2.48.0+rev1.prod`, errors similar to the following can pop up in the Balena logs:

```bash
tailscale  insmod wireguard...
tailscale  insmod: ERROR: could not insert module /wireguard/wireguard.ko: Invalid module format
tailscale  [  237.670251] wireguard: disagrees about version of symbol module_layout
```

Tailscale may still work, but without a kernel module to run WireGuard.

Using the Balena CLI you can run this project as follows:

```bash
# login to Balena
$ balena login

# push the application to Balena (or to your device in local mode)
$ balena push <application>
```

This can take a while, depending on internet speeds, speed of the Balena builders and whether (some of your) layers were cached before.

After the build process is finished and your device has downloaded the updated images, the `tailscale` and `speedtest` services will be started.
The first time the `tailscale` service will exit early, because the `TAILSCALE_KEY` environment variable is not set and `tailscale` will thus not be able to authenticate itself to the Tailscale servers.
You can create a (reusable) key [here](https://login.tailscale.com/admin/authkeys).
After creating a key, it should be made available as a `Service Variable` for the `tailscale` service in your application in Balena.
After adding the variable, the `tailscale` service will restart and it should show logs similar to the following in the Balena web console:

```
......

20.11.20 17:06:24 (+0100)  tailscale  wireguard version: 1.0.20201112
20.11.20 17:06:25 (+0100)  tailscale  logtail started
20.11.20 17:06:25 (+0100)  tailscale  Program starting: v1.2.8-tcde3a23b6-g1f7ecb611, Go 1.15.4-tsf9db43b: []string{"tailscaled", "-state=/tailscale/tailscaled.state"}
20.11.20 17:06:25 (+0100)  tailscale  Starting userspace wireguard engine with tun device "tailscale0"
20.11.20 17:06:25 (+0100)  tailscale  CreateTUN ok.
20.11.20 17:06:25 (+0100)  tailscale  Creating wireguard device...
20.11.20 17:06:25 (+0100)  tailscale  Bringing wireguard device up...

......

```

After the tunnel is up, you can check your (online) devices in the Tailscale dashboard. 
You can also find the IP address that Tailscale assigned to the Raspberry Pi in the Tailscale dashboard as well as in the Balena dashboard.
You can browse to the IP of the RPi on the other Tailscale connected device to run a speedtest over the WireGuard tunnel.

## Links

The code in this repository was inspired by the contents available on the links below:

* [How to run Wireguard VPN in balenaOS](https://www.balena.io/blog/how-to-run-wireguard-vpn-in-balenaos/)
* [Setting up Tailscale on Raspbian Buster](https://tailscale.com/kb/1043/install-raspbian-buster)
* [klutchell/balena-wireguard](https://github.com/klutchell/balena-wireguard)
* [jaredallard-home/wireguard-balena-rpi](https://github.com/jaredallard-home/wireguard-balena-rpi)
* [kazazes/balena-tailscale](https://github.com/kazazes/balena-tailscale)
* https://github.com/tailscale/tailscale/issues/504
* [Tailscale Kubernetes Operator gist](https://gist.github.com/hamishforbes/2ac7ae9d7ea47cad4e3a813c9b45c10f)


## TODO

* Additional, automatically determined, tags for Tailscale
* Utilities for working with tailscale(d)?
* Examples for DNS and routing configuration options that are available in Tailscale
* Custom -login-server example?
* Nicer build process and more supported devices?
