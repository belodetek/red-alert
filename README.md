# red-alert
> 🐳 low-code open source water leak monitoring with Helium & LoRaWAN - get SMS alerts when water leaks, where it shouldn't..

## try it
> Technically, balenaCloud/balenaOS isn't required, but it's free for the first ten devices and makes things so much easier; alternatively, any Linux machine with `dockerd` and compose tooling installed could be used, but then you'd have to sort out SSL certs, SSH auth, etc.

Required software and hardware:
* [Dragino LWL02 water leak sensor(s)](https://www.dragino.com/products/lorawan-nb-iot-door-sensor-water-leak/item/180-lwl02.html)
* balenaOS [supported](https://docs.balena.io/reference/hardware/devices/) hardware (e.g. Raspberry Pi)
* [balena-cli](https://github.com/balena-io/balena-cli)
* `Bash` shell, `jq` and `envsubst`

### prerequisites
* procure up to ten LW02 sensors
* sign-up to [balenaCloud](https://dashboard.balena-cloud.com/signup)
* start a free trial (~ $15) with Twilio](https://console.twilio.com/) and generate API keys
* subscribe to [Helium Foundation Console](https://docs.helium.com/use-the-network/console/) (10 devices and ~ 250 DCs free) and generate your API key
* in the console, create a new MQTT integration to the free [HiveMQ broker](https://www.hivemq.com/public-mqtt-broker/) <img width="826" alt="image" src="https://user-images.githubusercontent.com/2033996/223318833-db1bf054-6f3a-48bf-9bd8-7453654d0b3b.png">

### deployment
> (e.g.) [Node-RED](https://hub.docker.com/r/nodered/node-red/) on a [balenaFin](https://www.balena.io/fin), which is a [CM3](https://www.raspberrypi.com/products/compute-module-3/) based SBC

    $ set -a

    $ fleet=red-alert
    $ dt=fincm3

    $ git clone https://github.com/belodetek/red-alert.git \
      && cd pushd red-alert

    $ balena login

    $ balena fleet create $fleet --type $dt
    
    $ helium_org_id=$(curl https://console.helium.com/api/v1/organization \
      -H 'Key: {{ helium-api-key }}' | jq -r .id)

    $ balena env add __NODE_RED_BACKUP_EXPORT_JSON__ "$(cat < node-red.json.tmpl | envsubst)"
      --fleet $fleet
      
    $ balena env add TWILIO_ACCOUNT_SID '{{ your-account-sid }}'
      --fleet $fleet

    $ balena env add TWILIO_AUTH_TOKEN '{{ your-auth-token }}'
      --fleet $fleet

    $ balena env add TWILIO_MESSAGING_SERVICE_SID '{{ your-service-sid }}'
      --fleet $fleet

    $ balena env add SMS_ALERT_RCPT_TO '{{ your-cell-number }}'
      --fleet $fleet

    $ balena push $fleet

    $ balena os download $dt \
      --output balena.img \
      --version latest

    $ balena os configure balena.img \
      --fleet $fleet \
      --device-type $dt
      
    # flash appropriate SBC media (i.e. SD card or eMMC) and plug it all in
    $ balena local flash balena.img
    
    # (e.g.) first device only; will also show up in the balenaCloud dashboard 
    $ host=$(sudo balena scan --json | jq -r .[0].host)

    $ balena device public-url ${host/.local/} --enable

    $ uuid=$(balena device ${host/.local/} | grep UUID: | cut -c24-)
    
    # browser window will take you to your Node-RED dashboard
    $ open https://${uuid}.balena-devices.com


### testing/operating

* register your with Helium Foundation Console
* install sensors strategically around water egress areas around your dwelling
* tag/label you devices and link them with the integration <img width="617" alt="image" src="https://user-images.githubusercontent.com/2033996/223322225-c4894e99-a321-4586-be89-7c9c168dfb43.png">
* observe your device logs (Node-RED stdout) to see MQTT messages from HiveMQ come in periodocally (during registration and once every 24h)
* adjust the code to suit your specifc requirements
* export and store it in `__NODE_RED_BACKUP_EXPORT_JSON__ ` balenaCloud fleet env. var. as a compact JSON for a stateless configuration

### next steps
* transfer a used Helium miner off your local marketplace to your wallet, so you don't have to pay for Helium DCs in the long run

😬👍
