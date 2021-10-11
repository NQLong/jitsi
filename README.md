# System requirement
Ubuntu 18.04+, Debian10+

# system update
sudo apt update
sudo apt upgrade -y
sudo apt install curl wget

## java 8 jdk
sudo apt-get install -y openjdk-8-jre-headless -y

## install nginx
sudo apt install nginx-full -y
sudo systemctl enable nginx
sudo systemctl restart nginx

## hostname
sudo hostnamectl set-hostname <hostname>

sudo sed -i "1s/^/$(curl -4 icanhazip.com)      $(hostname -f)\n/" /etc/hosts
sudo sed -i "1s/^/127.0.1.1       $(hostname -f)\n/" /etc/hosts


## install jitsi meet
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
sudo sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi.list"
sudo apt-get update -y
sudo apt-get install jitsi-meet -y

#input hostname when installation
#choose self-signed cert

## update firewall rules
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow in 10000:20000/udp
sudo ufw enable



