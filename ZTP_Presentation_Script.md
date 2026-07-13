# Enterprise Zero-Touch Provisioning — Speaker Script

*11 slides · estimated 10–12 minutes at a natural pace*

---

## Slide 1 — Title: Enterprise Zero-Touch Provisioning

"Good [morning/afternoon] everyone. Today I'm going to walk you through our Enterprise Zero-Touch Provisioning solution — or ZTP for short — for Cisco and Arista switches.

The goal here is simple: take a switch from the moment it's racked, to the moment it's fully configured and production-ready, with zero manual configuration. We do that by integrating seven systems — Rack Builder, ServiceNow, Infoblox DDI, DHCP, Python bootstrap, Git, and Ansible Automation Platform, or AAP.

The whole workflow breaks down into two phases: Phase 1, Pre-ZTP Preparation, which is the human planning side, and Phase 2, ZTP Execution, which is fully automated. Let's go through both."

---

## Slide 2 — Purpose & Solution Overview

"So what problem are we solving? Traditionally, provisioning a switch means someone physically consoles into it and types configuration by hand — slow, inconsistent, and error-prone at scale.

Our solution splits the work into two clean stages. Provisioning actually begins long before the switch is even powered on — operational teams reserve rack space, register serial numbers, and allocate management IPs ahead of time. Then, once the switch is installed and powered up, it takes over: it retrieves its bootstrap script, validates its hardware and software, upgrades the OS if needed, pulls down its production configuration, runs health checks, and updates our inventory systems automatically.

Seven systems power this end-to-end: Rack Builder, ServiceNow, Infoblox DDI, DHCP, Python, Git, and Ansible AAP — each with a specific job, which we'll get into."

---

## Slide 3 — Two Phases, One Seamless Deployment

"This slide shows the hand-off at a glance. On the left, Phase 1 — Pre-ZTP Preparation — is where we get all our enterprise systems ready *before* the switch is ever connected. Rack Builder reserves the rack position and management port; ServiceNow creates the device record and tracks its lifecycle; Infoblox DDI allocates the management IP and DNS records, and sets DHCP Option 67 or 150 to point at the bootstrap file.

Once that's done, the output is simple: the network infrastructure is fully prepared, and the device is ready for automated provisioning.

On the right, Phase 2 — ZTP Execution — kicks off automatically the second the switch powers on. It boots, discovers DHCP, downloads its bootstrap script, hands off to Python and then AAP for configuration, and finishes with validation and handover. The output there: a switch that's fully configured, validated, and live."

---

## Slide 4 — Pre-ZTP Preparation: Three Systems, Three Outputs

"Let's zoom into Phase 1 a bit more. Three systems do the heavy lifting here, each with its own objective.

**Rack Builder** prepares the physical installation before the switch even arrives — reserving the rack and RU position, the management port, recording the serial number, model and asset tag, and verifying cabling and power are ready. The output is a deployment request that gets forwarded for inventory registration.

**ServiceNow CMDB** is our source of truth for network assets. It creates the device record, stores the serial, vendor, model, site, rack, and owner, associates the change and deployment tickets, and tracks status from Planned through In Progress to Completed. That gives us a validated inventory record, ready for automation.

**Infoblox DDI** prepares networking services ahead of boot — allocating the management IP, creating forward and reverse DNS records, and configuring the DHCP reservation with Option 67 or 150 pointing to the bootstrap URL. The result: the device can obtain everything it needs the moment it boots."

---

## Slide 5 — ZTP Execution: From Power-On to Bootstrap

"Now we move into Phase 2, and this is where it gets fully automated. The moment the switch is powered on, steps one through four happen without any human involvement.

**Step one, Device Boot** — the unconfigured Cisco or Arista switch powers on, detects its management interface, and enters ZTP mode.

**Step two, DHCP Discovery** — it sends a DHCP Discover request, and DHCP responds with the management IP, subnet, gateway, DNS, and critically, Option 67, which points to the bootstrap script.

**Step three, Bootstrap Download** — using that Option 67 URL, the switch securely pulls down the Python bootstrap script from the ZTP server over HTTP or HTTPS.

**Step four, Python Bootstrap** — that script collects hardware inventory — serial, model, OS version, interfaces — validates the software, installs root certificates if needed, and figures out whether an OS upgrade is required."

---

## Slide 6 — ZTP Execution: From Automation to Handover

"Continuing the automated flow, steps five through eight are where Python hands off to Ansible AAP, which does the real configuration work and closes the loop.

**Step five, Automation Handoff** — Python invokes AAP through REST APIs, passing along everything it discovered about the device.

**Step six, Configuration Deployment** — AAP generates a device-specific configuration from Jinja2 templates stored in Git, upgrades software if required, installs certificates, and applies and saves the configuration.

**Step seven, Validation** — automated health checks verify interfaces, routing protocols, LLDP neighbors, port channels, and overall operational status.

**Step eight, Handover** — the results get logged, ServiceNow is updated, monitoring begins, and the switch is officially marked production ready."

---

## Slide 7 — Inside the Automation: Three AAP Jobs, Step by Step

"This next slide goes one level deeper — into what's actually happening inside AAP itself, broken into the three jobs that run behind the scenes.

**Job one, On Boot — POAP.** This is the entry point: the switch checks for DHCP, gets its address and the POAP file link from DHCP options, and starts running poap.py. That script launches the Deploy Config job and waits for it to finish. On success, it downloads the config file from ZTP, copies it to scheduled-config, and applies it. It then launches the Certificate and Validate job — without waiting on a check — and reboots.

**Job two, Deploy Config.** This checks the switch's status in atWork — Staging, Provisioning, or Configuring — and sets it to Provisioning. It checks for the config intent, and for the peer's config intent too, pulls secrets from HCV, downloads and validates the configuration, and copies the configuration intent back to ZTP.

**Job three, Certificate and Validate.** This pulls secrets from HCV, runs an SSH reachability check, validates the upgrade path by comparing current and target versions, performs the upgrade and verifies the install, downloads and installs the certificate through Venafi, verifies the config intent from atWork against the current running config, runs tests, prepares a CSV report, and sends that testing report out.

Together, these three jobs are the engine underneath the 'Configuration Deployment' and 'Validation' steps we just saw."

---

## Slide 8 — Detailed Data Flow

"Now let's trace that same journey as one continuous data flow, ten steps end to end.

Steps one through five cover preparation through fact collection: Rack Builder flows into ServiceNow, into Infoblox, into DHCP; the switch powers on; it does a DHCP Discover using Options 67 and 43; downloads bootstrap.py; and collects switch information.

Steps six through ten cover automation through handover: a REST API call launches the AAP job; it downloads the configuration by serial number from atWork; deploys that configuration; validates it; and the switch is ready to use.

This is really the single thread that ties everything on the previous slides together."

---

## Slide 9 — Component Responsibilities

"It's worth pausing to lay out exactly who owns what, because nine systems touch this workflow across both phases.

In Phase 1: Rack Builder handles planning — reserving rack position, ports, and capturing device info. ServiceNow is our CMDB, maintaining inventory and the deployment lifecycle. Infoblox DDI covers IPAM, DNS and DHCP — allocating IPs, DNS records, and DHCP options.

In Phase 2: the DHCP server provides the IP and bootstrap URL. The ZTP server acts as a file repository, hosting the Python bootstrap scripts and OS images. Python Bootstrap handles discovery — collecting facts, validating the device, and calling AAP. Git is our source control for templates, playbooks, and configurations. Ansible AAP does the heavy automation — upgrading software, deploying config, and validating health. And Monitoring closes the loop, collecting telemetry and ongoing health status.

Every one of these has a single, clear responsibility — that's intentional, and it's what makes the automation reliable."

---

## Slide 10 — Benefits of the Automated Workflow

"So, why does all of this matter? Six outcomes, delivered end to end across both phases.

We get completely automated Day-0 provisioning — no one touching a console. Consistent configuration across every site, because it's all templated. Deployment time drops from hours down to minutes. We maintain a centralized inventory and full audit trail. Software compliance and validation happen automatically, every time. And because the workflow is vendor-independent, it supports both Cisco and Arista out of the box.

This isn't just faster — it's more reliable, more auditable, and it scales the same way whether we're deploying one switch or one thousand."

---

## Slide 11 — Two Phases. One Self-Provisioning Switch.

"To wrap up — this is the whole story on one slide. Human planning establishes identity and reservation; automation takes it the rest of the way to production.

On the human side: Rack Builder reserves the rack, RU and port; ServiceNow records the device as the CMDB source of truth; Infoblox DDI allocates the IP, DNS, and DHCP options; and the device is staged and ready for automated pickup.

On the automated side: the switch boots, discovers DHCP, and downloads bootstrap.py; Python collects facts and calls AAP via REST API; AAP deploys the configuration from Git and Jinja2 templates; and validation runs, followed by handover to production and monitoring.

Two phases, one seamless hand-off, and a switch that provisions itself. Thank you — happy to take any questions."

---

### Delivery notes
- Slides 5–8 are the technical core — slow down here if the audience includes engineers who'll want the detail; you can speed through if it's a leadership audience and lean more on slide 10 (benefits) and slide 11 (summary).
- Slide 7 has the densest content (23 sub-steps across 3 jobs) — consider pausing for questions right after it.
