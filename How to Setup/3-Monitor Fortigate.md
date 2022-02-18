- Cài đặt golang: https://go.dev/dl/
```
wget https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
tar -xvzf go1.12.5.linux-amd64.tar.gz
mv go /usr/local
```
- Export GOROOT và GOPATH
```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
- Kiểm tra version GoLang và verify lại cấu hình
```
go version
go env
```
- Mặc định export không thì GoLang chỉ tồn tại trong 1 shell. Nếu muốn all shell thì phải thêm dòng lệnh export vào .bashrc
```
vi .bashrc
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
- Sử dụng SNMP Exporter Config Generator biên dịch và download mibs.: https://github.com/prometheus/snmp_exporter
```
# Debian-based distributions.
sudo apt-get install unzip build-essential libsnmp-dev p7zip-full # Debian-based distros
# Redhat-based distributions.
sudo yum install gcc gcc-g++ make net-snmp net-snmp-utils net-snmp-libs net-snmp-devel # RHEL-based distros

go get github.com/prometheus/snmp_exporter/generator
cd ${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator
go build
make mibs
```
- Thực hiện biên dịch và download mibs từ tool generator, đối với những loại mibs ko có sẵn trong này, bạn phải download từ hãng về và bỏ thư mục chứa các mibs.
```
cd ${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator
go build         
make mibs
```
- Mibs của những thiết bị sẽ được download và copy tới đường dẫn sau
```
${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator
${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator/mibs
```
- Tải mib của thiết bị bằng cách sau: Login vào Fortigate, vào System, vào SNMP và chọn download mibs về.
- Thiết lập snmp trên thiết bị Fortigate
- Edit file generator.yml và thêm module foritgate. Mặc định file generator.yml có rất nhiều thông tin trong này, chúng ta khuyến khị chỉ sử dụng những thông tin liên quan đến thiết bị cần monitor.
```
vi go/src/github.com/prometheus/snmp_exporter/generator/generator.yml
```
```
modules:
###### Fortigate
  fortigate_snmp:
   walk:
   - ifXTable
   - fgVpn
   - fgSystem
   - fgIntf
   version: 2
   max_repetitions: 25
   retries: 3
   timeout: 10s
   auth:
    community: public
```
 Trong đó chúng ta cần chú ý các tham số sau:

fortigate_snmp là tên module.

ifXTable, fgVpn, fgSystem, fgIntf là các tham số OID.

version:2  là snmpv2.

community: communityname là community name khi enable trong Fortigate.
- Tiếp tục sau đó export mibs và generate ra file snmp.yml
```
export MIBDIRS=mibs
./generator generate
```
- Cài đặt smnp_exporter lên prometheus server. Link download: https://github.com/prometheus/snmp_exporter/releases
```
wget https://github.com/prometheus/snmp_exporter/releases/download/v0.15.0/snmp_exporter-0.15.0.linux-amd64.tar.gz
```
- Giải nén và copy source đến thư mục /usr/local/snmp_exporter
```
tar -xvzf snmp_exporter-0.15.0.linux-amd64.tar.gz
mv snmp_exporter* /usr/local/snmp_exporter
```
- Tạo service trong systemd cho snmp_exporter
```
vi /etc/systemd/system/snmp_exporter.service
```
```
[Unit]
Description=Snmp_exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/snmp_exporter/snmp_exporter \
--config.file=/usr/local/snmp_exporter/snmp.yml

[Install]
WantedBy=multi-user.target
```
- Copy file snmp.yml đến thư mục source của snmp_exporter
```
cp -R /root/go/src/github.com/prometheus/snmp_exporter/generator/snmp.yml /usr/local/snmp_exporter/
```
- Restart và enable service
```
systemctl restart snmp_exporter.service
systemctl status snmp_exporter.service
systemctl enable snmp_exporter.service
```
- Kiểm tra metric của snmp_exporter: http://ipprometheus:9116
```
Target: là ip của fortigate
Module: chính là module mà bạn tạo bằng generator cho fortigate
```
- Tạo job trong prometheus để giám sát Fortigate với nội dung job sau:
```
vi /usr/local/prometheus/prometheus.yml
```
```
###### FORTIGATE
  - job_name: 'fortigate'
    static_configs:
      - targets:
        - 172.16.100.1 # fortigate device.
        labels:                           
         hostname: GROUP-FG
         device: fortigate
         company: abc
    scrape_interval: 3m
    scrape_timeout : 3m
    metrics_path: /snmp
    params:
      module: [fortigate_snmp]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.10.10.98:9116  # SNMP exporter.
```
```
systemctl restart prometheus
```







