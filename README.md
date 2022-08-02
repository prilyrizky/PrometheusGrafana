# Installasi dan Konfigurasi Prometheus Grafana
Installasi dan Konfigurasi Prometheus dan Grafana

Langkah - langkah install dan konfigurasi Prometheus, Grafana, dan SNMP Exporter

Install dan Konfigurasi Prometheus
Install dan Konfigurasi Grafana
Install dan Konfigurasi SNMP Exporter

**I. Install dan Konfigurasi Prometheus**

**Langkah 1:** 

Buat user dan group prometheus
```
sudo groupadd --system prometheus

sudo useradd -s /sbin/nologin --system -g prometheus prometheus
````

**Langkah 2:**

Buat direktori untuk direktori data dan konfigurasi prometheus
````
sudo mkdir /var/lib/prometheus
for i in rules rules.d files_sd; do sudo mkdir -p /etc/prometheus/${i}; done
````

**Langkah 3:** 

Download Prometheus
````
sudo apt update
sudo apt -y install wget curl vim
mkdir -p /tmp/prometheus && cd /tmp/prometheus
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
````

**Langkah 4:**

Ekstrak prometheus
````
tar xvf prometheus*.tar.gz
cd prometheus*/
sudo mv prometheus promtool /usr/local/bin/
````

**Langkah 5:**

Cek versi 
````
prometheus --version
promtool --version
````

**Langkah 6:**

Pindahkan file konfigurasi prometheus ke direktori /etc/prometheus
````
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
cd $HOME
````

**Langkah 7:** 

Konfigurasi Prometheus
````
sudo nano /etc/prometheus/prometheus.yml
````
````
#Isi File konfigurasi dasarnya
#my global config
global:
  scrape_interval:     15s #Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s #Evaluate rules every 15 seconds. The default is every 1 minute
  #scrape_timeout is set to the global default (10s).

#Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      #- alertmanager:9093

#Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  #- "first_rules.yml"
  #- "second_rules.yml"
`````

**Langkah 8:**

Buat file system service untuk prometheus
Jalankan perintah di bawah
````
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
````

**Langkah 9:**
                                                     
Ubah hak akses permisi pada direktori
````
for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done 
for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done 
sudo chown -R prometheus:prometheus /var/lib/prometheus/
````
**Langkah 10**
                                                     
Nyalakan ulang daemon dan start service
````
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
````
