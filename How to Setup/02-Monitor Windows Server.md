- Download windows_exporter:
```
https://github.com/prometheus-community/windows_exporter/releases

wmi_exporter.exe (click to run, dành cho các bạn nào chỉ cần các metric được enable sẵn).
wmi_exporter.msi (dùng để cài đặt thông qua CMD, enable các tính năng thu thập metric nâng cao).
```
- Khi cài agent qua CMD sẽ giúp các bạn chọn lọc thu thập những loại metric nào cần thu thập, giảm các loại metric không cần thiết, có thể thu thập nhiều loại metric hơn mặc định. Mở port 9182 trên server linux, và windows server.
- Mở cmd bằng administrator, cài file msi bằng lệnh sau:
```
msiexec /i C:\wmi_exporter-0.9.0-amd64.msi ENABLED_COLLECTORS="ad,cpu,cs,logon,memory,logical_disk,os,service,system,process,tcp,net,textfile,thermalzone"
```
- Cài thành công thì trong Service sẽ có Service là Wmi exporter
- Kiểm tra metric bạn truy cập như sau: http://ipserver:9182
- Tạo job trong prometheus để giám sát Windows Server này với nội dung sau:
```
vi /usr/local/prometheus/prometheus.yml
```
```
  - job_name: 'windows'
    static_configs:
    - targets: ['10.10.10.8:9182']
      labels:                           
       hostname: DC01
       type: windows
       company: XYZDG
 ```
- Restart serivce prometheus
```
systemctl restart prometheus
systemctl status prometheus
```
