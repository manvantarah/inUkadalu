# Prometheus # 

## Step[1]: Downloading Prometheus.
a) Update the Ubuntu os:

	$ sudo apt-get update

b) create the necessary directories for storing Prometheus’ files and data:

	$ sudo mkdir /etc/prometheus
	$ sudo mkdir /var/lib/prometheus

c) Download and unpack the current stable version of Prometheus into the home directory:

	$ cd ~
    $ curl -LO https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz

d) use the sha256sum command to generate a checksum of the downloaded:

	$ sha256sum prometheus-2.0.0.linux-amd64.tar.gz

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Note: Compare the output from this command with the checksum on the Prometheus download page to ensure that your file is both genuine and not corrupted.

	Output:
	e12917b25b32980daee0e9cf879d9ec197e2893924bd1574604eb0f550034d46  prometheus-2.0.0.linux-amd64.tar.gz
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

e) unpack the downloaded archive:

	$ tar xvf prometheus-2.0.0.linux-amd64.tar.gz

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Note: This will create a directory containing two binary files (prometheus and promtool).
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

f) Copy the two binaries to the /usr/local/bin directory:

	$ sudo cp prometheus-2.0.0.linux-amd64/prometheus /usr/local/bin/
    $ sudo cp prometheus-2.0.0.linux-amd64/promtool /usr/local/bin/

g) Set the user and group ownership on the binaries to the prometheus:

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Note: manvantara:migroup(username:groupname)  

	$ sudo chown manvantara:migroup /etc/prometheus
    $ sudo chown manvantara:migroup /var/lib/prometheus
    $ sudo chown -R manvantara:migroup /etc/prometheus/consoles
    $ sudo chown -R manvantara:migroup /etc/prometheus/console_libraries

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

h) Remove the leftover files from home directory as they are no longer needed:

	$ rm -rf prometheus-2.0.0.linux-amd64.tar.gz prometheus-2.0.0.linux-amd64


			Prometheus  installed!!


## Step[2]: Configuring Prometheus.
a) Use any text editor to create a configuration file named prometheus.yml:

	$ sudo subl /etc/prometheus/prometheus.yml		

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Note: Insert the the following code in config file, hit save and exit
	
	global:
		scrape_interval: 15s
	scrape_configs:
		- job_name: 'prometheus'
    	scrape_interval: 5s
    	static_configs:
      		- targets: ['localhost:9090']

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

b) set the user and group ownership on the configuration file to the prometheus.yml:

	$ sudo chown manvantara:migroup /etc/prometheus/prometheus.yml


## Step[3]: Running Prometheus.
a) Start up Prometheus as the prometheus user, providing the path to both the configuration file and the data directory:

	    $ sudo -u prometheus /usr/local/bin/prometheus \
        --config.file /etc/prometheus/prometheus.yml \
        --storage.tsdb.path /var/lib/prometheus/ \
        --web.console.templates=/etc/prometheus/consoles \
        --web.console.libraries=/etc/prometheus/console_libraries

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------Expected Output:

	level=info ts=2017-11-17T18:37:27.474530094Z caller=main.go:215 msg="Starting Prometheus" version="(version=2.0.0, branch=HEAD, re
	vision=0a74f98628a0463dddc90528220c94de5032d1a0)"
	level=info ts=2017-11-17T18:37:27.474758404Z caller=main.go:216 build_context="(go=go1.9.2, user=root@615b82cb36b6, date=20171108-
	07:11:59)"
	level=info ts=2017-11-17T18:37:27.474883982Z caller=main.go:217 host_details="(Linux 4.4.0-98-generic #121-Ubuntu SMP Tue Oct 10 1
	4:24:03 UTC 2017 x86_64 prometheus-update (none))"
	level=info ts=2017-11-17T18:37:27.483661837Z caller=web.go:380 component=web msg="Start listening for connections" address=0.0.0.0
	:9090
	level=info ts=2017-11-17T18:37:27.489730138Z caller=main.go:314 msg="Starting TSDB"
	level=info ts=2017-11-17T18:37:27.516050288Z caller=targetmanager.go:71 component="target manager" msg="Starting target manager...
	"level=info ts=2017-11-17T18:37:27.537629169Z caller=main.go:326 msg="TSDB started"
	level=info ts=2017-11-17T18:37:27.537896721Z caller=main.go:394 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
	level=info ts=2017-11-17T18:37:27.53890004Z caller=main.go:371 msg="Server is ready to receive requests."

									^c to exit
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Special Note: If nginx is installed proceed to further steps else install nginx, which is much needed for Prometheus and Grafana

			$ sudo apt install nginx
			$ sudo nginx version

												nginx installed!!


b) open a new systemd service file:

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Note: Insert the code in the file, hit save and exit

	[Unit]
	Description=Prometheus
	Wants=network-online.target
	After=network-online.target
	[Service]
	User=manvantara
	Group=migroup
	Type=simple
	ExecStart=/usr/local/bin/prometheus \
    	--config.file /etc/prometheus/prometheus.yml \
    	--storage.tsdb.path /var/lib/prometheus/ \
    	--web.console.templates=/etc/prometheus/consoles \
    	--web.console.libraries=/etc/prometheus/console_libraries
	[Install]
	WantedBy=multi-user.target
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

c) Start systemctl: 

	$ sudo systemctl daemon-reload

d) Start prometheus:

	$ sudo systemctl start prometheus

e) Check the service’s status of prometheus:

	$ sudo systemctl status prometheus

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Output:

	● prometheus.service - Prometheus
	Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
	Active: active (running) since Thu 2020-01-09 16:19:13 IST; 2h 19min ago
	Main PID: 1132 (prometheus)
    Tasks: 9 (limit: 4675)
	CGroup: /system.slice/prometheus.service
           └─1132 /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/
	Jan 09 16:19:29 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:27.522868584Z caller=main.go:217 host_details="(Linux 5.0.0-23-generic #24~18.04.1-Ubuntu SMP Mon Jul 29 16:12:28 UTC 2019 x86_64 ka
	Jan 09 16:19:29 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:27.560621235Z caller=web.go:380 component=web msg="Start listening for connections" address=0.0.0.0:9090
	Jan 09 16:19:29 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:27.560547795Z caller=main.go:314 msg="Starting TSDB"
	Jan 09 16:19:29 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:27.579122946Z caller=targetmanager.go:71 component="target manager" msg="Starting target manager..."
	Jan 09 16:19:34 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:34.134271492Z caller=main.go:326 msg="TSDB started"
	Jan 09 16:19:34 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:34.134985401Z caller=main.go:394 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
	Jan 09 16:19:34 kadalu prometheus[1132]: level=info ts=2020-01-09T10:49:34.300276636Z caller=main.go:371 msg="Server is ready to receive requests."
	Jan 09 18:30:00 kadalu prometheus[1132]: level=info ts=2020-01-09T13:00:00.278401801Z caller=compact.go:361 component=tsdb msg="compact blocks" count=1 mint=1578564000000 maxt=1578571200000
	Jan 09 18:30:01 kadalu prometheus[1132]: level=info ts=2020-01-09T13:00:01.848261634Z caller=head.go:345 component=tsdb msg="head GC completed" duration=2.897233ms
	Jan 09 18:30:06 kadalu prometheus[1132]: level=info ts=2020-01-09T13:00:06.66579492Z caller=head.go:354 component=tsdb msg="WAL truncation completed" duration=4.81748847s
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Note: If the o/p is not active examine the previous command and configuration files. Do not proceed further !

f) enable the service to start on boot:

	$ sudo systemctl enable prometheus


		Prometheus is up and running


## Step[4]: Downloading Node Exporter.
a) Download the current stable version of Node Exporter into home directory:

	$ cd ~
    $ curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.15.1/node_exporter-0.15.1.linux-amd64.tar.gz

b) Use the sha256sum command to generate a checksum of the downloaded file:

	$ sha256sum node_exporter-0.15.1.linux-amd64.tar.gz

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Verify the below mentioned key to avoid installation of corrupted file

	7ffb3773abb71dd2b2119c5f6a7a0dbca0cff34b24b2ced9e01d9897df61a127  node_exporter-0.15.1.linux-amd64.tar.gz

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
c) Upack the downloaded archive:

	$ tar xvf node_exporter-0.15.1.linux-amd64.tar.gz

d) Copy the binary to the /usr/local/bin directory and set the user and group ownership to the node_exporter:

    $ sudo cp node_exporter-0.15.1.linux-amd64/node_exporter /usr/local/bin
    $ sudo chown manvantara:migroup /usr/local/bin/node_exporter

e) Remove the leftover files from your home directory as they are no longer needed:

    $ rm -rf node_exporter-0.15.1.linux-amd64.tar.gz node_exporter-0.15.1.linux-amd64


## Step[5]: Running Node Exporter.
a) Start by creating the Systemd service file for Node Exporter:

	$ sudo subl /etc/systemd/system/node_exporter.service

### Note: Insert the code in node_exporter.service
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	
	[Unit]
	Description=Node Exporter
	Wants=network-online.target
	After=network-online.target
	[Service]
	User=manvantara
	Group=migroup
	Type=simple
	ExecStart=/usr/local/bin/node_exporter
	[Install]
	WantedBy=multi-user.target

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

b) Reload systemd to use the newly created service:
	
	$ sudo systemctl daemon-reload

c) Run Node Exporter:

	$ sudo systemctl start node_exporter

d) Verify that Node Exporter’s running correctly with the status command:

	$ sudo systemctl status node_exporter

### Note: If the o/p is not active examine the previous command and configuration files. Do not proceed further ! 
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Output:

	● node_exporter.service - Node Exporter
	Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
	Active: active (running) since Thu 2020-01-09 19:09:05 IST; 12s ago
	Main PID: 9137 (node_exporter)
    Tasks: 6 (limit: 4675)
	CGroup: /system.slice/node_exporter.service
           └─9137 /usr/local/bin/node_exporter
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - netstat" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - zfs" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - uname" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - textfile" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - xfs" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - hwmon" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg=" - stat" source="node_exporter.go:52"
	Jan 09 19:09:08 kadalu node_exporter[9137]: time="2020-01-09T19:09:07+05:30" level=info msg="Listening on :9100" source="node_exporter.go:76"
	Jan 09 19:09:10 kadalu node_exporter[9137]: time="2020-01-09T19:09:10+05:30" level=error msg="ERROR: diskstats collector failed after 0.001920s: invalid line for /proc/diskstats for sda" source="collector
	Jan 09 19:09:15 kadalu node_exporter[9137]: time="2020-01-09T19:09:15+05:30" level=error msg="ERROR: diskstats collector failed after 0.000218s: invalid line for /proc/diskstats for sda" source="collector
	
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

e) Enable Node Exporter to start on boot:

	$ sudo systemctl enable node_exporter


## Step[6]: Configuring Prometheus to Scrape Node Exporter.
a) Open the configuration file:

	$ sudo subl /etc/prometheus/prometheus.yml

### Note: Insert the code in prometheus.yml

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	global:
		scrape_interval: 15s
	scrape_configs:
		- job_name: 'prometheus'
    	scrape_interval: 5s
    	static_configs:
      	- targets: ['localhost:9090']
      	- job_name: 'node_exporter'
    	scrape_interval: 5s
    	static_configs:
      		- targets: ['localhost:9100']    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

b) Restart Prometheus: 

	$ sudo systemctl restart prometheus

c) Verify that everything is running correctly with the status command:

	$ sudo systemctl status prometheus

### Note: If the o/p is not active examine the previous command and configuration files. Do not proceed further ! 
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Output:
	
	● prometheus.service - Prometheus
		Loaded: loaded (/etc/systemd/system/prometheus.service; disabled; vendor preset: enabled)
		Active: active (running) since Fri 2017-07-21 11:46:39 UTC; 6s ago
		Main PID: 2219 (prometheus)
    	Tasks: 6
		Memory: 19.9M
      	CPU: 433ms
      	CGroup: /system.slice/prometheus.service
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Step[7]: Securing Prometheus.

a) Install apache2-utils for access to the htpasswd utility for generating password files:

	$ sudo apt-get update
	$ sudo apt install apache2-utils

b) Create a password file:

	$ sudo htpasswd -c /etc/nginx/.htpasswd manvantara

### Note: "manvantara" is a username and typee password after executing this command

c) Make a Prometheus-specific copy of the default Nginx configuration:

	$ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/prometheus

d) Configure the new file

	$ sudo subl /etc/nginx/sites-available/prometheus

### Note: In location / block under the server block insert this code hit save and exit
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

	location / {
        auth_basic "Prometheus server authentication";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://localhost:9090;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

e) Deactivate the default Nginx configuration file by removing the link:

	$ sudo rm /etc/nginx/sites-enabled/default
	$ sudo ln -s /etc/nginx/sites-available/prometheus /etc/nginx/sites-enabled/

f) Check the configuration for errors:

	$ sudo nginx -t

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Output of Nginx configuration tests

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

g) Reload Nginx to incorporate all of the changes:

	$ sudo systemctl reload nginx

h) Verify that Nginx is up and running:

	$ sudo systemctl status nginx

### Note: If the o/p is not active examine the previous command and configuration files. Do not proceed further ! 
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Expected Output

	● nginx.service - A high performance web server and a reverse proxy server
	Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: en
	Active: active (running) since Mon 2017-07-31 21:20:57 UTC; 12min ago
	Process: 4302 ExecReload=/usr/sbin/nginx -g daemon on; master_process on; -s r
	Main PID: 3053 (nginx)
    Tasks: 2
	Memory: 3.6M
    CPU: 56ms
    CGroup: /system.slice/nginx.service
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


	Prometheus server is fully-functional and secured


## Step[9]: Testing Prometheus

### Go to local client browser and in URL type localhost:9090

In the HTTP authentication dialogue box, enter the username and password created in Step[7]


						Good to execute and visualize custom queries



# Grafana #

## Step[1]: Installing Grafana

a) Download the Grafana GPG key with wget:

	$ sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

b) Add the Grafana repository to your APT sources:

	$ sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

c) Refresh your APT cache to update your package lists:
	
	$ sudo apt update
	$ apt-cache policy grafana

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

	Output of apt-cache policy grafana
	grafana:
	Installed: (none)
	Candidate: 6.3.3
	Version table:
     6.3.3 500
        500 https://packages.grafana.com/oss/deb stable/main amd64 Packages
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

d) Install Grafana:

	$ sudo apt install grafana

e) Use systemctl to start the Grafana server:

	$ sudo systemctl start grafana-server

f) Verify that Grafana is running by checking the service’s status:

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Expected Output:

	● grafana-server.service - Grafana instance
	Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
	Active: active (running) since Tue 2019-08-13 08:22:30 UTC; 11s ago
    Docs: http://docs.grafana.org
	Main PID: 13630 (grafana-server)
    Tasks: 7 (limit: 1152)
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

g) Enable the service to automatically start Grafana on boot:

	$ sudo systemctl enable grafana-server

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Output of systemctl enable grafana-server
	
	Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
	Executing: /lib/systemd/systemd-sysv-install enable grafana-server
	Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /usr/lib/systemd/system/grafana-server.service.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Step[2]: Setting Up the Reverse Proxy:

a) Open the Nginx configuration file created at the time of nginx configuration

	$ sudo nano /etc/nginx/sites-available/prometheus

### Note: Insert the code in the configuaration file, hit save and exit

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

	location /grafana {
        proxy_pass http://localhost:3000;
    }

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

b) Test the new settings to make sure everything is configured correctly:

	$ sudo nginx -t

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Expected Output:

	nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
	nginx: configuration file /etc/nginx/nginx.conf test is successful

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## Strp[3]: Updating Credentials

### Go to local client browser and in URL type localhost:3000/grafana

### Type username and password as admin, admin and update the new username and password

	Grafana is ready to use!

