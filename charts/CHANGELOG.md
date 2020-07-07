### Change Log
This file documents all notable changes to Mariadb-Zabbix Helm Chart. 

#### 1.1.1
- added sea.sh to zabbix-server for sending alert message to SEA system.
- zabbix server image: 203.248.18.124/spc-global/zabbix-server:4.4.7-r5

#### 1.1.0
- modified mariadb image as 10.4.13-debian-10-r42
- added spec.revisionHistoryLimit = 0 to disable Old Replicaset retain on deployment yaml
- removed annotation : post-upgrade on Job yaml to not execute during upgrade process
- added mariadb nodeport -30306


#### 1.0.0
The first version of chart.
- mariadb image : bitnami/mariadb:10.4.13-debian-10-r32
- zabbix server image : 203.248.18.124/spc-global/zabbix-server:4.4.7-r3
- zabbix web image : 203.248.18.124/spc-global/zabbix-web:4.4.7-r1
- zabbix agent image : 203.248.18.124/spc-global/zabbix-agent:4.4.7-r1
- zabbix agent for mairadb : 203.248.18.124/spc-global/zabbix-agent-debian:4.4.1-r1
- ipmicollector image : 203.248.18.124/spc-global/ipmi-collector:1.1.4
- mariadb client image : 203.248.18.124/spc-global/mysql-mc:5.7
- check-resource image : 203.248.18.124/spc-global/check-restore:1.0-r1
- restore-zabbix image : 203.248.18.124/spc-global/restore-zabbix:1.0-r1
- modify-zabbix image : 203.248.18.124/spc-global/modify-zabbix:1.0-r1
