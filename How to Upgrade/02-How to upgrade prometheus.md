- check current version
```
# prometheus — version
prometheus, version 2.10.0 (branch: HEAD, revision: d20e84d0fb64aff2f62a977adc8cfb656da4e286)
build user: root@a49185acd9b0
build date: 20190525–12:28:13
go version: go1.12.5
```
- dừng tất cả service đang chạy
```
systemctl stop prometheus.service
systemctl stop alertmanager.service
systemctl stop blackbox.service
systemctl stop grafana-server.service
```
- Download phiên bản mới:
```
wget https://github.com/prometheus/prometheus/releases/download/v2.17.2/prometheus-2.17.2.linux-amd64.tar.gz
wget -O — -q https://github.com/prometheus/prometheus/releases/download/v2.17.2/sha256sums.txt | grep linux-amd64 | shasum -c -
```
```
tar -xvf prometheus-2.17.2.linux-amd64.tar.gz
cd prometheus-2.17.2.linux-amd64/
```
- Copy the 'prometheus' and 'promtool' vào thư mục /usr/local/bin 
```
cp prometheus-2.17.2.linux-amd64/{prometheus,promtool} /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
```
```
cp -r prometheus-2.17.2.linux-amd64/{consoles,console_libraries} /etc/prometheus/
chown -R prometheus:prometheus /etc/prometheus
```
- Verify the content of Prometheus data directory
```
ls -l /var/lib/prometheus
```
- reset all service:
```
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
systemctl start alertmanager.service
systemctl start blackbox.service
systemctl start grafana-server.service
```
- Verify the Prometheus Version
```
# prometheus — version
prometheus, version 2.17.2 (branch: HEAD, revision: 18254838fbe25dcc732c950ae05f78ed4db1292c)
build user: root@9cb154c268a2
build date: 20200420–08:27:08
go version: go1.13.10
```
---------------
```
#systemctl stop prometheus.service
#systemctl stop alertmanager.service
#systemctl stop blackbox.service
#systemctl stop grafana-server.service
#wget https://github.com/prometheus/prometheus/releases/download/v2.19.2/prometheus-2.19.2.linux-amd64.tar.gz
#tar -xvf prometheus-2.19.2.linux-amd64.tar.gz
#cp prometheus-2.19.2.linux-amd64/{prometheus promtool} /usr/local/bin/
#cp -r prometheus-2.19.2.linux-amd64/{consoles,console_libraries} /etc/prometheus/
#chown -R prometheus:prometheus /etc/prometheus
#ls -l /var/lib/prometheus
#systemctl daemon-reload
#systemctl start prometheus
#systemctl enable prometheus
#systemctl start alertmanager.service
#systemctl start blackbox.service
#systemctl start grafana-server.service
#systemctl status prometheus.service
# prometheus — version
prometheus, version 2.19.2 (branch: HEAD, revision: c448ada63d83002e9c1d2c9f84e09f55a61f0ff7)
build user: root@dd72efe1549d
build date: 20200626–09:02:20
go version: go1.14.4
```


