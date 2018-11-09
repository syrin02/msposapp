RTI Cloud Service
------------------------

Teleflora Managed Services Linux Point of Sale Applications deployed in Amazon AWS.



Overview
------------------------

This solution provides for the Teleflora Managed Services Linux point of sale applications to run in the cloud on a single-host Docker environment with a 1:1 contanier to host ratio, in a private cloud network accessible by all branch locations of the florist. It will utilize as many of the existing, proven compliant, internal processes for delivering a point of sale system to the florist as possible. This application serves to manage those processes as well as the a few additional ones for inserting the container layer. The end result will be a simple set of instructions to build, stage, and deploy a customer's point of sale server into the cloud in a small amount of time, with no loss of data.



Requirements
------------------------

- Very low cost.

- Minimal use of support time or resources.

- Automated build process.

- PA-DSS compliant.

- Minimal impact to other processes.

- Able to expand to a share-hosted style, or host multiple customers with one cloud account.

- Fast, repeatable, installation and configuration of Prod, QA, and Dev instances.

- Allow for ease to add other Teleflora POS Linux applications.

- Teleflora Managed Services Linux Cloud Backup Service.

- Minimal maintenance added to what is already done for the point of sale instance itself.

- Reporting; track use for billing, performance, and compliance purposes.

- Self-contained: The intention is to promote the conversion of physical point of sale machines to virtual. The POS servers need to be independant of each other. (If one is affected by an outage or other, no one else is effected.) Normally, it is the intention to move toward a distributed environment when moving toward cloud-hosted environments.



Design
------------------------

The solution can be considered in 4 peices (Each having different compliance implications):

1. Build (media creation):

	An automated build process, using containers, to quickly produce OS media prepared with all the required components needed by the OS and application installation. Technically, the use of pre-prepared media from a marketplace or other 3rd party, isn't recommended for PCI compliance. Additionally, in a catastrophic situation, quickly matching patch levels from a customer's physical server becomes a requirement.

		- Menu item 11 to build OS media image.



		Enter selection: 13
		REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
		rhel7-rti-16.1.3                   latest              05b1c483ffcf        19 seconds ago      1.38 GB
		registry.access.redhat.com/rhel7   latest              eb205f07ce7d        2 weeks ago         203 MB
		Press enter to continue..



2. Staging (creation of a running, generic, instance from media):

	Prepare the linux boot volume, combine with added required pieces needed for deployment from managed services for the application installation, run through the build process, then commit to the resulting container.



		Enter selection: 12
		...
		...
		...
		No packages marked for update
		sha256:1b69b029b807e52398b7446abbc5207d294b2dd0cc36703fb81ee93024a23dfb
		---
		rhel7-rti-12345678 instance is ready!
		---
		OSTools Version: 1.15.0
		updateos.pl: $Revision: 1.347 $
		CentOS Linux release 7.5.1804 (Core) 
		---
		Press enter to continue..



		Enter selection: 1
		CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                                                                    NAMES
		9e2f3ba06379        rhel7-rti-16.1.3    "/usr/sbin/init"    2 minutes ago       Up 2 minutes        22/tcp, 80/tcp, 111/tcp, 443/tcp, 445/tcp, 631/tcp, 2001-2006/tcp, 9100/tcp, 15022/tcp   12345678.teleflora.com
		Press enter to continue..



		Enter selection: 13
		REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
		12345678.teleflora.com             latest              1b69b029b807        3 minutes ago       1.58 GB
		rhel7-rti-16.1.3                   latest              05b1c483ffcf        7 minutes ago       1.38 GB
		registry.access.redhat.com/rhel7   latest              eb205f07ce7d        2 weeks ago         203 MB
		Press enter to continue..



3. Deployment (with data):

	Assign to a customer, create VPN connection, mount persisted data, and start application instance.

		- Menu item 112 and 12.

4. Reporting: 

	Creation of reporting sufficient enough to produce historical info for billing, performance, and compliance purposes.

	Examples: yearly key rotations, periodic patch updates, instance inventory/subscriber list, or perhaps running time for a time-slice billing option.

The resulting EC2 instance will be hardened, as well as address the gaps covered by the PCI references below. It will run the linux POS application in a container that is built with the same processes as the physical servers sold to the florists now. There will be a 1-to-1 container to host ratio to allow all host resources to be used by the point of sale application, as well as simplify the segregation of customer data per PA-DSS requirements. The point of sale instance will be connected by VPN connection to the florist's network(s), and route all traffic through the florist via that VPN tunnel. This allows us to block all ports inbound to the container itself because we are using the POS application server as the VPN client, who _initiates_ the connection.

need real image here

![](https://github.com/mykol-com/MSCloudServer/blob/master/msposapp/pics/docker_single_host.png)



Installation
------------------------

1. Launch an RHEL7 EC2 instance in AWS with the following confuration options:

	- A second network interface (eth1) assigned to the VM.
	- 100GB of disk space.
	- 2 Elastic IPs. Each assigned to each NIC. (One for the Docker host, one for the container.)
		- Assign the 2nd NIC (eth1) an IP of 192.168.222.22/24.
	- Ports to be opened inbound to host (eth0): ssh (22).
	- Ports to be opened inbound to container (eth1): None (Block all inbound _initiated_ connections).

2. Download and install cloud admin menus:

		sudo yum install git
		git clone https://github.com/mykol-com/msposapp.git
		cd ./msposapp
		sudo ./MENU
		
		10:18:27 - Not Installed
		┏━━━━━━━━━━━━
		┃ RTI Cloud Menu
		┣━
		┃ 1. Running Instances
		┃ 2. Start Instance
		┃ 3. Stop Instance
		┃ 4. Connect to Instance
		┃ 5. List NICs
		┃
		┃ 11. Build OS Media
		┃ 12. Stage a Server
		┃ 13. List Images
		┃ 14. Delete Image(s)
		┃
		┃ 111. Instance Snapshot
		┃ 112. List VPNs
		┃ 113. Create VPN
		┃ 114. Delete VPN
		┃
		┃ p. Purge All
		┃ d. I/C/U Deps
		┃ a. I/C/U AWS
		┃ x. Exit
		┗━
		Enter selection: 
		
		Select "d" to Install/Configure/Upgrade Dependant packages; 1st time need Redhat support login.
		Select "a" to I/C/U AWSCLI - Need key and secret key, desired region, and enter "text" for output.

- Next, build the OS media (11), stage an instance (12), create a VPN connection (113), mount persisted data (auto), then start the point of sale server (2).



Costs
------------------------

![](https://github.com/mykol-com/msposapp/blob/master/pics/ss1.png)


![](https://github.com/mykol-com/msposapp/blob/master/pics/ss2.png)


Add larger server option



PCI/Security References
------------------------

In my opinion, the most important quote from any of these articles: 

>_"And while there are hurdles to be jumped and special attention that is needed when using containers in a cardholder data environment, there are no insurmountable obstacles to achieving PCI compliance."_ - Phil Dorzuk.

- Good Container/PCI article:

	https://thenewstack.io/containers-pose-different-operational-security-challenges-pci-compliance/

- Another, with downloadable container specific PA-DSS guide:

	https://blog.aquasec.com/why-container-security-matters-for-pci-compliant-organizations

- And, one more, that outlines the differences to address between PA-DSS Virtualization and Docker/containers:

	https://www.schellman.com/blog/docker-pci-compliance

- PCI Cloud Computing Guidelines:

	https://www.pcisecuritystandards.org/pdfs/PCI_DSS_v2_Cloud_Guidelines.pdf

- CIS Hardening benchmarks for OS, virtualization, containers, etc., with downloadable pdf:

	https://learn.cisecurity.org/benchmarks

- Free container vulnerability scanner:
	
	https://www.open-scap.org/resources/documentation/security-compliance-of-rhel7-docker-containers/

- Docker hardening standards:

	https://benchmarks.cisecurity.org/tools2/docker/CIS_Docker_1.12.0_Benchmark_v1.0.0.pdf
	
	https://web.nvd.nist.gov/view/ncp/repository/checklistDetail?id=655



Other References
------------------------

- Amazon AWS:

	https://aws.amazon.com/
	
	https://aws.amazon.com/cli/

	https://calculator.s3.amazonaws.com/

- Redhat Containers:

	https://access.redhat.com/containers/

- CentOS:

	https://www.centos.org/
	
	https://www.centos.org/forums/

- Docker:

	https://www.docker.com/
	
	https://docs.docker.com/ 

- Kickstart Info:

	https://github.com/CentOS/sig-cloud-instance-build/tree/master/docker 

- Host to Container Network Configuration:

	https://github.com/jpetazzo/pipework 

- Teleflora Managed Services OSTools:

	http://rtihardware.homelinux.com/ostools/ostools.html 

- PCI Council:
	
	https://www.pcisecuritystandards.org/

- OpenSCAP:

	https://www.open-scap.org/

- vpnc for RHEL/CentOS7 installation instructions:

	https://linuxconfig.org/establishing-cisco-vpn-client-connection-on-rhel-7-using-vpnc



------------------------
Mike Green -- mgreen@teleflora.com
