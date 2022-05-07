# Cài đặt Prometheus
- https://prometheus.io/download/
```
wget https://github.com/prometheus/prometheus/releases/download/v2.33.3/prometheus-2.33.3.linux-amd64.tar.gz
tar -xvzf prometheus-2.33.3.linux-amd64.tar.gz
mv prometheus-2.33.3.linux-amd64 /usr/local/prometheus/
```
- Tạo service prometheus trong systemd
```
vim /etc/systemd/system/prometheus.service
```
- Nội dung file là:
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/prometheus/prometheus \
--config.file /usr/local/prometheus/prometheus.yml \
--storage.tsdb.path /usr/local/prometheus/ \
--web.console.templates=/usr/local/prometheus/consoles \
--web.console.libraries=/usr/local/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
- Restart và enable services.
```
systemctl daemon-reload
systemctl start prometheus
systemctl status prometheus
```

# Cài đặt Grafana
- https://grafana.com/grafana/download
```
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_8.4.1_amd64.deb
sudo dpkg -i grafana-enterprise_8.4.1_amd64.deb
service grafana-server start
sudo /sbin/chkconfig --add grafana-server
systemctl daemon-reload
systemctl start grafana-server
systemctl status grafana-server
sudo systemctl enable grafana-server.service
```
- Truy cập vào Grfana: http://IP:3000
default password: admin/admin
