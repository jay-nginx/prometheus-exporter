# prometheus-exporter
This repository takes you through the details on how to extract metrics from your NGINX Plus instance using prometheus exporter and view it in Grafana. 

<img src=https://user-images.githubusercontent.com/52437445/126335810-75b706d7-5856-40b3-9f3b-c0371858f842.png alt="Prometheus Exporter Architecture" width=850>

## Required Components

1. NGINX Plus Instance
2. NGINX Prometheus Exporter - Will require Docker to run
3. Prometheus - Install steps included as below
4. Grafana - Install steps


Steps below include details on how you configure all the required components to view NGINX Plus metrics in Grafana. 

### NGINX Plus

- You will need to have your NGINX Plus installed and running.
- Ensure that you have teh dashboard.conf applied and working


### Prometheus Exporter

- Ensure you have docker installed
   ` docker run -p 9113:9113 -d nginx/nginx-prometheus-exporter:0.9.0 -nginx.plus -nginx.scrape-uri=http://10.1.1.4:8080/api `
    
    
### Prometheus

Follow the steps below for Prometheus Installation

```
cd /opt/
sudo wget https://github.com/prometheus/prometheus/releases/download/v2.27.1/prometheus-2.27.1.linux-amd64.tar.gz
ls
tar xvf prometheus-2.27.1.linux-amd64.tar.gz
sudo tar xvf prometheus-2.27.1.linux-amd64.tar.gz
cd prometheus-2.27.1.linux-amd64
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
prometheus --version
promtool --version
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/  /var/lib/prometheus/
sudo chmod -R 775 /etc/prometheus/ /var/lib/prometheus/
sudo nano /etc/systemd/system/prometheus.service

		[Unit]
		Description=Prometheus
		Wants=network-online.target
		After=network-online.target

		[Service]
		User=prometheus
		Group=prometheus
		Type=simple
		ExecStart=/usr/local/bin/prometheus \
		    --config.file /etc/prometheus/prometheus.yml \
		    --storage.tsdb.path /var/lib/prometheus/ \
		    --web.console.templates=/etc/prometheus/consoles \
		    --web.console.libraries=/etc/prometheus/console_libraries

		[Install]
		WantedBy=multi-user.target


sudo cat /etc/systemd/system/prometheus.service
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
sudo docker ps
```

### Grafana

Follow the following steps to install Grafana

```
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana-enterprise
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service
```

## Let's check the configurations now

Ensure your NGINX Plus instane is working.

`sudo nginx -v`

Expected outcome should be similar to: `nginx version: nginx/1.19.10 (nginx-plus-r24-p1)`

`curl localhost:8080/api/6`

Expected outcome should be similar to: `["nginx","processes","connections","slabs","http","resolvers","ssl"]`

Run your NGINX Prometheus Exporter:
`docker run -p 9113:9113 -d nginx/nginx-prometheus-exporter:0.9.0 -nginx.plus -nginx.scrape-uri=http://<ip_local-machine>:8080/api`

Ensure Prometheus Exporter is scraping the metrics:
`curl localhost:9113/metrics`

Expected outcome should be similar to: 
	`
	# HELP nginxexporter_build_info Exporter build information
	# TYPE nginxexporter_build_info gauge
	nginxexporter_build_info{commit="5f88afbd906baae02edfbab4f5715e06d88538a0",date="2021-03-22T20:16:09Z",version="0.9.0"} 1
	# HELP nginxplus_connections_accepted Accepted client connections
	`

Access your Prometheus Instance:
via Browser - http://localhost:9090

Edit the prometheus.yaml file to ensure that you have target as - localhost:9113
Location of prometheus.yaml file - /etc/prometheus/prometheus.yaml
In the last line of the prometheus.yaml - ensure that you have  " - targets: ['localhost:9113'] "

`sudo systemctl restart prometheus`

via Browser, ensure that you are now able to view the " `nginxplus_*` " metrics in Prometheus.


Access your Grafana Instance:
via Browser - http://localhost:3000




