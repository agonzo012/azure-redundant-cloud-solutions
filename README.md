# Building Redundant Cloud Solutions in Azure

## What This Is

A multi-region redundant web application in Microsoft Azure, built through
the labITpro hands-on course "Building Redundant Cloud Solutions in Azure"
(29 labs, 9 modules, AZ-104 / AZ-305). The goal is a web app that stays up
even if an entire Azure region goes down, using resources in West US and
East US. All 29 labs live in this one repository.

Course page: https://labitpro.com/hands-on-azure-labs-building-redundant-cloud-solutions/

## Tools Used

- Microsoft Azure (Free Tier)
- Azure Resource Groups
- Azure Virtual Networks and Subnets
- Azure Virtual Machines
- Azure Load Balancers
- Azure Traffic Manager / Front Door (for cross-region routing)

## Labs — in numerical order

### Module 1 — Build Network Infrastructure

**Lab 1 — Deploy a Resource Group in Azure for the Project (done, in the labITpro simulation and in real Azure)**
- Create a resource group to hold the project's resources.
- Created a resource group named **RG-Lab** in **East US**
- Subscription in the simulation: Surf City Boats; in real Azure: Azure subscription 1
- Steps: Resource groups → + Create → name RG-Lab → Review + create → Create
- Result: "Resource group created — Creating resource group 'RG-Lab' in
  subscription 'Surf City Boats' succeeded" (simulation); real Azure confirmed
  "Creating resource group 'RG-Lab' in subscription 'Azure subscription 1' succeeded"
- Screenshot: lab1-resource-group-screenshot.pdf
- Note: a resource group is free.

**Lab 2 — Deploy a Virtual Network and Subnet in West US (done in real Azure)**
- Create a virtual network and subnet in West US for the web app's west-region resources.
- VNet **vNet-West**, RG-Lab, West US, address space **10.0.0.0/16**, subnet **Subnet-West 10.0.1.0/24** (251 available IPs).

**Lab 3 — Deploy a Virtual Network and Subnet in East US (done in real Azure)**
- Create a virtual network and subnet in East US for the web app's east-region resources.
- VNet **vNet-East**, RG-Lab, East US, address space **10.1.0.0/16**, subnet **Subnet-East 10.1.1.0/24** (251 available IPs).

**Lab 4 — Deploy a NAT Gateway in the West US Region (done in real Azure)**
- Deploy a NAT gateway so west-region VMs can reach the internet outbound.
- NAT gateway **NAT-West**, RG-Lab, West US, SKU StandardV2, TCP idle timeout 4 minutes. Public IP **PIP-NATWest** (Static, Standard, IPv4). Associated subnet: vNet-West / Subnet-West (10.0.1.0/24).

**Lab 5 — Deploy a NAT Gateway in the East US Region (done in real Azure)**
- Deploy a NAT gateway so east-region VMs can reach the internet outbound.
- NAT gateway **NAT-East**, RG-Lab, East US, SKU StandardV2, TCP idle timeout 4 minutes. Public IP **PIP-NATEast** (Static, Standard, IPv4). Associated subnet: vNet-East / Subnet-East (10.1.1.0/24).
- The two NAT gateways bill about $0.09/hour combined while they exist; they will be deleted with the RG-Lab resources when the labs are finished.

### Module 2 — Deploy Azure Bastion

**Lab 6 — Connect Virtual Networks with vNet Peering (done in real Azure)**
- Connect the West US and East US virtual networks so they can talk to each other.
- Peering **vNetEast-vNetWest** on vNet-West and reciprocal peering **vNetWest-vNetEast** on vNet-East, both Fully Synchronized and Connected, virtual network access and forwarded traffic enabled in both directions.

**Lab 7 — Perform an Automated Azure Bastion Deployment (done in real Azure)**
- Deploy Azure Bastion automatically for secure RDP/SSH access to the VMs without public IPs.
- Bastion host **vNet-West-bastion**, RG-Lab, West US, SKU Standard, Provisioning Succeeded. The automated flow auto-created the **AzureBastionSubnet** and public IP **vNet-West-IPv4**.
- Cost note: the automated flow fixes the SKU at Standard ($0.29/hour) with no SKU choice, so the free Developer SKU could not be used here. Delete the Bastion host when it is no longer needed to stop the hourly charge.

### Module 3 — Deploy and Configure Load Balancers

**Lab 8 — Deploy and Configure a Public Load Balancer in West US (done in real Azure)**
- Deploy and configure a public load balancer to spread traffic across the west-region VMs.
- Load balancer **LB-West**, RG-Lab, West US, SKU Standard, Type Public. Frontend **FE-LBWest**, public IP **PIP-LBWest** (Static, Standard, IPv4). Backend pool **BE-LBWest** (vNet-West, NIC-based) — WEST01 (Lab 10) and WEST02 (Lab 12) both added. Health probe **HealthHTTP** (HTTP, port 80). Load balancing rule **InboundHTTP** (TCP, frontend port 80 → backend port 80, TCP Reset off).
- Note: Lab 8 does not name the frontend, public IP, backend pool, probe, or rule, so the Lab 9 naming pattern was used for consistency.

**Lab 9 — Deploy and Configure a Public Load Balancer in East US (done in real Azure)**
- Deploy and configure a public load balancer to spread traffic across the east-region VMs.
- Load balancer **LB-East**, RG-Lab, East US, SKU Standard, Type Public. Frontend **FE-LBEast**, public IP **PIP-LBEast** (Static, Standard, IPv4). Backend pool **BE-LBEast** (vNet-East, IP address configuration as the lab specifies) — EAST01 (Lab 14) and EAST02 (Lab 16) both added. Health probe **HealthHTTP** (HTTP, port 80). Load balancing rule **InboundHTTP** (TCP, frontend port 80 → backend port 80, TCP Reset enabled).

### Module 4 — Deploy Resilient Azure VMs in West US

**Lab 10 — Deploy and Configure the WEST01 Virtual Machine (done in real Azure)**
- Deploy and configure the first virtual machine in West US.
- VM **WEST01**, RG-Lab, West US, Running. OS: Windows Server 2019 Datacenter Gen2. Size **Standard_D2s_v6** (2 vCPUs, 8 GiB) at $0.21/hour — the lab's Standard_D2s_v3 is unavailable in West US for this subscription, so the current-generation equivalent was used (same 2 vCPUs and 8 GiB, effectively the same price). Availability set **AvailSet-West** (new). vNet **vNet-West**, subnet **Subnet-West** (10.0.1.0/24); NIC **west01502**, private IP **10.0.1.4**, no NIC-level public IP — the subscription's public IP quota is exhausted, so the VM is reached through the LB-West frontend IP instead. NSG **WEST01-nsg** (new, Advanced) with inbound rules allow RDP (TCP 3389, portal default) and allow HTTP (TCP 80, lab-required). Added to load balancer **LB-West**, backend pool **BE-LBWest**. OS disk Premium SSD, 127 GiB. Admin username **labadmin**.

**Lab 11 — Install IIS on WEST01 Remotely via PowerShell (done in real Azure)**
- Install the IIS web server on WEST01 remotely using PowerShell.
- Run Command (RunPowerShellScript): `Install-WindowsFeature -Name Web-Server -IncludeManagementTools`. Result: Success, no restart needed. W3SVC service Running; Web-Server feature Installed; HTTP GET http://localhost returned 200. Verified publicly: the default IIS welcome page loads at http://172.184.161.146/ — the LB-West frontend IP, proving the load balancer is already serving traffic from WEST01.

**Lab 12 — Deploy and Configure the WEST02 Virtual Machine (done in real Azure)**
- Deploy and configure the second virtual machine in West US.
- VM **WEST02**, RG-Lab, West US, Running. OS: Windows Server 2019 Datacenter Gen2. Size **Standard_D2s_v6** (2 vCPUs, 8 GiB) at $0.21/hour — same D2s_v3 substitution as WEST01. Availability set **AvailSet-West** (existing), fault domain 1 — WEST01 is fault domain 0, so the two VMs sit in different fault domains, which is the point of the availability set. vNet **vNet-West**, subnet **Subnet-West** (10.0.1.0/24); NIC **west02968**, private IP **10.0.1.5**, no NIC-level public IP (public IP quota exhausted). NSG **WEST02-nsg** (new, Advanced) with inbound rules allow RDP (TCP 3389, portal default) and allow HTTP (TCP 80, lab-required). Added to load balancer **LB-West**, backend pool **BE-LBWest**. OS disk Premium SSD. Admin username **labadmin**, same password as WEST01 per the standing rule.

**Lab 13 — Install IIS on WEST02 Remotely via PowerShell (done in real Azure)**
- Install the IIS web server on WEST02 remotely using PowerShell.
- Run Command (RunPowerShellScript): `Install-WindowsFeature -Name Web-Server -IncludeManagementTools`. Result: Success, no restart needed. W3SVC service Running (Automatic); HTTP GET http://localhost returned 200. Verified publicly: the default IIS welcome page loads at http://172.184.161.146/ — the LB-West frontend IP, now serving from both WEST01 and WEST02.

### Module 5 — Deploy Resilient Azure VMs in East US

**Lab 14 — Deploy and Configure the EAST01 Virtual Machine (done in real Azure)**
- Deploy and configure the first virtual machine in East US.
- VM **EAST01**, RG-Lab, East US, Running. OS: Windows Server 2019 Datacenter Gen2. Size **Standard_D2s_v7** (2 vCPUs, 8 GiB) at $0.2240/hour — both the lab's Standard_D2s_v3 and the west fallback Standard_D2s_v6 are unavailable in East US for this subscription, so the next-generation equivalent was used (same 2 vCPUs and 8 GiB). Availability set **AvailSet-East** (new), fault domain 0. vNet **vNet-East**, subnet **Subnet-East** (10.1.1.0/24); NIC **east01565**, private IP **10.1.1.4**, no NIC-level public IP (public IP quota exhausted). NSG **EAST01-nsg** (new, Advanced) with inbound rules AllowHTTP (TCP 80, priority 100) and default-allow-rdp (TCP 3389, priority 1000). Added to load balancer **LB-East**, backend pool **BE-LBEast**. OS disk Standard SSD. Admin username **labadmin**, same password as the west VMs per the standing rule.

**Lab 15 — Install IIS on EAST01 Remotely via PowerShell (done in real Azure)**
- Install the IIS web server on EAST01 remotely using PowerShell.
- Run Command (RunPowerShellScript): `Install-WindowsFeature -Name Web-Server -IncludeManagementTools`. Result: Success, no restart needed. W3SVC service Running; HTTP GET http://localhost returned 200. Verified publicly: the default IIS welcome page loads at http://48.217.132.155/ — the LB-East frontend IP.

**Lab 16 — Deploy and Configure the EAST02 Virtual Machine (done in real Azure)**
- Deploy and configure the second virtual machine in East US.
- VM **EAST02**, RG-Lab, East US, Running. OS: Windows Server 2019 Datacenter Gen2. Size **Standard_D2s_v7** (2 vCPUs, 8 GiB) at $0.2240/hour — matches EAST01 (D2s_v3 and D2s_v6 are unavailable in East US). Availability set **AvailSet-East** (existing), fault domain 1 — EAST01 is fault domain 0, so the two east VMs sit in different fault domains. vNet **vNet-East**, subnet **Subnet-East** (10.1.1.0/24); NIC **east0211**, private IP **10.1.1.5**, no NIC-level public IP (public IP quota exhausted). NSG **EAST02-nsg** (new, Advanced) with inbound rules AllowHTTP (TCP 80, priority 100) and default-allow-rdp (TCP 3389, priority 1000) — matches EAST01-nsg exactly. Added to load balancer **LB-East**, backend pool **BE-LBEast**, which now holds both east VMs. OS disk Standard SSD, 127 GiB (verified on the final Review page). Admin username **labadmin**, same password as the other VMs per the standing rule.

**Lab 17 — Install IIS on EAST02 Remotely via PowerShell (done in real Azure)**
- Install the IIS web server on EAST02 remotely using PowerShell.
- Run Command (RunPowerShellScript): `Install-WindowsFeature -Name Web-Server -IncludeManagementTools`. Result: Success, no restart needed. W3SVC service Running; HTTP GET http://localhost returned 200. Verified publicly: the default IIS welcome page loads at http://48.217.132.155/ — the LB-East frontend IP, now serving from both EAST01 and EAST02.

### Module 6 — Test Load Balancers

**Lab 18 — Ensure Virtual Machine IP Addresses Are Static (done in real Azure)**
- Change the VMs' IP addresses from dynamic to static so load balancer rules stay stable.
- All four NICs changed from Dynamic to Static, keeping the same addresses: WEST01 10.0.1.4, WEST02 10.0.1.5, EAST01 10.1.1.4, EAST02 10.1.1.5. Portal confirmed "Successfully updated network interface" for each, and a final re-check confirmed all four Static.

**Lab 19 — Test Load Balancing Functionality in West US (done in real Azure)**
- Verify the west-region load balancer spreads traffic across WEST01 and WEST02.
- Test page `C:\inetpub\wwwroot\testpage.htm` created on both VMs (shows the server name). LB-West frontend 172.184.161.146: this browser consistently got WEST02, and EAST01 as a second client consistently got WEST01 — both backends served traffic through the same frontend IP. Health status 100%, both instances Up. Failover test: IIS stopped on WEST01, traffic went to WEST02; IIS restarted and confirmed Running.

**Lab 20 — Test Load Balancing Functionality in East US (done in real Azure)**
- Verify the east-region load balancer spreads traffic across EAST01 and EAST02.
- Same test page on both east VMs. LB-East frontend 48.217.132.155: this browser consistently got EAST01, and WEST01 as a second client consistently got EAST02 — both backends served traffic. Failover test: IIS stopped on EAST01, traffic went to EAST02; IIS restarted and confirmed Running. Health status 100%, both instances Up.

### Module 7 — Deploy and Configure Azure Blob Storage

**Lab 21 — Deploy Blob Storage in the West US Region (done in real Azure)**
- Deploy blob storage in West US for the app's files.
- Storage account **surfcitywest999ag**, West US, StorageV2, Standard performance, RA-GRS replication (Standard_RAGRS), Hot access tier, public network access from all networks, blob anonymous access Enabled, storage account key access Disabled, blob versioning Enabled, blob change feed Enabled. Container **images** created with anonymous access level Container (public read). Deployment succeeded Nov 21, 10:50 PM.
- The lab's name surfcitywest999 was already taken globally, so the suffix "ag" was added.

**Lab 22 — Deploy Blob Storage in the East US Region (done in real Azure)**
- Deploy blob storage in East US for the app's files.
- Storage account **surfcityeast999ag**, East US, same settings (StorageV2, Standard, RA-GRS, Hot, public all networks, anonymous access Enabled, key access Disabled), blob versioning Enabled, change feed Disabled (destination). Container **images** created with access level Container. Deployment succeeded Nov 21, 10:55 PM.
- The lab's name surfcityeast999 was already taken globally, so the suffix "ag" was added.

**Lab 23 — Configure Blob Storage Object Replication Between Regions (done in real Azure)**
- Configure object replication so blobs in one region copy to the other.
- Replication policy from surfcitywest999ag/images to surfcityeast999ag/images, Copy over = Everything (the portal's current wording for the lab's "All — New and updated"), 0 filters, tags/priority/metrics not enabled. Policy ID e718197e-2510-4c79-bb12-7ee7e42a07e4; Rule ID 7f87b8db-6e1f-4114-8721-b68c3eb5cc0b.
- Replication verified live: test image lab23-test-image.png uploaded to the West container at 11:06 PM and appeared in the East container at 11:08 PM, about two minutes later.
- The labs assign Storage Blob Data Contributor to "Thomas Mitchell", but no such user exists in the tenant — so the role was assigned to the signed-in account instead on both storage accounts (needed for Entra ID auth uploads, since key access is disabled). If Thomas Mitchell is added to the tenant later, the same role can be assigned to him.

### Module 8 — Deploy and Configure Azure Front Door CDN

**Lab 24 — Deploy and Configure an Azure Front Door CDN Profile** — next
- Deploy and configure an Azure Front Door CDN profile to route users to the nearest region.

**Lab 25 — Add a Second Origin to the Azure Front Door CDN** — next
- Add the second region as another origin so Front Door can fail over between regions.

**Lab 26 — Configure a Custom Domain Name for AFD CDN** — next
- Configure a custom domain name for the Front Door CDN endpoint.

### Module 9 — Deploy and Configure Azure Traffic Manager

**Lab 27 — Create and Configure a Traffic Manager Profile in Azure** — next
- Create and configure a Traffic Manager profile to route traffic between regions.

**Lab 28 — Confirm Functionality of Azure Traffic Manager** — next
- Verify Traffic Manager routes traffic and fails over as expected.

**Lab 29 — Configure a Custom Domain for Traffic Manager** — next
- Configure a custom domain for the Traffic Manager profile.

## Why It Matters

Real enterprise applications can't go down when a region has an outage.
Building the same app in two regions, with traffic routed between them,
is how you keep a service up. This project shows I can design and build
multi-region infrastructure, not just a single VM.

## Cost Rules

- A resource group is free.
- VMs and load balancers bill by the hour, in both regions. Build it,
  screenshot it, then delete the resource group to stop charges.
- After practicing, STOP VMs (don't delete) to save credits when I want
  to keep the work.

## Screenshots

- Lab 1: lab1-resource-group-screenshot.pdf (simulation — RG-Lab in East US)

## Next Steps

- Lab 2 and Lab 3: build the West US and East US VNets and subnets
- Continue through the remaining modules (VMs, load balancers, cross-region routing)
- Recreate each lab in real Azure and screenshot it

## Design Decisions

- **Two regions (West US + East US).** The whole point of the project is surviving a regional outage, so the app lives in two regions and traffic routes between them. One region alone would not prove redundancy.
- **Separate address spaces (10.0.0.0/16 West, 10.1.0.0/16 East).** The VNets must not overlap so they can be peered and so a subnet in one region is never confused with a subnet in the other.
- **One resource group (RG-Lab) for the whole project.** Everything for the redundant app lives in one group, so it can be deleted in a single step to stop all charges when the labs are finished.
- **A NAT gateway in each region.** The VMs get outbound internet through the NAT gateway instead of carrying their own public IPs, which keeps the VM network interfaces private.
- **StandardV2 NAT gateway SKU (portal default).** No premium tier is needed for a lab; the standard SKU covers the outbound traffic the labs generate.
- **VM size D2s_v6 instead of the lab's D2s_v3.** Azure refused D2s_v3 in West US for this subscription, so the current-generation equivalent was used — same 2 vCPUs and 8 GiB of RAM, effectively the same price. The lab's intent (a small general-purpose VM) is unchanged.
- **No public IP on the WEST01 network interface.** The subscription's public IP quota is exhausted (four public IPs already exist), so WEST01 was created without a NIC-level public IP. It is reached through the LB-West frontend IP instead, which matches the design: VMs stay private behind the load balancer, and the NAT gateway handles outbound traffic.
- **Both west VMs in one availability set, in different fault domains.** WEST01 is fault domain 0 and WEST02 is fault domain 1, so a single rack-level failure cannot take both web servers down at once. That is the availability set doing its job.
- **EAST01 uses D2s_v7.** Both D2s_v3 and D2s_v6 are unavailable in East US for this subscription, so the next-generation equivalent was used — same 2 vCPUs and 8 GiB, slightly higher at $0.2240/hour vs $0.1930/hour for D2s_v6. The lab's intent (a small general-purpose VM) is unchanged; the east and west VMs do not need identical sizes.
- **Both east VMs in one availability set, in different fault domains.** EAST01 is fault domain 0 and EAST02 is fault domain 1, mirroring the west pair. Each region now has two web servers that cannot both fall to a single rack-level failure.
- **Static private IPs for all four VMs.** Dynamic addresses could change on reboot and break the load balancer backend mappings, so each NIC was set to Static with the same address it already had.
- **A test page showing the server name.** `testpage.htm` on each VM prints its own computer name, so any request through the load balancer shows exactly which backend VM answered — the simplest way to prove traffic is being distributed.
- **Blob versioning on both accounts, change feed on the source only.** Versioning is required for object replication and is on both storage accounts; the change feed is enabled on the west (source) account and disabled on the east (destination), which is all replication needs.
- **Storage account key access disabled; uploads use Entra ID auth.** With key access off, blob uploads need a role assignment (Storage Blob Data Contributor) rather than a key — a more secure pattern than the lab's key-based default.
- **Storage account names carry an "ag" suffix.** The lab's names surfcitywest999 and surfcityeast999 were already taken globally (storage account names must be unique worldwide), so "ag" was added to both.
- **Both storage accounts are Standard RA-GRS.** The lab text did not state performance or redundancy; the Lab 22 simulation showed Standard performance with geo-redundant storage and read access, so both accounts were created that way.

## What Went Wrong

- **Portal defaulted new subnets to "Private subnet: Enabled."** The labs do not mention this setting. It was left as the default — it creates no extra resource and adds no cost, and the NAT gateways provide the outbound path.
- **Portal defaulted regions to West US 2 / East US 2.** The lab specifies West US and East US, so the exact regions were selected instead of accepting the defaults.
- **vNet-East defaulted to the same address space as vNet-West (10.0.0.0/16).** It was changed to the lab's 10.1.0.0/16 to avoid overlap between the two VNets.
- **The simulation subscription is "Surf City Boats."** The real subscription is "Azure subscription 1" — the real one was used for every resource.
- **The current portal's Add Peering form creates both directions in one operation.** The lab shows two separate steps; the form takes the local and remote link names together, so both peerings were set in one form with no lab value changed.
- **The automated Bastion deployment fixes the SKU at Standard ($0.29/hour).** The lab does not name a SKU, and the automated flow offers no SKU choice. The free Developer SKU is only offered in the VM-connect flow, which needs a VM selected — there are no VMs yet, so it could not be used. The Bastion host will be deleted when it is no longer needed to stop the hourly charge.
- **The portal defaulted LB-West's region to East US.** RG-Lab itself is located in East US, so the load balancer form defaulted there. It was caught before anything was deployed — the backend pool's virtual network list only offered vNet-East — and the region was set to West US explicitly. Nothing was ever deployed in the wrong region.
- **Lab 10's size Standard_D2s_v3 is unavailable in West US for this subscription.** Azure refused it ("NotAvailableForSubscription"). The current-generation equivalent Standard_D2s_v6 was used instead — same 2 vCPUs and 8 GiB, effectively the same price ($153.30/month vs $152.57/month).
- **The subscription's public IP quota is exhausted (0 of 3 remain).** WEST01 could not get its own public IP, so Public IP was left as None. The VM is still publicly reachable through the LB-West frontend IP, and the lab's written steps do not specify a NIC public IP.
- **The NSG carries the portal's default RDP rule alongside the lab-required HTTP rule.** The Basics inbound-ports default adds allow-RDP (TCP 3389, priority 1000); the lab's allow-HTTP rule (TCP 80, priority 1010) sits beside it.
- **Both D2s_v3 and D2s_v6 are unavailable in East US for this subscription.** EAST01 uses D2s_v7 instead — verified in the portal size picker under both Trusted Launch and Standard security. Same 2 vCPUs and 8 GiB, $0.2240/hour.
- **The OS disk type selection did not stick on the first pass for EAST01.** The first Review page showed Premium SSD even though Standard SSD had been selected. It was caught on review, re-selected to Standard SSD, and verified on the final Review page before creating. Always check the Review page, not just the form.
- **The VM size quotas are now fully used in both regions.** The Standard Dsv7 family vCPU quota in East US is exhausted (0 of 4 remain) after EAST01 and EAST02, and D2s_v6 in West US shows insufficient regional quota for new deployments. No more VMs are needed for the labs, so this blocks nothing — but no additional VM can be created in either region without a quota increase.
- **Repeated refreshes from one browser did NOT alternate between backends.** Azure's load balancer distributes by a 5-tuple hash (source IP, source port, destination IP, destination port, protocol), so one client gets pinned to one backend. This is expected Azure behavior, not a misconfiguration. To prove both backends serve, a second client (another VM) was used, and the failover test — stopping IIS on one VM and watching traffic reroute to the other — proved the load balancer works.
- **The labs assign Storage Blob Data Contributor to "Thomas Mitchell", but no such user exists in the tenant.** The portal member search returned 0 results and the tenant's member list has no Thomas Mitchell. The role was assigned to the signed-in account instead on both storage accounts, which was needed for Entra ID auth uploads since key access is disabled. If Thomas Mitchell is added to the tenant later, the same role can be assigned to him.
