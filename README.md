# Reference guides:
- https://tweak-cd.mr-d.nl/eigen-gpon-sfp-op-een-udmpro/
- https://www.techconnect.nl/blog/11327/freedom-internet-icm-gpon-sfp-stick-met-unifi/

These guides are already very good and I used them as a guide. This guide mostly follows these guides, so credits go to the authors. However I found them lacking slightly at some points.

I purchased this stick: https://www.fs.com/de-en/products/133619.html (	GPON ONU Stick with MAC SFP 1310nm-TX/1490nm-RX 1.244G-TX/2.488G-RX Class B+ 20km DOM Simplex SC/APC SMF Optical Transceiver Module (Industrial))

Remember that during the next steps, your internet connection will be down.

First you have to read out the serial number of the Huawei ONT. This requires a direct wire connection to the ONT. Then if I remember correctly you have to give your device a fixed IP (in the 192.168.18.X range). Afterwards you can use your favorite browser to navigate to http://192.168.18.1 (username: Epuser, password: userEp). Navigate to Status > Device Information and look for the SN field. Copy the number that has the HWT prefix, you will need this for the next step.

Now we have the required serial number that Freedom expects, we need to spoof the serial number of our GPON stick. 

The step I was missing in the guides is that in order to connect to the GPON stick, you have to connect to it on IP 192.168.1.10, requiring you to connect the stick to a switch/router port with this subnet. With Unifi you can go to Settings > Networks and press New Virtual Network. Give it the Host Address of 192.168.1.1. I think you can leave everything else on default, don't forget to save your settings.

Note: it was quite some hit or miss in order to connect to the GPON stick successfully. Sometimes I had to reseat the stick or switch to a different port. In the end the connection was OK and I could connect. But be advised that this might be quite finnicky.

Then plug in the GPON stick to an SFP port on your router, and assign this port to your newly created 192.168.1.X network. The GPON stick requires an active fiber in order to come online, so connect your provider's to the stick.
Then also connect your own machine (laptop/phone/potato) to your router and ensure that the connected port is also set to this newly created network. This ensures that both the GPON stick and your machine are on the same network.

Then SSH into the stick:
ssh -oHostKeyAlgorithms=+ssh-dss -oKexAlgorithms=diffie-hellman-group14-sha1 192.168.1.10 -l ONTUSER
The password is 7sp!lwUBz1

Then set the new serial number using:
set_serial_number HWTXXXXXXXX

You can check if you were successful by checking:
fw_printenv | grep nSerial

Which in my case didn't work, it just printed the old serial number.

I tried again using an sfp_i2c command:
sfp_i2c -i8 -s "HWTXXXXXXXXX"

But even then the fw_printenv command gave me the old serial number.

I reseated the GPON stick and reseated the fiber a couple of times in order to connect to it again.

Afterwards I ssh'd to the stick again and attempted a serial number change again:
set_serial_number HWTXXXXXXXX

Which was successful.

Now I tried to switch over my WAN network to the stick (ensure that the IPv4 and vlan settings are set correctly), but this would not succeed. My UDM Pro did not recognise the stick anymore, with or without a fiber attached. Reseating did not help. I decided to give up for a few weeks. Then yesterday I checked the port manager and suddenly saw the GPON stick. I switched the fiber over expecting the internet to drop out, but everything worked flawlessly.
