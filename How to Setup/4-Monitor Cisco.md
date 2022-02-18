- Cấu hình snmpv3 trên thiết bị Cisco
- Add module cisco vào file generator.yml để tạo ra file config snmp.yml có chứa module cisco.
```
cd ${GOPATH-$HOME/go}/src/github.com/prometheus/snmp_exporter/generator
vi generator.yml
```
```
modules:
###### Cisco 
  cisco:
#   walk: [sysUpTime, interfaces, ifXTable]
   walk:
   - 1.3.6.1.2.1.2.2.1.1
   - 1.3.6.1.2.1.2.2.1.2
   - 1.3.6.1.2.1.2.2.1.10
   - 1.3.6.1.2.1.2.2.1.13
   - 1.3.6.1.2.1.2.2.1.14
   - 1.3.6.1.2.1.2.2.1.16
   - 1.3.6.1.2.1.2.2.1.19
   - 1.3.6.1.2.1.2.2.1.2
   - 1.3.6.1.2.1.2.2.1.20
   - 1.3.6.1.2.1.2.2.1.5
   - 1.3.6.1.2.1.2.2.1.7
   - 1.3.6.1.2.1.2.2.1.8
   - 1.3.6.1.2.1.31.1.1.1.1
   - 1.3.6.1.2.1.31.1.1.1.18
   - 1.3.6.1.4.1.9.9.48.1.1.1.5
   - 1.3.6.1.4.1.9.9.48.1.1.1.6
   - 1.3.6.1.4.1.9.2.1
   - 1.3.6.1.2.1.1.5
   lookups:
     - source_indexes: [ifIndex]
       lookup: ifAlias
     - source_indexes: [ifIndex]
       lookup: ifDescr
     - source_indexes: [ifIndex]
       # Use OID to avoid conflict with Netscaler NS-ROOT-MIB.
       lookup: 1.3.6.1.2.1.31.1.1.1.1 # ifName
   overrides:
     ifAlias:
       ignore: true # Lookup metric
     ifDescr:
       ignore: true # Lookup metric
     ifName:
       ignore: true # Lookup metric
     ifType:
       type: EnumAsInfo
   version: 2
   max_repetitions: 25
   retries: 3
   timeout: 10s
   auth:
    community: public
```
OID. Thông tin về OID kiểm tra tại đây . https://cric.grenoble.cnrs.fr/Administrateurs/Outils/MIBS/
```
export MIBDIRS=mibs
./generator generate
cp -R /root/go/src/github.com/prometheus/snmp_exporter/generator/snmp.yml /usr/local/snmp_exporter/
systemctl restart snmp_exporter.service
systemctl status snmp_exporter.service
```
https://ipprometheus:9116
- Tạo job trong prometheus để giám sát cisco với nội dung job sau:
```
vi /usr/local/prometheus/prometheus.yml
```
```
####### CISCO
      - targets: ['10.10.10.2']
         # Core switch 01.
        labels:                           
         hostname: Core-SW1
         device: cisco
         company: abc
    scrape_interval: 3m
    scrape_timeout : 3m
    metrics_path: /snmp
    params:
      module: [cisco]
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















