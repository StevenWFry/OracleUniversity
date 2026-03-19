## Lesson 3 - Preparing for Deployment (in which the appliance is still in a box, but the paperwork has already started demanding attention)

Welcome to the deployment-prep module, where Oracle gently reminds you that before you rack the shiny hardware, you must first survive support registration, shipping logistics, cabling expectations, and tool requirements. In other words: the appliance may be engineered, but the preparation still involves humans.

By the end of this lesson, you should be able to:

- Register Oracle Database Appliance (ODA) with My Oracle Support using the correct support identifiers
- Download the appropriate software bundle for your intended deployment model before the system arrives
- Recognize how different ODA models are shipped and what hardware arrives together versus separately
- Prepare the site for ODA power, cooling, space, and environmental requirements
- Identify the basic tools and equipment needed for installation
- Understand the rack requirements and the high-level rack-mount sequence for shelves and server nodes
- Identify the required cabling for single-node and HA ODA systems, including power, storage, ILOM, and public-network connections
- Understand which additional deployment topics follow next, including power-up, network plumbing, ILOM, and deployment-type selection

---

## 1. What This Module Covers (the calm administrative prequel before anything actually gets bolted into a rack)

This deployment-preparation module introduces the work that comes before the fun part of actually provisioning the appliance.

Topics called out in the lesson introduction include:

- Registering the ODA with Oracle Support
- Preparing the site and rack for the appliance
- Connecting to the ODA network and reviewing power cabling
- Powering up the ODA and plumbing the network
- Configuring ILOM
- Identifying the deployment type

This part of the lesson focuses on the earliest steps: support registration, software download, shipment expectations, and install tools.

---

## 2. Registering the ODA with My Oracle Support (because nothing says "enterprise readiness" like entering identifiers into a portal)

When you purchase ODA, Oracle provides:

- A hardware Customer Support Identifier (CSI)
- A software Customer Support Identifier (CSI)

The next steps are:

1. Add the hardware CSI to your My Oracle Support profile
2. Add the software CSI to your My Oracle Support profile
3. Download the latest software before the appliance arrives

Why this matters:

- You want support access ready before deployment day
- You want the current software bundle available before the hardware lands in your data center like an expensive surprise

This is one of those tasks that feels bureaucratic right up until you skip it and discover that blocked downloads are an awful way to begin an implementation.

---

## 3. Downloading the Right Software Bundle (bare metal and KVM are not the same thing, despite everyone's optimism)

Before the system arrives, you should review the Oracle Database Appliance release notes and download the correct software bundle for your intended deployment style.

Use the Oracle Database Appliance release notes for:

- Oracle Database Appliance bare metal patches
- Oracle Database Appliance KVM patches

The point is simple:

- If you plan a bare metal deployment, get the bare metal files
- If you plan a virtualized deployment, get the KVM-related files that match that configuration

This is not the moment for improvisation. "We'll just figure out which bundle we need after the pallet shows up" is exactly how preparation becomes delay.

---

## 4. What Arrives When the Hardware Ships (the appliance logistics episode no one puts on the marketing slide)

The shipping details vary by model.

For the smaller single-server platforms:

- The X11 small and X11 large models ship on a single pallet

For high-availability systems:

- Server node 0 and server node 1 arrive together on their own pallet
- The storage shelf ships separately on its own pallet
- Any storage expansion shelf also ships separately

Included with the appliance hardware:

- Rack-mounting hardware
- Cables
- Labels
- A setup poster
- A booklet tailored to the specific hardware model

Important shipping caveat:

- Power cords ship separately
- Power cord shipment depends on the destination country

Translation: do not assume "the appliance has arrived" means every required piece is physically in the same pile. Enterprise shipping loves a staggered entrance.

---

## 5. Site Power and Environmental Requirements (because the data center also gets a vote)

Before you rack the appliance, the site itself has to meet Oracle's electrical, thermal, and environmental requirements.

The lesson calls out power planning first:

- Each X11 server uses two hot-swappable, redundant power supplies
- For X11 servers, the rated maximum output is `1400W` per power supply at `200-240 VAC`
- For HA systems, you also need to account for the storage shelf and any expansion shelf in the power budget

This is the point where Oracle is quietly begging you not to treat "it powers on" as the same thing as "the site is correctly designed."

Environmental requirements called out for the X11 family include:

- Operating temperature: `5 C to 35 C`
- Optimal temperature: `21 C to 23 C`
- Operating humidity: `10% to 90%` relative humidity, noncondensing
- Operating altitude: up to `3000 m`, with temperature derating above `900 m`

Acoustic-noise guidance also matters here, which is Oracle's way of reminding you that enterprise hardware does not believe in indoor voices.

Practical rule:

- Check the Oracle Database Appliance Owner's Guide for the exact model-specific electrical, thermal, humidity, altitude, and acoustic figures before finalizing the site

That advice is not decorative. It is the difference between "deployment prep" and "why is facilities suddenly in this meeting?"

---

## 6. Physical Specifications and Rack Space (because 2U plus 4U plus optimism is not a sizing strategy)

There are a few physical facts you want in your head before anyone touches a rack.

At a high level:

- X11-S and X11-L are each single `2RU` servers
- X11-HA uses two `2RU` server nodes plus one `4RU` storage shelf
- That means HA needs `8RU` without a storage expansion shelf
- HA needs `12RU` if you include the optional `4RU` storage expansion shelf

Important planning note from the lesson:

- If you expect to add the storage expansion shelf later, reserve that extra `4RU` in advance

The physical-install kit components you should be aware of include:

- Rack-mount slide rails
- Cable-management hardware
- Model-specific install material

This is not glamorous, but it is operationally useful. Hardware deployments go much better when everyone knows the box dimensions and rack footprint before they start improvising with tape measures and bad optimism.

---

## 7. Rack Requirements and Placement Order (the rack does not care how urgent your project is)

Oracle requires a supported four-post rack.

Key rack points:

- Four-post rack is required
- Two-post racks are not supported
- Supported rack types include square-hole or supported threaded round-hole racks
- For X11 rack hardware, Oracle calls out `9.5 mm` square-hole racks and supported round-hole/threaded rack options

For HA deployments, Oracle's guidance is very clear:

- Stabilize the rack first
- Deploy anti-tilt legs or anti-tilt bars
- Mount hardware from the bottom up
- Install storage shelves below the server nodes to reduce tipping risk

The lesson's physical layout is:

1. Storage expansion shelf at the bottom, if present
2. Main storage shelf above that
3. Server node 0
4. Server node 1

Additional prep rules called out:

- Ensure AC supply circuits are rated for the maximum power your exact system can draw
- Apply identification labels to each unit

Translation: this is not IKEA. The order matters, the stability matters, and the rack absolutely will punish improvisation.

---

## 8. High-Level Rack-Mount Sequence (the part where the diagrams stop being decorative)

The lesson walks through the physical rack-mount process at a high level.

For the storage shelf, the sequence is:

1. Install the left and right storage-shelf rails
2. Slide the shelf into the rack cabinet
3. Secure the rear of the shelf with the appropriate rear locking hardware

For the X11 server nodes, the sequence is:

1. Install the mounting brackets on the server
2. Install the side-rail assemblies in the rack
3. Route or prepare AC power as required by the rail/arm layout
4. Slide the server into the rack on the rail assemblies
5. Install the cable management arm when used

The diagrams matter here, but the most important study takeaway is the order and dependency chain:

- Shelves first
- Server brackets next
- Rails next
- Server insertion next
- Cable-management hardware last

That is a much better memory anchor than trying to memorize every arrow in every installation figure like you are cramming for an exam in industrial choreography.

---

## 9. Tools and Equipment You Need (the glamorous world of screwdrivers and not dropping expensive gear)

Oracle calls out a short but important tool list:

- A Phillips #2 screwdriver at least four inches long
- A T20 Torx driver if you are using a threaded rack
- A mechanical lift, which is recommended

That last item is Oracle's gentle way of saying, "please do not try to manhandle enterprise hardware into a rack with optimism and lower-back regret."

---

## 10. Cabling Preparation (the rainbow of enterprise hardware chaos)

Once the hardware is mounted, the next step is cabling.

The lesson's first rule is refreshingly sensible:

- Identify all required cables before you begin
- Verify the bill of materials (BOM) for your exact configuration
- Confirm whether your setup needs SAS, SFP28 fiber, TwinAx, RJ-45, or additional expansion-shelf cabling

For the HA storage environment, the lesson diagrams emphasize color-coded SAS cables:

- Dark blue
- Light blue
- Dark red
- Light red

The diagrams also use:

- Green and yellow for fiber/SFP28 paths
- Black for power cables

This is one of those rare moments where color coding is not decorative. It is Oracle's attempt to stop you from turning storage cabling into modern art.

---

## 11. What Ships Versus What You Still Need (because enterprise hardware never misses a chance to make cabling conditional)

The appliance includes core system cabling, but not every network cable you may need.

What the course calls out as supplied:

- Power cables that ship with the system hardware package
- SAS cables for the HA storage shelf layout
- Additional storage-related cables for the optional storage expansion shelf

What the lesson specifically flags as not supplied:

- `1000Base-T` RJ-45 network cables for Oracle ILOM / `NET MGT`
- `10GbE` RJ-45 public-network cables

For SFP28-based networking, Oracle's current X11 guidance also expects you to use the correct transceivers and appropriate optical or TwinAx cabling for your switch design.

So the real rule is:

- Do not assume the rack is cable-complete just because the appliance has arrived

That assumption is how deployment days become scavenger hunts.

---

## 12. Cabling X11-S and X11-L (the single-node version of "plug in the obvious things, but correctly")

For the X11-S and X11-L models, the cabling sequence is straightforward:

1. Connect power to the server power supplies
2. Connect the Oracle ILOM management port (`NET MGT`)
3. Connect the selected public-network ports

Important details:

- Each power supply should be connected to AC power
- For redundancy, connect the two power supplies to separate AC sources when available
- The `NET MGT` port is the dedicated Oracle ILOM management interface
- Public-network connections depend on the NIC option you ordered

For public networking on X11-S and X11-L, Oracle supports either:

- `10GBase-T` copper networking using RJ-45 ports and Cat-6 cabling
- `10/25GbE` SFP28 networking using the proper transceivers and fiber or TwinAx cables

Translation: the single-node cabling is simple, but "simple" still depends on whether you bought copper, fiber, or both and whether anyone remembered to order the right transceivers.

---

## 13. Cabling X11-HA (where the appliance becomes an actual topology instead of a single box)

The HA configuration adds shared storage and node-to-node topology, so the cabling gets more interesting very quickly.

High-level HA cabling elements include:

- Two server nodes
- One shared storage shelf
- Optional storage expansion shelf
- Public network connections
- Oracle ILOM management connections
- Interconnect/network paths depending on the selected adapters

The storage shelf cabling uses the four color-coded SAS cables:

- Dark blue
- Light blue
- Dark red
- Light red

Oracle's X11 documentation maps those cables between specific SAS ports on the compute nodes and the storage shelf or expansion shelf. The lesson depends on diagrams for the exact port-to-port mapping, so the study-note takeaway is:

- Match the SAS cable color to the matching color-coded port
- Follow the documented node-to-shelf order exactly
- Verify the latest BOM and X11 deployment guide before connecting anything

The HA diagrams in the lesson also show green and yellow SFP28 paths for network connectivity. In practice, the exact public-network and interconnect wiring depends on the network option cards and deployment design, so use the X11 deployment guide for the final port map instead of trusting your memory under fluorescent lights.

---

## 14. Power, ILOM, and First Network Connections (the part where the appliance finally starts acting like infrastructure)

The lesson's power and network bring-up rules are:

- Connect AC power to the storage shelf first for HA systems
- Connect AC power to the server-node power supplies
- For redundancy, ensure each component has one power supply connected to a separate AC source
- Plug in a network cable to the Oracle ILOM management port
- Connect the public-network ports last

Additional notes called out:

- On `Node0`, a USB peripheral connection is optional
- Port assignments can vary depending on the ordered network options

Practical sequence to remember:

1. Storage power first on HA
2. Server power next
3. ILOM management network next
4. Public network after that

That order matters because Oracle expects you to establish management access cleanly before you start pretending the appliance is already provisioned.

---

## 15. Powering Up X11-S and X11-L (standby mode first, panic later)

Before powering on a single-node `X11-S` or `X11-L`, review the power-panel figures in the Oracle documentation so you know exactly which LEDs and buttons you are staring at.

The startup sequence is:

1. Confirm the green `SP OK` LED is steady `ON`
2. Confirm the green `Power/OK` LED is flashing slowly
3. Recognize that this state means the server is in standby power mode
4. Press the power button once to apply full power
5. Wait for the green `Power/OK` LED to turn steadily `ON`

Important behavior:

- The `Power/OK` LED may blink for several minutes during startup
- Do not repeatedly press the power button
- A slowly flashing `Power/OK` LED is not a failure condition here; it is the appliance waiting for full power-on

This is one of those moments where impatience adds exactly zero value and can absolutely make things worse.

---

## 16. Powering Up X11-HA and Plumbing the First Network (the clustered version of "wait your turn")

For `X11-HA`, power order matters more because the storage shelves must be fully online before the host nodes come up.

Recommended sequence:

1. Power on the storage shelf by plugging power into each power supply
2. Power on the optional storage expansion shelf, if present
3. Wait until each shelf shows a steady green `Power/OK` LED
4. Confirm the green `SP OK` LED is steady `ON` on each host node
5. Press the power button on each host node once
6. Wait for each node's green `Power/OK` LED to turn steadily `ON`

Important behavior:

- Shelf startup can take several minutes, especially with more drives installed
- Do not power on the host nodes before the storage shelves are fully ready
- Do not repeatedly press the host power buttons

For initial network plumbing:

- Oracle ILOM uses the dedicated `NET MGT` connection for out-of-band management
- During the first startup, the appliance detects the initial public network based on which public-network ports are physically connected
- On `X11-HA`, the dual-port `10/25GbE` SFP28 adapter is used for the cluster interconnect

Practical takeaway:

- Connect the intended public-network ports before first power-on
- Bring up `NET MGT` cleanly so Oracle ILOM is reachable
- Let the appliance discover the initial network layout instead of changing cables mid-bring-up like a maniac

Once the first public network is configured, the rest of deployment can continue locally on the appliance or from a remote system.

---

## 17. Oracle ILOM (the computer inside the computer, which sounds fake until it saves your weekend)

Oracle Integrated Lights Out Manager, or Oracle ILOM, is the appliance's embedded service processor.

Its job is to manage ODA independently of the host operating system.

That matters because Oracle ILOM can still help when the main OS is unavailable, half-booted, misconfigured, or otherwise having what professionals call "a situation."

Core responsibilities include:

- Out-of-band management independent of the database host OS
- Power control and restart options
- Hardware and environmental monitoring
- Fault isolation
- Remote console access
- Alerting through mechanisms such as SNMP traps, IPMI PETs, and remote syslog

In practical terms, Oracle ILOM is how you keep administrative control when the appliance itself is not in the mood to cooperate.

---

## 18. Ways to Configure Oracle ILOM (choose your preferred flavor of controlled inconvenience)

There are several ways to get Oracle ILOM configured and reachable.

**DHCP-based access**

- The `NET MGT` port is a dedicated `1 GbE` management interface for Oracle ILOM
- By default, it is configured for DHCP
- Once the DHCP server assigns an address, you can browse to the Oracle ILOM web interface and log in

**Static IP configuration through the Oracle ILOM CLI**

- Connect through the current DHCP-assigned ILOM address
- Log in as `root`
- Move to `/SP/network`
- Set the pending IP address, netmask, and gateway
- Commit the changes and verify the network settings

**IPMI tool configuration**

- Oracle also supports assigning Oracle ILOM network settings with `ipmitool`
- This is useful when you want a scripted or lower-level path instead of relying on the browser UI

**Serial connection**

- You can connect to the `SER MGT` port from a laptop or terminal session
- For ODA bring-up, the serial speed must match `115200` baud
- The serial-console settings typically follow `8N1`: eight data bits, no parity, one stop bit
- Disable hardware flow control (`CTS/RTS`)
- Disable software flow control (`XON/XOFF`)

The high-level rule is simple:

- If DHCP works, Oracle ILOM setup is easier
- If DHCP does not work, serial and CLI methods become your emergency exits

---

## 19. Plumbing the First Network (because the browser UI is useless until the network stops being theoretical)

Before you can use the Browser User Interface, you must plumb the first public network.

Oracle's bare metal provisioning flow uses the first-network configuration to enable the appliance software to continue deployment.

The working sequence is:

1. Connect through the Oracle ILOM remote console
2. Log into Oracle Database Appliance as `root`
3. Run `odacli configure-firstnet`
4. Complete the prompted network settings for the public interface
5. Provide the IP address, netmask, and gateway information

For the command itself, current X11 documentation uses:

```bash
/opt/oracle/dcs/bin/odacli configure-firstnet
```

On high-availability systems, Oracle's detailed plumbing procedure has you run the first-network configuration on both nodes.

This step must happen before you start using the Browser User Interface, because until the public network is plumbed, the appliance is basically an expensive promise with LEDs.

---

## 20. Bonding and LACP Choices (the networking decision you do not want to "figure out later")

During `odacli configure-firstnet`, you can choose whether to:

- Bond ports on the same network card
- Bond across supported network PCI cards of the same type
- Enable Link Aggregation Control Protocol (`LACP`) on the bonding interface

Important operational notes:

- On current Oracle guidance for `X9-2`, `X10`, and `X11`, `odacli configure-firstnet` supports bonding across two supported network PCI cards of the same type
- If you enable `LACP`, Oracle sets `lacp_rate` to `1` (`fast`)
- Your switch configuration must support that `lacp_rate`

The "do not wing this" warning:

- Bonding is a deployment-time design choice
- If you need to change the default network bonding later, current Oracle guidance says to run `cleanup.pl -cleanDefNet` and then rerun `odacli configure-firstnet`

That is still vastly less fun than choosing correctly the first time.

---

## 21. Deployment Type Checkpoint (decide what you are building before the appliance starts making assumptions)

Before you proceed into full provisioning, be clear about the deployment type.

At this stage, the big decisions are:

- Single-node versus high-availability platform
- Bare metal versus KVM-based deployment
- Database-only versus broader VM or DB system usage

Why this matters:

- It affects the software bundle you downloaded
- It affects network design choices
- It affects how you will use the Browser User Interface and `odacli`
- It affects whether you are building a straightforward bare metal appliance or a virtualized platform with DB systems and application VMs

This is not a cosmetic checkbox. It is the moment where "an appliance" becomes your specific appliance.

---

## 22. Key Takeaways (the checklist before the checklist)

- Register both the hardware and software CSIs in My Oracle Support
- Download the latest ODA software bundle before the hardware arrives
- Use the correct files for the intended deployment model, especially bare metal versus KVM
- Expect different shipping layouts for single-node and HA systems
- Validate site power, cooling, humidity, altitude, and acoustic requirements before installation
- Plan rack space correctly: `2RU` for single-node systems, `8RU` for HA, `12RU` with an expansion shelf
- Use a supported four-post rack and mount HA components from the bottom up
- Identify the required power, management, storage, and public-network cables before starting
- Use the color-coded SAS-cable scheme exactly as documented for HA storage wiring
- Connect redundant power supplies to separate AC sources when available
- Connect `NET MGT` for Oracle ILOM and then attach the public network based on the installed NIC option
- On `X11-S` and `X11-L`, wait for `SP OK` to be steady `ON`, then press the power button once and wait for `Power/OK` to go steadily `ON`
- On `X11-HA`, power the storage shelves first and wait for their `Power/OK` LEDs before powering the host nodes
- Initial public-network detection depends on which public ports are connected at first startup
- Oracle ILOM manages ODA independently of the host operating system and is central to restart, monitoring, alerting, and remote-console access
- Use `odacli configure-firstnet` before the Browser User Interface to plumb the first public network
- If you enable `LACP`, the switch must support `lacp_rate=1`
- On current Oracle guidance, changing default network bonding later requires clearing the default network with `cleanup.pl -cleanDefNet` and rerunning `odacli configure-firstnet`
- Be clear on deployment type before provisioning: single-node or HA, bare metal or KVM
- Power cords may ship separately and are country-dependent
- Have the required install tools ready before rack work begins

---

## 23. Wrap-Up (the appliance is still not running, but at least the chaos is now organized)

You now have the first layer of deployment prep: support registration, software readiness, shipping expectations, site requirements, rack planning, installation tools, cabling expectations, initial power-on behavior, Oracle ILOM access, and first-network configuration. From here, the appliance is finally ready to move from carefully staged hardware into an actual deployment.
