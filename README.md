# System requirement
Ubuntu 18.04+, Debian10+

## system update
````
sudo apt update
sudo apt upgrade -y
sudo apt install curl wget
````

## java 8 jdk
````
sudo apt-get install -y openjdk-8-jre-headless -y
````

## install nginx
````
sudo apt install nginx-full -y
sudo systemctl enable nginx
sudo systemctl restart nginx
````

## hostname
````
sudo hostnamectl set-hostname <hostname>
````

## modify /etc/hosts
### insert
```
127.0.1.1   <hostname>
<publicIP>  <hostname>
```

## install jitsi meet
````
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list
wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add -
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
sudo sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi.list"
sudo apt-get update -y
sudo apt-get install jitsi-meet -y
````

input hostname when installation

choose self-signed cert

## update firewall rules
````
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow in 10000:20000/udp
sudo ufw enable
````

## Generate a Let's Encrypt certificate
````
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
````

# Authentication
## insert /etc/prosody/conf.avail/[hostname].cfg.lua after line 'VirtualHost "[hostname]"'
````
    authentication = "internal_plain"
````

## insert bottom /etc/prosody/conf.avail/[hostname].cfg.lua
````
VirtualHost \"guest.$(hostname -f)\"
    authentication = "anonymous"
    c2s_require_encryption = false
````

## add anonymousdomain /etc/jitsi/meet/[hostname]-config.js
````
var config = {
    hosts: {
            domain: '<hostname>',
            anonymousdomain: 'guest.<hostname>',
````


## modify /etc/jitsi/jicofo/jicofo.conf
````
jicofo {
  authentication: {
    enabled: true
    type: XMPP
    login-url: <hostname>
 }
````
## create user 
````
sudo prosodyctl register <username> <host> <password>
````

## restart services
````
systemctl restart prosody
systemctl restart jicofo
systemctl restart jitsi-videobridge2
````


# Turn setup
## modify url in external services in /etc/prosody/conf.avail/[hostname].cfg.lua
````
external_service_secret = "<turnserver-secretkey>";
external_services = {
     { type = "stun", host = "<turnserver-url>", port = 3478 },
     { type = "turn", host = "<turnserver-url>", port = 443, transport = "udp", secret = true, ttl = 86400, algorithm = "turn" },
     { type = "turns", host = "<turnserver-url>", port = 443, transport = "tcp", secret = true, ttl = 86400, algorithm = "turn" }
};
````

# Jibri setup
## ALSA and Loopback Device
````
sudo apt install linux-image-extra-virtual
echo "snd-aloop" >> /etc/modules
modprobe snd-aloop
lsmod | grep snd_aloop
````

## Ffmpeg with X11 capture support
````
sudo add-apt-repository ppa:mc3man/trusty-media
sudo apt update
sudo apt install ffmpeg
````

## google chrome stable
````
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add - 
sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
sudo apt update 
sudo apt install google-chrome-stable -y
````

## Chrome driver
````
CHROME_DRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE`
wget -N http://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip -P ~/
unzip ~/chromedriver_linux64.zip -d ~/
rm ~/chromedriver_linux64.zip
sudo mv -f ~/chromedriver /usr/local/bin/chromedriver
sudo chown root:root /usr/local/bin/chromedriver
sudo chmod 0755 /usr/local/bin/chromedriver
````

## Miscellaneous required tools
````
sudo apt-get install default-jre-headless ffmpeg curl alsa-utils icewm xdotool xserver-xorg-video-dummy
````

## install jibri
add jitsi repo (if wasnt added)

````
sudo curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
sudo echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi.list > /dev/null
````
isntall jibri
````
sudo apt update
sudo apt install jibri
sudo usermod -aG adm,audio,video,plugdev jibri
````
# Config jibri

