- Cài đặt node_exporter: https://github.com/prometheus/node_exporter/releases/tag/v1.3.1
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar -xvzf node_exporter-1.3.1.linux-amd64.tar.gz
mv node_exporter-1.0.0-rc.1.linux-amd64/node_exporter /usr/local/node_exporter
```
- Tạo service trong systemd cho node_exporter
```
vim /etc/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=root
ExecStart=/usr/local/node_exporter/node_exporter
[Install]
WantedBy=default.target
```
- Truy cập vào node để kiểm tra: http://ip:9100
- Máy có iptables thì sử dụng lệnh sau để mở port
```
iptables -I INPUT -p tcp -s xxx.xxx.xxx.xxx -m tcp --dport 9100 -j ACCEPT
```
- Tạo job trong prometheus để giám sát server Linux này với nội dung job sau:
```
###### Linux
  - job_name: 'Linux'
    static_configs:
    - targets: ['10.10.10.54:9100']
      labels:
       hostname: Jitsi
       type: linux
       company: AIS
```
```
systemctl restart prometheus
```
