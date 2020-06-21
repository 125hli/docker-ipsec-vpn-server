# IPsec VPN Server on Docker

[![Build Status](https://img.shields.io/travis/hwdsl2/docker-ipsec-vpn-server.svg?maxAge=1200)](https://travis-ci.org/hwdsl2/docker-ipsec-vpn-server) [![GitHub Stars](https://img.shields.io/github/stars/hwdsl2/docker-ipsec-vpn-server.svg?maxAge=86400)](https://github.com/hwdsl2/docker-ipsec-vpn-server/stargazers) [![Docker Stars](https://img.shields.io/docker/stars/hwdsl2/ipsec-vpn-server.svg?maxAge=86400)](https://hub.docker.com/r/hwdsl2/ipsec-vpn-server/) [![Docker Pulls](https://img.shields.io/docker/pulls/hwdsl2/ipsec-vpn-server.svg?maxAge=86400)](https://hub.docker.com/r/hwdsl2/ipsec-vpn-server/)

Docker image to run an IPsec VPN server, with both `IPsec/L2TP` and `Cisco IPsec`.

Based on Debian 10 (Buster) with [Libreswan](https://libreswan.org) (IPsec VPN software) and [xl2tpd](https://github.com/xelerance/xl2tpd) (L2TP daemon).

[**&raquo; See also: IPsec VPN Server on Ubuntu, Debian and CentOS**](https://github.com/hwdsl2/setup-ipsec-vpn)

*Read this in other languages: [English](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README.md), [简体中文](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md).*

#### Table of Contents

- [Install Docker](https://github.com/hwdsl2/docker-ipsec-vpn-server#install-docker)
- [Download](https://github.com/hwdsl2/docker-ipsec-vpn-server#download)
- [How to use this image](https://github.com/hwdsl2/docker-ipsec-vpn-server#how-to-use-this-image)
- [Next steps](https://github.com/hwdsl2/docker-ipsec-vpn-server#next-steps)
- [Important notes](https://github.com/hwdsl2/docker-ipsec-vpn-server#important-notes)
- [Update Docker image](https://github.com/hwdsl2/docker-ipsec-vpn-server#update-docker-image)
- [Advanced usage](https://github.com/hwdsl2/docker-ipsec-vpn-server#advanced-usage)
- [Technical details](https://github.com/hwdsl2/docker-ipsec-vpn-server#technical-details)
- [See also](https://github.com/hwdsl2/docker-ipsec-vpn-server#see-also)
- [License](https://github.com/hwdsl2/docker-ipsec-vpn-server#license)

## Install Docker

First, [install and run Docker](https://docs.docker.com/engine/install/) on your Linux server.

**Note:** This image does not support Docker for Mac or Windows.

## Download

Get the trusted build from the [Docker Hub registry](https://hub.docker.com/r/hwdsl2/ipsec-vpn-server/):

```
docker pull hwdsl2/ipsec-vpn-server
```

Supported platforms: `linux/amd64`, `linux/386`, `linux/arm64`, `linux/arm/v7` and `linux/arm/v6`.

Alternatively, you may [build from source code](https://github.com/hwdsl2/docker-ipsec-vpn-server#build-from-source-code) on GitHub.

## How to use this image

### Environment variables

This Docker image uses the following variables, that can be declared in an `env` file ([example](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/vpn.env.example)):

```
VPN_IPSEC_PSK=your_ipsec_pre_shared_key
VPN_USER=your_vpn_username
VPN_PASSWORD=your_vpn_password
```

This will create a user account for VPN login, which can be used by your multiple devices[*](https://github.com/hwdsl2/docker-ipsec-vpn-server#important-notes). The IPsec PSK (pre-shared key) is specified by the `VPN_IPSEC_PSK` environment variable. The VPN username is defined in `VPN_USER`, and VPN password is specified by `VPN_PASSWORD`.

Additional VPN users are supported, and can be optionally declared in your `env` file like this. Usernames and passwords must be separated by spaces, and usernames cannot contain duplicates. All VPN users will share the same IPsec PSK.

```
VPN_ADDL_USERS=additional_username_1 additional_username_2
VPN_ADDL_PASSWORDS=additional_password_1 additional_password_2
```

**Note:** In your `env` file, DO NOT put `""` or `''` around values, or add space around `=`. DO NOT use these special characters within values: `\ " '`. A secure IPsec PSK should consist of at least 20 random characters.

All the variables to this image are optional, which means you don't have to type in any environment variable, and you can have an IPsec VPN server out of the box! Read the sections below for details.

### Start the IPsec VPN server

Create a new Docker container from this image (replace `./vpn.env` with your own `env` file):

```
docker run \
    --name ipsec-vpn-server \
    --env-file ./vpn.env \
    --restart=always \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -d --privileged \
    hwdsl2/ipsec-vpn-server
```

### Retrieve VPN login details

If you did not specify an `env` file in the `docker run` command above, `VPN_USER` will default to `vpnuser` and both `VPN_IPSEC_PSK` and `VPN_PASSWORD` will be randomly generated. To retrieve them, view the container logs:

```
docker logs ipsec-vpn-server
```

Search for these lines in the output:

```
Connect to your new VPN with these details:

Server IP: your_vpn_server_ip
IPsec PSK: your_ipsec_pre_shared_key
Username: your_vpn_username
Password: your_vpn_password
```

(Optional) Backup the generated VPN login details (if any) to the current directory:

```
docker cp ipsec-vpn-server:/opt/src/vpn-gen.env ./
```

### Check server status

To check the status of your IPsec VPN server, you can pass `ipsec status` to your container like this:

```
docker exec -it ipsec-vpn-server ipsec status
```

Or display current established VPN connections:

```
docker exec -it ipsec-vpn-server ipsec whack --trafficstatus
```

## Next steps

Get your computer or device to use the VPN. Please refer to:

**[Configure IPsec/L2TP VPN Clients](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md)**

**[Configure IPsec/XAuth ("Cisco IPsec") VPN Clients](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-xauth.md)**

**[Advanced usage: Configure and use IKEv2 VPN](https://github.com/hwdsl2/docker-ipsec-vpn-server#configure-and-use-ikev2-vpn)**

If you get an error when trying to connect, see [Troubleshooting](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md#troubleshooting).

Enjoy your very own VPN!

## Important notes

*Read this in other languages: [English](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README.md#important-notes), [简体中文](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md#重要提示).*

**Windows users**: This [one-time registry change](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md#windows-error-809) is required if the VPN server and/or client is behind NAT (e.g. home router).

**Android users**: If you encounter connection issues, try [these steps](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md#android-mtumss-issues).

The same VPN account can be used by your multiple devices. However, due to an IPsec/L2TP limitation, if you wish to connect multiple devices simultaneously from behind the same NAT (e.g. home router), you must use only [IPsec/XAuth mode](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-xauth.md).

For servers with an external firewall (e.g. [EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)/[GCE](https://cloud.google.com/vpc/docs/firewalls)), open UDP ports 500 and 4500 for the VPN. Aliyun users, see [#433](https://github.com/hwdsl2/setup-ipsec-vpn/issues/433).

If you need to edit VPN config files, you must first [start a Bash session](https://github.com/hwdsl2/docker-ipsec-vpn-server#bash-shell-inside-container) in the running container.

If you wish to add, edit or remove VPN user accounts, first update your `env` file, then you must remove and re-create the Docker container using instructions from the [next section](https://github.com/hwdsl2/docker-ipsec-vpn-server#update-docker-image). Advanced users can [bind mount](https://github.com/hwdsl2/docker-ipsec-vpn-server#bind-mount-the-env-file) the `env` file.

Clients are set to use [Google Public DNS](https://developers.google.com/speed/public-dns/) when the VPN is active. If another DNS provider is preferred, [read below](https://github.com/hwdsl2/docker-ipsec-vpn-server#use-alternative-dns-servers).

## Update Docker image

To update your Docker image and container, follow these steps:

```
docker pull hwdsl2/ipsec-vpn-server
```

If the Docker image is already up to date, you should see:

```
Status: Image is up to date for hwdsl2/ipsec-vpn-server:latest
```

Otherwise, it will download the latest version. To update your Docker container, first write down all your [VPN login details](https://github.com/hwdsl2/docker-ipsec-vpn-server#retrieve-vpn-login-details). Then remove the Docker container with `docker rm -f ipsec-vpn-server`. Finally, re-create it using instructions from [this section](https://github.com/hwdsl2/docker-ipsec-vpn-server#how-to-use-this-image).

## Advanced usage

### Use alternative DNS servers

Clients are set to use [Google Public DNS](https://developers.google.com/speed/public-dns/) when the VPN is active. If another DNS provider is preferred, define `VPN_DNS_SRV1` and optionally `VPN_DNS_SRV2` in your `env` file, then follow instructions above to re-create the Docker container. For example, if you wish to use [Cloudflare's DNS service](https://1.1.1.1):

```
VPN_DNS_SRV1=1.1.1.1
VPN_DNS_SRV2=1.0.0.1
```

### Configure and use IKEv2 VPN

*Read this in other languages: [English](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README.md#configure-and-use-ikev2-vpn), [简体中文](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md#配置并使用-ikev2-vpn).*

Using this Docker image, advanced users can configure and use IKEv2. This mode has improvements over IPsec/L2TP and IPsec/XAuth ("Cisco IPsec"), and does not require an IPsec PSK, username or password. Read more [here](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/ikev2-howto.md). After setup, you will be able to connect using any of the three modes.

Please follow these steps:

1. [Download the latest version](https://github.com/hwdsl2/docker-ipsec-vpn-server#update-docker-image) of this Docker image, write down all your [VPN login details](https://github.com/hwdsl2/docker-ipsec-vpn-server#retrieve-vpn-login-details), then remove the Docker container.

   ```
   # Download the latest version of Docker image
   docker pull hwdsl2/ipsec-vpn-server

   # First, write down all your VPN login details
   # Then remove the Docker container
   docker rm -f ipsec-vpn-server
   ```

1. Create a new Docker container (replace `./vpn.env` with your own [env file](https://github.com/hwdsl2/docker-ipsec-vpn-server#how-to-use-this-image)).

   ```
   docker run \
       --name ipsec-vpn-server \
       --env-file ./vpn.env \
       --restart=always \
       -v ikev2-vpn-data:/etc/ipsec.d \
       -p 500:500/udp \
       -p 4500:4500/udp \
       -d --privileged \
       hwdsl2/ipsec-vpn-server
   ```

   In this command, we use the `-v` option of `docker run` to create a new [Docker volume](https://docs.docker.com/storage/volumes/) named `ikev2-vpn-data`, and mount the volume into `/etc/ipsec.d/` in the container. Data will persist in the volume, and later when you need to re-create the Docker container, just specify the same volume again.

1. Check [Docker logs](https://github.com/hwdsl2/docker-ipsec-vpn-server#retrieve-vpn-login-details) to make sure the VPN container started successfully.

   ```
   docker logs ipsec-vpn-server
   ```

1. [Start a Bash session](https://github.com/hwdsl2/docker-ipsec-vpn-server#bash-shell-inside-container) in the running container.

   ```
   docker exec -it ipsec-vpn-server env TERM=xterm bash -l
   ```

1. Download and run the [IKEv2 setup helper script](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/ikev2-howto.md#using-helper-scripts), and follow the prompts.

   ```
   wget https://git.io/ikev2setup -O ikev2.sh && bash ikev2.sh
   ```

   **Note:** If you want to generate certificates for additional VPN clients, just run the helper script again. To revoke a client certificate, see [here](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/ikev2-howto.md#revoke-a-client-certificate).

1. When finished, `exit` the container and continue to [configure IKEv2 VPN clients](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/ikev2-howto.md#configure-ikev2-vpn-clients). To copy the generated `.p12` file to the current directory on the Docker host, you may use, e.g.:

   ```
   docker cp ipsec-vpn-server:/etc/ipsec.d/vpnclient-date-time.p12 ./
   ```

### Build from source code

Advanced users can download and compile the source code from GitHub:

```
git clone https://github.com/hwdsl2/docker-ipsec-vpn-server.git
cd docker-ipsec-vpn-server
docker build -t hwdsl2/ipsec-vpn-server .
```

Or use this if not modifying the source code:

```
docker build -t hwdsl2/ipsec-vpn-server github.com/hwdsl2/docker-ipsec-vpn-server.git
```

### Bash shell inside container

To start a Bash session in the running container:

```
docker exec -it ipsec-vpn-server env TERM=xterm bash -l
```

(Optional) Install the `nano` editor:

```
apt-get update && apt-get -y install nano
```

Then run your commands inside the container. When finished, exit the container and restart if needed:

```
exit
docker restart ipsec-vpn-server
```

### Bind mount the env file

As an alternative to the `--env-file` option, advanced users can bind mount the `env` file. The advantage of this method is that after updating the `env` file, you can restart the Docker container to take effect instead of re-creating it. To use this method, you must first edit your `env` file and use single quotes `''` to enclose the values of all variables. Then (re-)create the Docker container (replace the first `vpn.env` with your own `env` file):

```
docker run \
    --name ipsec-vpn-server \
    --restart=always \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -v "$(pwd)/vpn.env:/opt/src/vpn.env:ro" \
    -d --privileged \
    hwdsl2/ipsec-vpn-server
```

### Enable Libreswan logs

To keep the Docker image small, Libreswan (IPsec) logs are not enabled by default. If you are an advanced user and wish to enable it for troubleshooting purposes, first start a Bash session in the running container:

```
docker exec -it ipsec-vpn-server env TERM=xterm bash -l
```

Then run the following commands:

```
apt-get update && apt-get -y install rsyslog
service rsyslog restart
service ipsec restart
sed -i '/pluto\.pid/a service rsyslog restart' /opt/src/run.sh
exit
```

When finished, you may check Libreswan logs with:

```
docker exec -it ipsec-vpn-server grep pluto /var/log/auth.log
```

To check xl2tpd logs, run `docker logs ipsec-vpn-server`.

## Technical details

There are two services running: `Libreswan (pluto)` for the IPsec VPN, and `xl2tpd` for L2TP support.

The default IPsec configuration supports:

* IKEv1 with PSK and XAuth ("Cisco IPsec")
* IPsec/L2TP with PSK

The ports that are exposed for this container to work are:

* 4500/udp and 500/udp for IPsec

## See also

* [IPsec VPN Server on Ubuntu, Debian and CentOS](https://github.com/hwdsl2/setup-ipsec-vpn)
* [IKEv2 VPN Server on Docker](https://github.com/gaomd/docker-ikev2-vpn-server)

## License

Copyright (C) 2016-2020 [Lin Song](https://www.linkedin.com/in/linsongui) [![View my profile on LinkedIn](https://static.licdn.com/scds/common/u/img/webpromo/btn_viewmy_160x25.png)](https://www.linkedin.com/in/linsongui)   
Based on [the work of Thomas Sarlandie](https://github.com/sarfata/voodooprivacy) (Copyright 2012)

This work is licensed under the [Creative Commons Attribution-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-sa/3.0/)   
Attribution required: please include my name in any derivative and let me know how you have improved it!
