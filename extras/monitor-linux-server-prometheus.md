# Basic steps to congfigure Prometheus server

## Special note:

1. Prometheus Node Exporter needs Prometheus server to be up and running.

2. Port 9100 opened in server firewall as Prometheus reads metrics on this port.

## Start system daemon and node exporter service

	$ sudo systemctl daemon-reload
	$ sudo systemctl start node_exporter

## Check the node exporter status

	$ sudo systemctl status node_exporter

### Expected Output:

	● node_exporter.service - Node Exporter
	Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor p
	Active: active (running) since Mon 2020-01-27 18:32:12 IST; 4s ago
	Main PID: 17543 (node_exporter)
    Tasks: 4 (limit: 4666)
	CGroup: /system.slice/node_exporter.service
       	└─17543 /usr/local/bin/node_exporter

## Enable the node exporter service to the system startup

	$ sudo systemctl enable node_exporter

## configure the Server as target on prometheus server

	$ sudo subl /etc/prometheus/prometheus.yaml

### Edit the yaml file

	- job_name: 'node_exporter_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['pod-ip:9100']  # pod-ip as shown in `$ sudo kubectl get pods -o wide`

## Restart the prometheus service

	$ sudo systemctl restart prometheus


### Go to the client browser and type in url

	http://<prometheus-IP>:9090/targets

### Note: The end point state should be up 