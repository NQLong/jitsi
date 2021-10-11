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
VirtualHost "guest.$(hostname -f)"
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
````
jibri {
  // A unique identifier for this Jibri
  // TODO: eventually this will be required with no default
  id = ""
  // Whether or not Jibri should return to idle state after handling
  // (successfully or unsuccessfully) a request.  A value of 'true'
  // here means that a Jibri will NOT return back to the IDLE state
  // and will need to be restarted in order to be used again.
  single-use-mode = false
  api {
    http {
      external-api-port = 2222
      internal-api-port = 3333
    }
    xmpp {
      // See example_xmpp_envs.conf for an example of what is expected here
      environments = [
          {
            // A user-friendly name for this environment
            name = "env nqlong"

            // A list of XMPP server hosts to which we'll connect
            xmpp-server-hosts = [ "<hostname>" ]

            // The base XMPP domain
            xmpp-domain = "<hostname>"

            // An (optional) base url the Jibri will join if it is set
            // base-url = "https://<hostname>"

            // The MUC we'll join to announce our presence for
            // recording and streaming services
            control-muc {
                domain = "internal.auth.<hostname>"
                room-name = "JibriBrewery"
                nickname = "jibri-nickname"
            }

            // The login information for the control MUC
            control-login {
                domain = "auth.<hostname>"
                // Optional port, defaults to 5222.
                username = "jibri"
                password = "jibri12340"
            }

            // An (optional) MUC configuration where we'll
            // join to announce SIP gateway services

            // The login information the selenium web client will use
            call-login {
                domain = "recorder.<hostname>"
                username = "recorder"
                password = "recorder12340"
            }

            // The value we'll strip from the room JID domain to derive
            // the call URL
            strip-from-room-domain = "conference."

            // How long Jibri sessions will be allowed to last before
            // they are stopped.  A value of 0 allows them to go on
            // indefinitely
            usage-timeout = 0

            // Whether or not we'll automatically trust any cert on
            // this XMPP domain
            trust-all-xmpp-certs = true
        }
      ]
    }
  }
  recording {
    recordings-directory = "<recording directory>"
    # TODO: make this an optional param and remove the default
    #finalize-script = ""
  }
  streaming {
    // A list of regex patterns for allowed RTMP URLs.  The RTMP URL used
    // when starting a stream must match at least one of the patterns in
    // this list.
    rtmp-allow-list = [
      // By default, all services are allowed
      ".*"
    ]
  }
#   sip {
#     // The routing rule for the outbound scenario in VoxImplant is based on this prefix
#     outbound-prefix = "out_"
#   }
  ffmpeg {
    resolution = "1920x1080"
    // The audio source that will be used to capture audio on Linux
    audio-source = "alsa"
    // The audio device that will be used to capture audio on Linux
    audio-device = "plug:bsnoop"
  }
  chrome {
    // The flags which will be passed to chromium when launching
    flags = [
      "--use-fake-ui-for-media-stream",
      "--start-maximized",
      "--kiosk",
      "--enabled",
      "--disable-infobars",
      "--autoplay-policy=no-user-gesture-required"
    ]
  }
  stats {
    enable-stats-d = true
  }
  webhook {
    // A list of subscribers interested in receiving webhook events
    subscribers = []
  }
  jwt-info {
    // The path to a .pem file which will be used to sign JWT tokens used in webhook
    // requests.  If not set, no JWT will be added to webhook requests.
    # signing-key-path = "/path/to/key.pem"

    // The kid to use as part of the JWT
    # kid = "key-id"

    // The issuer of the JWT
    # issuer = "issuer"

    // The audience of the JWT
    # audience = "audience"

    // The TTL of each generated JWT.  Can't be less than 10 minutes.
    # ttl = 1 hour
  }
  call-status-checks {
    // If all clients have their audio and video muted and if Jibri does not
    // detect any data stream (audio or video) comming in, it will stop
    // recording after NO_MEDIA_TIMEOUT expires.
    no-media-timeout = 30 seconds

    // If all clients have their audio and video muted, Jibri consideres this
    // as an empty call and stops the recording after ALL_MUTED_TIMEOUT expires.
    all-muted-timeout = 10 minutes

    // When detecting if a call is empty, Jibri takes into consideration for how
    // long the call has been empty already. If it has been empty for more than
    // DEFAULT_CALL_EMPTY_TIMEOUT, it will consider it empty and stop the recording.
    default-call-empty-timeout = 30 seconds
  }
}
````


## Configuring Jitsi-meet
### /etc/prosody/prosody.cfg.lua
````
-- internal muc component, meant to enable pools of jibri and jigasi clients
Component "internal.auth.yourdomain.com" "muc"
    modules_enabled = {
      "ping";
    }
    -- storage should be "none" for prosody 0.10 and "memory" for prosody 0.11
    storage = "memory"
    muc_room_cache_size = 1000

VirtualHost "recorder.yourdomain.com"
    modules_enabled = {
        "ping";
    }
    authentication = "internal_plain"

````
### /etc/jitsi/jicofo/sip-communicator.properties
````
org.jitsi.jicofo.jibri.BREWERY=JibriBrewery@internal.auth.yourdomain.com
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
````

### /etc/jitsi/meet/[hostname]-config.js
````
fileRecordingsEnabled: true, // If you want to enable file recording
liveStreamingEnabled: true, // If you want to enable live streaming
hiddenDomain: 'recorder.yourdomain.com',
````

## Setup account for jibri
must match with the one use in jibir.conf
````
prosodyctl register jibri auth.<hostname> <password>
prosodyctl register recorder recorder.<hostname> <password>
````
