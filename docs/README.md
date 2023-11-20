# PiHole Install

First, I familiarized myself with what WordPress was via https://kinsta.com/knowledgebase/what-is-wordpress/

Upon realizing that WordPress was more unfamiliar to me than I initially anticipated, as well as in the face of time constraints, I decided to switch to PiHole. An independent study of WordPress will at some point still be conducted, because it seems like a good thing to know. 

Walkthrough at https://pimylifeup.com/pi-hole-docker/ helped in setup/installation, as well as general background on PiHole.

https://github.com/pi-hole/docker-pi-hole also used as reference

## Installing Docker

I downloaded Docker and Docker Compose on my Ubuntu vm using
```bash
sudo apt update
sudo apt install docker.io
sudo apt install docker-compose-plugin

docker compose version # test to verify successful install
```

## Installing PiHole
We will write a "docker-compose" config file, which tells Docker what containers it needs to download and what ports it needs to open

### Create a Directory for Pi-Hole

First, I created a directory to store the configuration file for the Pi-Hole Docker container

```bash
mkdir ~/pihole
cd ~/pihole # move into directory
```

### Write the Docker-Compose Config File

Define the Pi-Hole docker container and the options we want passed to the container

```bash
nano docker-compose.yml
```

enter the following lines in the file:

```
version: "3"

services: 
    pihole:
        container_name: pihole
        image: pihole/pihole:latest
        ports:
            - "53:53/tcp"
            - "53:53/udp"
            - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
            - "80:80/tcp"
        environment:
            TZ: 'America/Chicago' # set timezone
            WEBPASSWORD: 'Brave9t+ad*' # sets a secure password for the Pi-Hole web interface
        # volumes store data between container upgrades
        volumes:
            - './etc-pihole:/etc/pihole'
            - './etc-dnsmasq.d:/etc/dnsmasq.d'
        cap_add:
            - NET_ADMIN # not needed unless using Pi-hole as DHCP server
        restart: unless-stopped
```

### Disable Systemd-Resolve
Pi-Hole will want to operate on the same part the resolve service does, so we need to disable the Systemd-resolved service (used to provide network name resolution).

Small issue solved: systemd-resolve --> systemd-resolved

```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved # prevents from starting again on restart
```

`sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf` disables systemd-resolved's DNS stub resolver, which would otherwise prevent pi-hole from listening on port 53.

`sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'` changes the /etnameserver settings

### Start the Pi-Hole Docker Container
run 'sudo docker compose up -d'

issue: error response from daemon: invalid volume specification: '/home/sysadmin/pihole/etc-dnsmasq.d:etc/dnsmasq.d' mount path must be absolute
solution: fixing the path of the line

## Access the Pi-Hole Web Interface

First, I needed to know my IP Address
'hostname -I'

With my local IP, I input the following into a browser:
```
http://IPADDRESS/admin
```
if port conflict, change port away:
```
http://IPADDRESS:PORT/admin
```

![Pihole Web Interface](docs/assets/Working.png "Pi-hole Web Interface")
