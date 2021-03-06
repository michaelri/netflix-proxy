# netflix-proxy [![Build Status](https://travis-ci.org/ab77/netflix-proxy.svg?branch=master)](https://travis-ci.org/ab77/netflix-proxy) [![](https://www.paypalobjects.com/en_GB/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=5UUCDR8YXWERQ)
`Docker` packaged smart DNS proxy to watch `Netflix`, `Hulu`[n2], `HBO Now` and others out of region using `BIND` and `sniproxy`[n1]. Works for blocked sites too, such as [PornHub](http://www.pornhub.com/).

This solution will only work with devices supporting Server Name Indication (SNI)[n7]. To test, open a web browser on the device you are planning to watch content and go to [this](https://sni.velox.ch/) site (`https://sni.velox.ch/`).

**Update March/2016**: IPv6 addresses of common hosting providers are now blocked in the same way as IPv4. Netflix could to be "tagging" accounts too, so if your account is tagged, the only device that will work out of region is the desktop web browser (i.e. Chrome)[n11]. Netflix and BBC iPlayer are also perfoming geo checks on their media hosts, so the relevant media domains are now proxied by default[n8]. Please note, that proxying media delivery could increase the bandwidth bill you get from your VPS provider. However, since most VPS providers offer 1TB per month inclusive with each server and most home ISPs don't offer anywhere near that amount, it should be a moot point in most situations.

If you feel all of this is too complicated, I don't blame you. If you want change, vote with your wallet by cancelling your Netflix subscription and/or sign the petition:

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/petition.png)](https://act.openmedia.org/netflix)

Please see the [**Wiki**](https://github.com/ab77/netflix-proxy/wiki) page(s) for some common troubleshooting ideas.

**Unblocked Netflix?** Great success! [Vote](http://www.poll-maker.com/poll622505x596e406D-26) now and see the [results](http://www.poll-maker.com/results622505xfcC7F6aA-26).

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/poll_results.png)](http://www.poll-maker.com/results622505xfcC7F6aA-26)

# Supported Services
The following are supported out of the box, however adding additional services is trivial and is done by updating `zones.override` file and running `docker restart bind`:
* Netflix
* Hulu[n2]
* HBO Now 
* Amazon Instant Video
* Crackle
* Pandora
* Vudu
* blinkbox
* BBC iPlayer[n5]
* NBC Sports and potentially many [more](https://github.com/ab77/netflix-proxy/blob/data/conf/zones.override)

# Instructions
The following paragraphs show how to get this solution up and running with a few different Cloud providers I've tried so far.

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/digitalocean.png)](https://m.do.co/c/937b01397c94)

The following is based on a standard Ubuntu Docker image provided by `DigitalOcean`, but should in theory work on any Linux distribution **with** Docker pre-installed. Do **not** enable IPv6 on the host.

1. Head over to [Digital Ocean](https://m.do.co/c/937b01397c94) to get **$10 USD credit**
2. Create a Droplet using `Docker 1.x` on `Ubuntu 14.04` (find in under `One-click Apps` tab).
3. Create a free [tunnel broker](https://tunnelbroker.net/register.php) account.
4. Create a [regular tunnel](https://tunnelbroker.net/new_tunnel.php) and **write-down** your `Routed /64` prefix/subnet.
5. Set the `IPv4 Endpoint` to the Droplet IP, pick a tunnel server in the US and click `Create Tunnel`.
6. Select `Example Configurations` tab, select `Debian/Ubuntu` from the drop-down and copy the tunnel configuration.
7. SSH to your Droplet and add the tunnel configuration to `/etc/network/interfaces` file.
8. Run `ping6 netflix.com` and if you get `Network is unreachable` proceed to the next step, otherwise remove native IPv6 first.
9. Save the file and run: `ifup he-ipv6 && git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -s <your-routed-64-prefix-subnet>`
10. Make sure to **record the credentials** for the `netflix-proxy` admin site.
11. Set your DNS server to the IP of the Droplet, then go to [this](http://ipinfo.io/) site to make sure your Droplet IP is displayed.
12. Finally, enjoy `Netflix` and others out of region.
13. Enjoy or raise a new [issue](https://github.com/ab77/netflix-proxy/issues/new) if something doesn't work quite right (also `#netflix-proxy` on [freenode](https://webchat.freenode.net/?channels=netflix-proxy)).

### Authorising Additional IPs
If you want to share your system with friends and family, you can authorise their home IP address(s) using the `netflix-proxy` admin site, located at `http://<ipaddr>:8080/`, where `ipaddr` is the public IP address of your VPS. Login using `admin` account with the password you recorded during the build, in step 6.

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/admin.png)](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/admin.png)

#### Dynamic IPs
You can also use the `netflix-proxy` admin site to update your IP address, should your ISP assign you a new one (e.g. via DHCP). If your IP address does change, all HTTP/HTTPS requests will automatically be redirected to the admin site on port `8080`. All DNS requests will be redirected to `dnsmasq` instance running on port `5353`. You will most likely need to purge your browser and system DNS caches after this (e.g. `ipconfig /flushdns` and `chrome://net-internals/#dns`) and/or reboot the relevant devices. This mechanism should work on browsers, but will most likely cause errors on other devices, such as Apple TVs and smart TVs. If you Internet stops working all of a sudden, try loading a browser and going to `netflix.com`.

#### Automatic IP Authorization
**WARNING**: do not do enable this unless you know what you are doing.

To enable automatic authorization of every IP that hits your proxy, set `AUTO_AUTH = True` in `auth/settings.py` and run `service netflix-proxy-admin restart`. This setting will effectively authorize any IP hitting your proxy IP with a web browser for the first time, including bots, hackers, spammers, etc. Upon successful authorization, the browser will be redirected to [Google](http://google.com/).

The DNS service is configured with recursion turned on by [default](https://github.com/ab77/netflix-proxy#security), so after a successful authorization, anyone can use your VPS in DNS amplification attacks, which will probably put you in breach of contract with the VPS provider. You have been **WARNED**.

### Security
The build script automatically configures the system with **DNS recursion turned on**. This has security implications, since it potentially opens your DNS server to a DNS amplification attack, a kind of a [DDoS attack](https://en.wikipedia.org/wiki/Denial-of-service_attack). This should not be a concern however, as long as the `iptables` firewall rules configured automatically by the build script for you remain in place. However if you ever decide to turn the firewall off, please be aware of this.

If you want to turn DNS recursion off, please be aware that you will need a mechanism to selectively send DNS requests for domains your DNS server knows about (i.e. netflix.com) to your VPS and send all of the other DNS traffic to your local ISP's DNS server. Something like [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) can be used for this and some Internet routers even have it built in. In order to switch DNS recursion off, you will need to build your system using the following command:

```
git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -r 0 -b 1
```

### Command Line Options
The following command line options can be optionaly passed to `build.sh` for additional control:

```
Usage: ./build.sh [-r 0|1] [-b 0|1] [-c <ip>] [-i 0|1] [-d 0|1] [-t 0|1] [-z 0|1]
        -r      enable (1) or disable (0) DNS recursion (default: 1)
        -b      grab docker images from repository (0) or build locally (1) (default: 0)
        -c      specify client-ip instead of being taken from ssh_connection[n3]
        -i      skip iptables steps
        -d      skip Docker steps
        -t      skip testing steps
        -s      specify IPv6 subnet for Docker (e.g. 2001:470:abcd:123::/64)
        -z      disable caching resolver (default: 0)
```

## Other Cloud Providers

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/vultr.png)](http://www.vultr.com/?ref=6871746)

The following is based on a Debian image provided by `Vultr`, but should in theory work on any Debian distribution. Do **not** enable IPv6 on the host.

1. Head over to [Vultr](http://www.vultr.com/?ref=6871746) to create an account.
2. Create a compute instance using `Debian 8 x64 (jessie)` image.
3. Create a free [tunnel broker](https://tunnelbroker.net/register.php) account.
4. Create a [regular tunnel](https://tunnelbroker.net/new_tunnel.php) and **write-down** your `Routed /64` prefix/subnet.
5. Set the `IPv4 Endpoint` to the IP address of your Vultr instance, pick a tunnel server in the US and click `Create Tunnel`.
6. Select `Example Configurations` tab, select `Debian/Ubuntu` from the drop-down and copy the tunnel configuration.
7. SSH to your server and add the tunnel configuration to `/etc/network/interfaces` file.
8. Run `ping6 netflix.com` and if you get `Network is unreachable` proceed to the next step, otherwise remove native IPv6 first.
9. Save the file and run: `ifup he-ipv6 && apt-get update && apt-get -y install vim dnsutils curl sudo git && curl -sSL https://get.docker.com/ | sh && git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -s <your-routed-64-prefix-subnet>`
10. Make sure to record the credentials for the `netflix-proxy` admin site.
11. Set your DNS server to the IP of the Vultr instance, then go to [this](http://ipinfo.io/) site to make sure your Vultr instance IP is displayed.
12. Finally, enjoy `Netflix` and others out of region.
13. Enjoy or raise a new [issue](https://github.com/ab77/netflix-proxy/issues/new) if something doesn't work quite right (also `#netflix-proxy` on [freenode](https://webchat.freenode.net/?channels=netflix-proxy)).

[![](http://www.ramnode.com/images/banners/affbannerdarknewlogo.png)](https://clientarea.ramnode.com/aff.php?aff=3079)

The following is based on a Debian or Ubuntu OS images provided by `RamNode`. Do **not** enable IPv6 on the host.

1. Head over to [RamNode](https://clientarea.ramnode.com/aff.php?aff=3079) to create an account and buy a **KVM** VPS (OpenVZ won't work).
2. Log into the `VPS Control Panel` and (re)install the OS using `Ubuntu 14.04 x86_64 Server Minimal` or `Debian 8.0 x86_64 Minimal` image.
3. Create a free [tunnel broker](https://tunnelbroker.net/register.php) account.
4. Create a [regular tunnel](https://tunnelbroker.net/new_tunnel.php) and **write-down** your `Routed /64` prefix/subnet.
5. Set the `IPv4 Endpoint` to the IP address of your RamNode VPS, pick a tunnel server in the US and click `Create Tunnel`.
6. Select `Example Configurations` tab, select `Debian/Ubuntu` from the drop-down and copy the tunnel configuration.
7. SSH to your server and add the tunnel configuration to `/etc/network/interfaces` file.
8. Run `ping6 netflix.com` and if you get `Network is unreachable` proceed to the next step, otherwise remove native IPv6 first.
9. Save the file and run: `ifup he-ipv6 && apt-get update && apt-get -y install vim dnsutils curl sudo git && curl -sSL https://get.docker.com/ | sh && git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -s <your-routed-64-prefix-subnet>`
10. Make sure to record the credentials for the `netflix-proxy` admin site.
11. Set your DNS server to the IP of your RamNode intance, then go to [this](http://ipinfo.io/) site to make sure your RamNode instance IP is displayed.
12. Finally, enjoy `Netflix` and others out of region.
13. Enjoy or raise a new [issue](https://github.com/ab77/netflix-proxy/issues/new) if something doesn't work quite right (also `#netflix-proxy` on [freenode](https://webchat.freenode.net/?channels=netflix-proxy)).

[![](https://www.linode.com/media/images/logos/standard/light/linode-logo_standard_light_small.png)](https://www.linode.com/?r=ceb35af7bad520f1e2f4232b3b4d49136dcfe9d9)

**(untested)** The following is based on a standard Ubuntu image provided by `Linode`, but should work on any Linux distribution **without** Docker installed. Do **not** enable IPv6 on the host or disable it post-build and before moving onto step 8.

1. Head over to [Linode](https://www.linode.com/?r=ceb35af7bad520f1e2f4232b3b4d49136dcfe9d9) and sign-up for an account.
2. Create a new `Linode` and deploy an `Ubuntu 14-04 LTS` image into it.
3. Create a free [tunnel broker](https://tunnelbroker.net/register.php) account.
4. Create a [regular tunnel](https://tunnelbroker.net/new_tunnel.php) and **write-down** your `Routed /64` prefix/subnet.
5. Set the `IPv4 Endpoint` to the IP address of your Linode, pick a tunnel server in the US and click `Create Tunnel`.
6. Select `Example Configurations` tab, select `Debian/Ubuntu` from the drop-down and copy the tunnel configuration.
7. SSH to your server and add the tunnel configuration to `/etc/network/interfaces` file.
8. Run `ping6 netflix.com` and if you get `Network is unreachable` proceed to the next step, otherwise remove native IPv6 first.
9. Save the file and run: `ifup he-ipv6 && curl -sSL https://get.docker.com/ | sh && git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -s <your-routed-64-prefix-subnet>`
10. Make sure to record the credentials for the `netflix-proxy` admin site.
11. Set your DNS server to the Linode IP, then go to [this](http://ipinfo.io/) site to make sure your Linode IP is displayed.
12. Finally, enjoy `Netflix` and others out of region.
13. Enjoy or raise a new [issue](https://github.com/ab77/netflix-proxy/issues/new) if something doesn't work quite right (also `#netflix-proxy` on [freenode](https://webchat.freenode.net/?channels=netflix-proxy)).

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/dreamhost.png)](http://www.dreamhost.com/r.cgi?2124700)

**(untested)** The following is based on a standard Ubuntu image provided by `DreamHost`, but should work on any Linux distribution **without** Docker installed and running under **non-root** user (e.g. `Amazon Web Services`). Do **not** enable IPv6 on the host.

1. Head over to [DreamHost](http://www.dreamhost.com/r.cgi?2124700) and sign-up for an account.
2. Find the `DreamCompute` or `Public Cloud Computing` section and launch an `Ubuntu 14-04-Trusty` instance.
3. Make sure to add an additional firewall rule to allow DNS: `Ingress	IPv4	UDP	53	0.0.0.0/0 (CIDR)`
4. Also add a `Floating IP` to your instance.
5. Create a free [tunnel broker](https://tunnelbroker.net/register.php) account.
6. Create a [regular tunnel](https://tunnelbroker.net/new_tunnel.php) and **write-down** your `Routed /64` prefix/subnet.
7. Set the `IPv4 Endpoint` to the IP address of your instance, pick a tunnel server in the US and click `Create Tunnel`.
8. Select `Example Configurations` tab, select `Debian/Ubuntu` from the drop-down and copy the tunnel configuration.
9. SSH to your server and add the tunnel configuration to `/etc/network/interfaces` file.
10. Run `ping6 netflix.com` and if you get `Network is unreachable` proceed to the next step, otherwise remove native IPv6 first.
11. Save the file and run: `ifup he-ipv6 && curl -sSL https://get.docker.com/ | sh && sudo usermod -aG docker $(whoami | awk '{print $1}') && sudo git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -s <your-routed-64-prefix-subnet>`
12. Make sure to record the credentials for the `netflix-proxy` admin site.
13. Point your DNS at the instance IP, then go to [this](http://ipinfo.io/) site to make sure your instance IP is displayed.
14. Finally, enjoy `Netflix` and others out of region.
15. Enjoy or raise a new [issue](https://github.com/ab77/netflix-proxy/issues/new) if something doesn't work quite right (also `#netflix-proxy` on [freenode](https://webchat.freenode.net/?channels=netflix-proxy)).

[![](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/gandi.png)](https://www.gandi.net/hosting/iaas/buy)

The following is based on (slightly broken) Ubuntu image provided by `Gandi` using` root` login with SSH key only (no password). For default non-root `admin` login, adjust step 8 to use `sudo` where nesessary.

1. Head over to [Gandi](https://www.gandi.net/hosting/iaas/buy) to create a virtual server.
2. Create a free [tunnel broker](https://tunnelbroker.net/register.php) account.
3. Create a [regular tunnel](https://tunnelbroker.net/new_tunnel.php) and **write-down** your `Routed /64` prefix/subnet.
4. Set the `IPv4 Endpoint` to the IP address of your server, pick a tunnel server in the US and click `Create Tunnel`.
5. Select `Example Configurations` tab, select `Debian/Ubuntu` from the drop-down and copy the tunnel configuration.
6. SSH to your server and add the tunnel configuration to `/etc/network/interfaces` file.
7. Remove native IPv6 IP(s) from the public network interfaces (i.e. `eth0`)
8. Save the file and run: `ifup he-ipv6 && apt-get -y update && apt-get -y install vim dnsutils curl sudo git && export LANGUAGE=en_US.UTF-8 && export LANG=en_US.UTF-8 && export LC_ALL=en_US.UTF-8 && locale-gen en_US.UTF-8 && sudo apt-get -y install language-pack-id && sudo dpkg-reconfigure locales && curl -sSL https://get.docker.com/ | sh && git clone https://github.com/ab77/netflix-proxy /opt/netflix-proxy && cd /opt/netflix-proxy && ./build.sh -s <your-routed-64-prefix-subnet>`
9. Make sure to record the credentials for the `netflix-proxy` admin site.
10. Point your DNS at the server IP, then go to [this](http://ipinfo.io/) site to make sure your server IP is displayed.
11. Finally, enjoy `Netflix` and others out of region.
12. Enjoy or raise a new [issue](https://github.com/ab77/netflix-proxy/issues/new) if something doesn't work quite right (also `#netflix-proxy` on [freenode](https://webchat.freenode.net/?channels=netflix-proxy)).

### Microsoft Azure (advanced)
The following **has not been tested** and is based on a standard `Ubuntu` image provided by `Microsoft Azure` using `cloud-harness` automation tool I wrote a while back and assumes an empty `Microsoft Azure` subscription.

1. Head over to [Microsoft Azure](https://azure.microsoft.com/en-gb/) and sign-up for an account.
2. Get [Python](https://www.python.org/downloads/).
3. On your workstation, run `git clone https://github.com/ab77/cloud-harness.git /opt/cloud-harness`.
4. Follow `cloud-harness` [Installation and Configuration](https://github.com/ab77/cloud-harness#installation-and-configuration) section to set it up.
5. [Create](https://github.com/ab77/cloud-harness#create-storage-account-name-must-be-unique-as-it-forms-part-of-the-storage-url-check-with---action-check_storage_account_name_availability) a storage account.
6. [Create](https://github.com/ab77/cloud-harness#create-a-new-hosted-service-name-must-be-unique-within-cloudappnet-domain-check-with---action-check_storage_account_name_availability) a new hosted service.
7. [Add](https://github.com/ab77/cloud-harness#add-x509-certificate-containing-rsa-public-key-for-ssh-authentication-to-the-hosted-service) a hosted service certificate for SSH public key authentication
8. [Create](https://github.com/ab77/cloud-harness#create-a-reserved-ip-address-for-the-hosted-service) a reserved ip address.
9. [Create](https://github.com/ab77/cloud-harness#create-virtual-network) a virtual network.
10. [Create](https://github.com/ab77/cloud-harness#create-a-new-linux-virtual-machine-deployment-and-role-with-reserved-ip-ssh-authentication-and-customscript-resource-extensionn3) a `Ubuntu 14.04 LTS` virtual machine as follows:

```
    ./cloud-harness.py azure --action create_virtual_machine_deployment \
    --service <your hosted service name> \
    --deployment <your hosted service name> \
    --name <your virtual machine name> \
    --label 'Netflix proxy' \
    --account <your storage account name> \
    --blob b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04-LTS-amd64-server-20140414-en-us-30GB \
    --os Linux \
    --network VNet1 \
    --subnet Subnet-1 \
    --ipaddr <your reserved ipaddr name> \
    --size Medium \
    --ssh_auth \
    --disable_pwd_auth \
    --verbose
```

11. Use the [Azure Management Portal](https://manage.windowsazure.com/) to add `DNS (UDP)`, `HTTP (TCP)` and `HTTPS (TCP)` endpoints and secure them to your home/work/whatever IPs using the Azure `ACL` feature.
12. SSH to your VM as `azureuser` using custom public TCP port (not `22`) and use any non-root user Ubuntu instructions to build/install `netflix-proxy`.

### Automated Tests
I've linked this project with `Travis CI` to automatically test the build. The helper Python script `__testbuild.py` now runs automatically after every commit. This script deploys a test `Droplet` and then runs a serious of tests to verify (a) that both `Docker` containers start; and (b) the `built.sh` script outputs the correct message at the end. The test `Droplet` is destroyed and the end of the run.

The `__testbuild.py` script can also be used to programatically deploy `Droplets` from the command line as follows:

	python ./__testbuild.py digitalocean --api_token abcdef0123456789... --fingerprint 'aa:bb:cc:dd:...' --region 'abc1'
	
* `--api_token abcdef0123456789...` is your `DigitalOCean` API v2 token, which you can generate [here](https://cloud.digitalocean.com/settings/applications).
* `--fingerprint aa:bb:cc:dd:...` are your personal SSH key fingerprint(s) quoted and separated by spaces. You can manage your SSH keys [here](https://cloud.digitalocean.com/settings/security). If you don't specify a fingerprint, it will default to my test one, which means you **won't** be able to SSH into your `Droplet`.
* `--region abc1` is the region where you want the `Droplet` deployed. The default is `nyc3`, but you can use `--list_regions` to see the available choices.
* `--help` parameter will also list all of the available command line options to pass to the script.

Note, you will need a working `Python 2.7` environment and the modules listed in `requirements.txt` (run `pip install -r requirements.txt`).

### IPv6 and Docker
This solution uses IPv6 downstream from the proxy to unblock IPv6 enabled providers, such as Netflix. No IPv6 support on the client is required for this to work, only the VPS must public IPv6 connectivity. You may also need to turn off IPv6 on your local network (and/or relevant devices).[n6]

```
+----------+                  +-----------+                 +-----------------+
|          |                  |           |                 |                 |
|  client  | +--------------> |   proxy   | +-------------> |  Netflix, etc.  |
|          |      (ipv4)      |           |      (ipv6)     |                 |
+----------+                  +-----------+                 +-----------------+
```

When IPv6 public address is present on the host, Docker is configured with public IPv6 support. This is done by assuming the smallest possible IPv6 allocation, dividing it further by two and assigning the second half to the Docker system. Network Discovery Protocol (NDP) proxying is required for this to work, since the second subnet can not be routed[n9]. Afterwards, Docker is running in dual-stack mode, with each container having a public IPv6 address. This approach seems to work in most cases where native IPv6 is used. If IPv6 is provided via a tunnel, Docker subnet can not be reliably calculated and must be specified using `-s` parameter to the `build.sh` script. If IPv6 is not enabled at all, the VPS is built with IPv4 support only.

#### RamNode
RamNode (and any other provider which uses SolusVM as its VPS provisioning system[n10]) assign a `/64` subnet to the VPS, but don't route it. Instead, individual addresses must be added in the portal if they are to be used on the host. After speaking with RamNode support, it appears this is a side-effect of MAC address filtering, which prevents IP address theft. This means that even though the subnet can be further divided on the host, only the main IPv6 address bound to `eth0` is ever accessible from the outside and none of the IPv6 addresses on the bridges below can communicate over IPv6 to the outside.

To demonstrate this behavour, follow these steps:
```
IPV6_SUBNET=<allocated-ipv6-subnet> (e.g. 2604:180:2:abc)
IPV6_ADDR=<allocated-ipv6-addr> (e.g. 2604:180:2:abc::abcd)

# re-configure eth0
ip -6 addr del ${IPV6_ADDR}/64 dev eth0
ip -6 addr add ${IPV6_ADDR}/80 dev eth0

# install Docker
apt-get update && apt-get -y install vim dnsutils curl sudo git && curl -sSL https://get.docker.com/ | sh

# update DOCKER_OPTS
# for upstart based systems (e.g. Ubuntu):
printf "DOCKER_OPTS='--ipv6 --fixed-cidr-v6=\"${IPV6_SUBNET}:1::/80\"'\n" > /etc/default/docker && \
  service docker restart

# -- OR --

# for systemd based systems (e.g. Debian):
# change "ExecStart" in /lib/systemd/system/docker.service to:
ExecStart=/usr/bin/docker daemon -H fd:// --ipv6 --fixed-cidr-v6="${IPV6_SUBNET}:1::/80"

systemctl daemon-reload && \
  systemctl docker restart


# verify IPv6 configuration inside Docker containers
docker run -it ubuntu bash -c "ip -6 addr show dev eth0; ip -6 route show"

# test (this will fail)
docker run -it ubuntu bash -c "ping6 google.com"
```

However, if we NAT all IPv6 traffic from this host using `eth0`, communication will be allowed:
```
# NAT all IPv6 traffic behind eth0
ip6tables -t nat -A POSTROUTING -o eth0  -j MASQUERADE

# test (this will succeed)
docker run -it ubuntu bash -c "ping6 google.com"
```


### Further Work
This solution is meant to be a quick and dirty (but functional) method of bypassing geo-restrictions for various services. While it is (at least in theory) called a `smart DNS proxy`, the only `smart` bit is in the `zones.override` file, which tells the system which domains to proxy and which to pass through. You could easilly turn this into a `dumb/transparent DNS proxy`, by replacing the contents of `zones.override` with a simple[n4] statement:

    zone "." {
        type master;
        file "/data/conf/db.override";
    };

This will in effect proxy every request that ends up on your VPS if you set your VPS IP as your main and only DNS server at home. This will unfortunately invalidate the original purpose of this project. Ideally, what you really want to do, is to have some form of DNS proxy at home, which selectively sends DNS requests to your VPS only for the domains you care about (i.e. netflix.com) and leaves everything else going out to your ISP DNS server(s). [Dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) could be used to achieve this, in combination, perhaps, with a small Linux device like Raspberry Pi or a router which can run OpenWRT.

There is a [similar](https://github.com/trick77/dockerflix) project to this, which automates the Dnsmasq configuration.

If your client is running OS X, you can skip dnsmasq and simply redirect all DNS requests for e.g. `netflix.com` to your VPS IP by creating a file at `/etc/resolver/netflix.com` with these contents:

    nameserver xxx.yyy.zzz.ttt

replacing `xxx.yyy.zzz.ttt` with your VPS IP, of course.

### Contributing
If you have any idea, feel free to fork it and submit your changes back to me.

### Donate
If you find this useful, please feel free to make a small donation with [PayPal](https://www.paypal.me/belodetech) or Bitcoin.

| Paypal | Bitcoin |
| ------ | ------- |
|<center>[![](https://www.paypalobjects.com/en_GB/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=5UUCDR8YXWERQ)</center>|<center>![91c446adbd54ef84eef1c6c1c723586aa0ba85d7](https://raw.githubusercontent.com/ab77/netflix-proxy/master/static/bitcoin_qr.png)<br />91c446adbd54ef84eef1c6c1c723586aa0ba85d7</center>|

[![ab1](https://avatars2.githubusercontent.com/u/2033996?v=3&s=96)](http://ab77.github.io/)

#### Footnotes
[n1] https://github.com/dlundquist/sniproxy by Dustin Lundquist `dustin@null-ptr.net`

[n2] `Hulu` is heavily geo-restricted from most non-residential IP ranges and doesn't support IPv6.

[n3] You can now specify your home/office/etc. IP manually using `-c <ip>` option to `build.sh`.

[n4] See, serverfault [post](http://serverfault.com/questions/396958/configure-dns-server-to-return-same-ip-for-all-domains).

[n5] See, this [issue](https://github.com/ab77/netflix-proxy/issues/42#issuecomment-152128091).

[n6] If you have a working IPv6 stack, then your device may be preferring it over IPv4, see this [issue](https://forums.he.net/index.php?topic=3056).

[n7] See, https://en.wikipedia.org/wiki/Server_Name_Indication.

[n8] See, https://www.reddit.com/r/VPN/comments/48v03v/netflix_begins_geo_checks_on_cdn/.

[n9] See, [Using NDP proxying](https://docs.docker.com/engine/userguide/networking/default_network/ipv6/). Both the caching resolver and Docker dual-stack support are disabled by default due to differences in IPv6 configurations provided by various hosting providers (i.e. RamNode).

[n10] See, http://www.webhostingtalk.com/showthread.php?t=1262537&p=9157381#post9157381.

[n11] See, [https://www.facebook.com/GetflixAU/posts/650132888457824](https://www.facebook.com/GetflixAU/posts/650132888457824), [Netflix Geoblocking - Part 2](http://forums.whirlpool.net.au/forum-replies.cfm?t=2508180#r5) and read [How Netflix is blocking VPNs](http://www.techcentral.co.za/how-netflix-is-blocking-vpns/63882/) and [Wiki](https://github.com/ab77/netflix-proxy/wiki/On-how-Netflix-enforces-geographical-boundaries-in-the-Information-Age..).
