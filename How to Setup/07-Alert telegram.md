- Đầu tiên chúng ta sẽ download source cài đặt vào trong server: https://github.com/prometheus/alertmanager
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz
```
- Giải nén và copy vào thư mục /usr/local để dễ dàng quản lý
```
tar -xvzf alertmanager-0.20.0.linux-amd64.tar.gz
mv alertmanager-0.20.0.linux-amd64 /usr/local/alertmanager
```
- Tạo service trong systemd cho alertmanager
```
vi /etc/systemd/system/alertmanager.service
```
```
[Unit]
Description=AlertManager
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/alertmanager/alertmanager \
--config.file=/usr/local/alertmanager/alertmanager.yml
[Install]
WantedBy=multi-user.target
```
```
systemctl daemon-reload
systemctl enable alertmanager
systemctl restart alertmanager
```
- Cấu hình alert trong prometheus
```
vi /usr/local/prometheus/prometheus.yml
```
- Tại targets: nhập thông tin ip, port , tại đây chính là ip của prometheus server và port 9093 (alertmanager dùng port này).
```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

#### Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - 192.168.x.x:9093
```
- Tạo các rule alert trong prometheus. Để tạo alert trong prometheus, bạn cần phải định nghĩa ra các rule. Tạo các file rule trong thư mục promethues /usr/local/promethéu với format sau: rulename.yml
- Ví dụ tạo Windows-rules.yml
```
vi /usr/local/prometheus/windows-rules.yml
```
```
groups:
- name: Windows-alert
  rules:

################ Memory Usage High
  - alert: Memory Usage High
    expr: 100-100 * windows_os_physical_memory_free_bytes/ windows_cs_physical_memory_bytes > 85
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "Memory Usage (instance {{ $labels.instance }})"
      description: "Memory Usage is more than 85%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ CPU Usage High
  - alert: Cpu Usage High
    expr: 100 - (avg by (instance) (rate(windows_cpu_time_total{mode="idle"}[2m])) * 100)  > 85
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "CPU Usage (instance {{ $labels.instance }})"
      description: "CPU Usage is more than 85%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ Disk Usage
  - alert: DiskSpaceUsage
    expr: 100.0 - 100 * ((windows_logical_disk_free_bytes{} / 1024 / 1024 ) / (windows_logical_disk_size_bytes{}  / 1024 / 1024)) > 85
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Disk Space Usage (instance {{ $labels.instance }})"
      description: "Disk Space on Drive is used more than 85%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ ServiceStatus
  - alert: ServiceStatus
    expr: windows_service_status{status="ok"} != 1
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Service Status (instance {{ $labels.instance }})"
      description: "Windows Service state is not OK\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

################ CollectorError
  - alert: CollectorError
    expr: windows_exporter_collector_success == 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Collector Error (instance {{ $labels.instance }})"
      description: "Collector {{ $labels.collector }} was not successful\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```
```
vi /usr/local/prometheus/prometheus.yml
```
```
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
  - "windows-rules.yml"
  - "linux-rules.yml"
  - "prometheus-rules.yml"
  - "vmware-rules.yml"
```
- Kiểm tra lại rule đã tạo bằng lệnh sau:
```
cd /usr/local/bin/
./promtool check config  /usr/loca/prometheus/prometheus.yml
```
- restart lại service prometheus và kiểm tra kết quả
- Tải bot telegram: https://github.com/inCaller/prometheus_bot
```
cd ${GOPATH-$HOME/go}/src/github.com
go get github.com/inCaller/prometheus_bot
cd ${GOPATH-$HOME/go}/src/github.com/inCaller/prometheus_bot
make clean
make
```
- Copy bỏ vào thư mục /usr/local để dễ dàng quản lý.
```
mv /root/go/src/github.com/inCaller/prometheus_bot /usr/local
```
- Cách tạo telegram bot tham khảo tại đây: https://www.teleme.io/articles/create_your_own_telegram_bot?hl=vi
- Khi tạo telegram bot mình sẽ ghi lại token của con bot, dùng để điền vào file config.yaml mà chúng ta tạo sau đó.
- Cách lấy chatid có thể sử dụng: Type “@RawDataBot” and select “Telegram Bot Raw”
- Tạo file cấu hình của prometheus_bot
```
vi /usr/local/prometheus_bot/config.yaml
```
```
telegram_token: "997872129:AAEPKYz3nPwmFsgq6ao-MdPsC5fy5z376GQ"

# ONLY IF YOU USING TEMPLATE required for test
template_path: "template.tmpl"
time_zone: "Asia/Ho_Chi_Minh"
split_token: "|"

# ONLY IF YOU USING DATA FORMATTING FUNCTION, NOTE for developer: important or test fail
time_outdata: "02/01/2006 15:04:05"
split_msg_byte: 4000
```
- Tiếp tục cấu hình alert manager:
```
vi /usr/local/alertmanager/alertmanager.yml
```
```
global:
  resolve_timeout: 1m
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 30m
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://172.16.253.188:9087/alert/-679249431'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
~
```
- Thực hiện test gửi nhận tin nhắn trên telegram với telegram_bot với cú pháp sau:
```
cd /usr/local/prometheus_bot/
export TELEGRAM_CHATID="-364942581"
make test
```
- Tiếp tục tạo templet gửi alert trong prometheus. Trong thư mục testdata có nhiều templet cho bạn sử dụng, thường mình dùng templet production_example.tmpl
```
cd /usr/local/prometheus_bot/
cp production_example.tmpl /usr/local/prometheus_bot/template.tmpl
```
- Tiếp tục edit lại tên của template trong config.yaml của prometheus_bot
- Reset lại các service đã cấu hình
```
service prometheus_bot restart
service alertmanager restart
service prometheus restart
```
https://prometheus.io/docs/alerting/alertmanager/
https://awesome-prometheus-alerts.grep.to/

















