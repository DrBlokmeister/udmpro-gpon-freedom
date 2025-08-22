# Replacing a Huawei ONT with a GPON SFP stick on a UniFi Dream Machine Pro (Freedom Internet / KPN GPON)

> Step-by-step notes from my own Dutch UniFi setup. This mostly follows the two excellent guides below, but fills in a few missing, *finnicky* steps—especially getting management access to the SFP.

## Reference guides
- https://tweak-cd.mr-d.nl/eigen-gpon-sfp-op-een-udmpro/
- https://www.techconnect.nl/blog/11327/freedom-internet-icm-gpon-sfp-stick-met-unifi/

Credits to the authors of those posts—I used them as the base and add the bits I wish I’d known.

## Hardware I used
- **FS GPON ONU Stick with MAC SFP (Industrial)**  
  https://www.fs.com/de-en/products/133619.html  
  *1310 nm TX / 1490 nm RX, Class B+, DOM, simplex SC/APC.*

> ⚠️ Your internet will be **down** during this process.

---

## 1) Read the serial number from the Huawei ONT
You need the ONT’s GPON serial (the one Freedom/KPN expects).

1. Connect a PC directly to the ONT (Ethernet).
2. Give your PC a static IP in **192.168.18.x**.
3. Browse to **http://192.168.18.1**  
   - **Username**: `Epuser`  
   - **Password**: `userEp`
4. Go to **Status → Device Information** and copy the **SN** with the **HWT** prefix (e.g. `HWTXXXXXXXX`). You’ll clone this onto the SFP.

---

## 2) Create a temporary management network for the SFP (192.168.1.0/24)
Most GPON SFP sticks expose a tiny management interface at **192.168.1.10**. Put the stick **and** your client in that subnet.

**UniFi → Settings → Networks → “Create New Network”:**
- Name: `SFP-Management` (or anything)
- Gateway/Subnet: `192.168.1.1/24`
- DHCP: enabled (default is fine)

Then:
- Plug the GPON SFP into a UDM Pro SFP port.
- Assign that switch port the **SFP-Management** network (Port Manager).
- Connect your laptop/phone to another port **also** set to **SFP-Management**.

> ⚠️ Gotcha: the stick often only comes fully up with **live fiber** connected. Attach your provider fiber (SC/APC) while doing this.

---

## 3) SSH into the GPON SFP and clone the serial
Default management IP: **192.168.1.10**

```bash
ssh -oHostKeyAlgorithms=+ssh-dss \
    -oKexAlgorithms=diffie-hellman-group14-sha1 \
    ONTUSER@192.168.1.10
````

**Password:**

```
7sp!lwUBz1
```

Set the serial (replace with your `HWTXXXXXXXX`):

```bash
set_serial_number HWTXXXXXXXX
```

Verify (this may **not** update immediately, see notes below):

```bash
fw_printenv | grep nSerial
```

If needed, try the I²C helper:

```bash
sfp_i2c -i8 -s "HWTXXXXXXXX"
```

Then re-check:

```bash
fw_printenv | grep nSerial
```

> **Reality check:** I had to reseat the SFP and fiber a couple of times before the new serial “stuck”. Don’t panic if `fw_printenv` shows the old value at first—try again, power-cycle, reseat, and re-run `set_serial_number`.

---

## 4) Move WAN to the GPON SFP

Switch your **WAN** to the SFP port and apply the same Internet settings you used with the Huawei ONT:

* **KPN/Freedom GPON** typically uses **VLAN 6** and **PPPoE** (use your Freedom credentials).
* Keep the rest identical to your working ONT setup.

**In my case, the UDM Pro initially didn’t “see” the stick after the change—even with the fiber in. A few reseats later I gave up. **Weeks later**, the Port Manager suddenly showed the GPON stick; I swapped the fiber over and… everything worked flawlessly.**

---

## Troubleshooting notes & “things the guides didn’t say”

* **Management VLAN is key**: You must place both your client and the SFP in **192.168.1.0/24** to reach **192.168.1.10**.
* **Live fiber helps boot**: Some sticks won’t expose management or apply SN reliably until an optical link is present.
* **Be patient with SN changes**: `fw_printenv` may not reflect the new value until a reseat/power-cycle. Re-run `set_serial_number`.
* **Port flakiness**: If the SFP isn’t detected, try another SFP cage or switch port, reseat, and check Port Manager again.
* **Keep WAN identical**: Use the same VLAN/PPPoE/DHCP details that worked with the ONT; only the physical ONT changes.

---

## Why publish this?

I wanted a single, practical checklist that an actual UniFi user (Freedom/KPN in NL) could follow without guesswork. If you spot mistakes or have a different stick/OLT combo, open an issue or PR!

## Acknowledgements

* [https://tweak-cd.mr-d.nl/eigen-gpon-sfp-op-een-udmpro/](https://tweak-cd.mr-d.nl/eigen-gpon-sfp-op-een-udmpro/)
* [https://www.techconnect.nl/blog/11327/freedom-internet-icm-gpon-sfp-stick-met-unifi/](https://www.techconnect.nl/blog/11327/freedom-internet-icm-gpon-sfp-stick-met-unifi/)

````
