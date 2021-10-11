# DEV env
## on local Unix base machine
## Pre-requisites: nginx, Node14+, git, curl, make
## installation
cd ~

clone jitsi-meet

````
cd ~
git clone https://github.com/jitsi/jitsi-meet.git
cd ~/jitsi-meet/
npm update && npm install
````


clone lib-jitsi-meet

````
cd ~
git clone https://github.com/jitsi/lib-jitsi-meet.git
cd ~/lib-jitsi-meet
npm update
````

Remove jitsi-meet lib-jitsi-meet package
```sudo rm -R ~/jitsi-meet/node_modules/lib-jitsi-meet```


modify jitsi-meet's package.json
````
...
"lib-jitsi-meet": "file:../lib-jitsi-meet",
...
````

## run dev
````
cd ~/jitsi-meet
export WEBPACK_DEV_SERVER_PROXY_TARGET=https://jitsi.elitelearning.vn
npm install lib-jitsi-meet --force && make dev
````


## product
### modify nginx root to jitsi-meet repository
```root /home/vo.luan/jitsi-meet```
### build
````
cd ~/jitsi-meet
npm install lib-jitsi-meet --force && make
````

## Error
if came accross "System limit for number of file watchers reached"
```echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p```
