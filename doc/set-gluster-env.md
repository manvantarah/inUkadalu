# Setting up Gluster environment on Ubuntu 18.04 virtual machine

## Update and Upgrade the system packages

	$ sudo apt-get update
	$ sudo apt upgrade

## Create a brick directory for GlusterFS volumes on the GlusterFS storage device mount point 

	$ mkdir -p /gfs/gvs

## Install latest GlusterFS on Ubuntu 18.04

	$ sudo apt-get install software-properties-common
	$ sudo add-apt-repository ppa:gluster/glusterfs-7

### Note: Once PPA repository is added, update the system

	$ sudo apt update

## Install GlusterFS and enable it to run on system boot

	$ sudo apt install glusterfs-server
	$ sudo systemctl enable glusterd
	$ sudo systemctl start glusterd

### Note: To check the status of glusterFS

	$ sudo systemctl is-active glusterd

## Add all the node available in 'sudo kubectl get nodes -o wide' to /etc/hosts

	$ sudo subl /etc/hosts

### Note: Add rule to firewall or trun-off the firewall to communicate with other node

	$ sudo ufw allow from <other-node-IP>

### Note: Once the node added verify with ping coomand with no packet loss

	$ ping -c 3 <node-ip or name>

## Configure GlusterFS trusted pool

	$ sudo gluster peer probe <node-name>

### Expected output:

	peer probe: success.

## Check the status of the trusted pool just created

	$ sudo gluster peer status

### Expected output

	Number of Peers: 1
	Hostname: minikube
	Uuid: 5a2dd392-9e3b-4710-8803-e6055694a955
	State: Peer in Cluster (Connected)

## List the storage pools created

	$ sudo gluster pool list

### Expected O/P:
	
	UUID									 Hostname 	 State
	0c5104f6-1253-41b6-98a8-c723eab30401	localhost	Connected 
	5a2dd392-9e3b-4710-8803-e6055694a955	minikube	Connected 

## Create Directory for mounting

	$ sudo su
	$ mkdir /gfs/gvs

### Note: Make a new directory for every new volume created

## Create Distributed GlusterFS volume

	$ sudo gluster volume create <volume-name> transport tcp <node-name>:/gfs/gvs force 

### Expected O/P:

	volume create: <volume-name>: success: please start the volume to access data

## Start the created volume

	$ sudo gluster volume start <volume-name>

### Expected Result:

	volume start: <volume-name>: success

### See the information of the created volume

	$ sudo gluster volume info

#### Expected O/P:

	Volume Name: gps0
	Type: Distribute
	Volume ID: fd205504-4eb6-4db1-973b-4663a80cb922
	Status: Started
	Snapshot Count: 0
	Number of Bricks: 1
	Transport-type: tcp
	Bricks:
	Brick1: manvantara:/gfs/gvs
	Options Reconfigured:
	transport.address-family: inet
	storage.fips-mode-rchecksum: on
	nfs.disable: on
	diagnostics.latency-measurement: on
	diagnostics.count-fop-hits: on


	Gluster environment setup is ready to work!


