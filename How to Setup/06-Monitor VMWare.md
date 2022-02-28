- Ubuntu 20.04 đã tích hợp sẵn Python 3.8 nên chỉ cần cài python3-pip để cài vmware_exporter. Đối với các Distro khác thì phải cài Python >=3.6
```
apt install python3-pip
```
```
wget https://www.python.org/ftp/python/3.6.4/Python-3.6.4.tar.xz
tar -xJf Python-3.6.4.tar.xz
cd Python-3.6.4
./configure
make
make install
pip3 install --upgrade pip
```
- Cài vmware_exporter
```
pip3 install vmware_exporter
```
- Đường dẫn lưu trữ thư viện python: /usr/local/lib/python3.6/site-packages/vmware_exporter  . Trong trường hợp không có trong đường dẫn thì có thể sử dụng lệnh sau để tìm:
```
find / -name "vmware_exporter"
```
- Tạo user quyền read-only để monitor vCenter hoặc ESXi host
- Tạo file cấu hình cho vmware_exporter
```
cd /usr/local/lib/python3.6/site-packages/vmware_exporter
vi config.yml
```
```
default:
    vsphere_host: '192.168.1.2'
    vsphere_user: 'username'
    vsphere_password: 'password'
    ignore_ssl: True
    specs_size: 5000
    fetch_custom_attributes: True
    fetch_tags: True
    fetch_alarms: True
    collect_only:
        vms: True
        vmguests: True
        datastores: True
        hosts: True
        snapshots: True
```
- Tạo service trong systemd cho vmware_exporter
```
vi /etc/systemd/system/vmware_exporter.service
```
```
[Unit]
Description=Prometheus VMWare Exporter
After=network.target
[Service]
User=root
Group=root
ExecStart=/usr/bin/python3 /usr/local/bin/vmware_exporter -c /usr/local/lib/python3.6/site-packages/vmware_exporter/config.yml
Type=simple
[Install]
WantedBy=multi-user.target
```
- Enable và start service
```
systemctl enable vmware_exporter.service
systemctl start vmware_exporter.service
```
- Tạo job trong prometheus để giám sát Vmware
```
  - job_name: 'vCenter'
    metrics_path: '/metrics'
    scrape_timeout: 15s
    scrape_interval: 15s
    static_configs:
      - targets:
        - '192.168.1.1'
        labels:
         hostname: VCENTER
         device: VMWARE
         company: abc
    params:
      section: [default]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.x.x:9272
```
- Sau đó restart lại service prometheus.
- Tham khảo thêm tại: https://github.com/pryorda/vmware_exporter
