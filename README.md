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

**Lab 6 — Connect Virtual Networks with vNet Peering** — next
- Connect the West US and East US virtual networks so they can talk to each other.

**Lab 7 — Perform an Automated Azure Bastion Deployment** — next
- Deploy Azure Bastion automatically for secure RDP/SSH access to the VMs without public IPs.

### Module 3 — Deploy and Configure Load Balancers

**Lab 8 — Deploy and Configure a Public Load Balancer in West US** — next
- Deploy and configure a public load balancer to spread traffic across the west-region VMs.

**Lab 9 — Deploy and Configure a Public Load Balancer in East US** — next
- Deploy and configure a public load balancer to spread traffic across the east-region VMs.

### Module 4 — Deploy Resilient Azure VMs in West US

**Lab 10 — Deploy and Configure the WEST01 Virtual Machine** — next
- Deploy and configure the first virtual machine in West US.

**Lab 11 — Install IIS on WEST01 Remotely via PowerShell** — next
- Install the IIS web server on WEST01 remotely using PowerShell.

**Lab 12 — Deploy and Configure the WEST02 Virtual Machine** — next
- Deploy and configure the second virtual machine in West US.

**Lab 13 — Install IIS on WEST02 Remotely via PowerShell** — next
- Install the IIS web server on WEST02 remotely using PowerShell.

### Module 5 — Deploy Resilient Azure VMs in East US

**Lab 14 — Deploy and Configure the EAST01 Virtual Machine** — next
- Deploy and configure the first virtual machine in East US.

**Lab 15 — Install IIS on EAST01 Remotely via PowerShell** — next
- Install the IIS web server on EAST01 remotely using PowerShell.

**Lab 16 — Deploy and Configure the EAST02 Virtual Machine** — next
- Deploy and configure the second virtual machine in East US.

**Lab 17 — Install IIS on EAST02 Remotely via PowerShell** — next
- Install the IIS web server on EAST02 remotely using PowerShell.

### Module 6 — Test Load Balancers

**Lab 18 — Ensure Virtual Machine IP Addresses Are Static** — next
- Change the VMs' IP addresses from dynamic to static so load balancer rules stay stable.

**Lab 19 — Test Load Balancing Functionality in West US** — next
- Verify the west-region load balancer spreads traffic across WEST01 and WEST02.

**Lab 20 — Test Load Balancing Functionality in East US** — next
- Verify the east-region load balancer spreads traffic across EAST01 and EAST02.

### Module 7 — Deploy and Configure Azure Blob Storage

**Lab 21 — Deploy Blob Storage in the West US Region** — next
- Deploy blob storage in West US for the app's files.

**Lab 22 — Deploy Blob Storage in the East US Region** — next
- Deploy blob storage in East US for the app's files.

**Lab 23 — Configure Blob Storage Object Replication Between Regions** — next
- Configure object replication so blobs in one region copy to the other.

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

## What Went Wrong

- **Portal defaulted new subnets to "Private subnet: Enabled."** The labs do not mention this setting. It was left as the default — it creates no extra resource and adds no cost, and the NAT gateways provide the outbound path.
- **Portal defaulted regions to West US 2 / East US 2.** The lab specifies West US and East US, so the exact regions were selected instead of accepting the defaults.
- **vNet-East defaulted to the same address space as vNet-West (10.0.0.0/16).** It was changed to the lab's 10.1.0.0/16 to avoid overlap between the two VNets.
- **The simulation subscription is "Surf City Boats."** The real subscription is "Azure subscription 1" — the real one was used for every resource.
