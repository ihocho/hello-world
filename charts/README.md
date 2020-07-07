## SPC zabbix deployment & Disaster recovery

#### ▣  Workflow

##### spc-mariadb-zabbix
![spc-mariadb-zabbix-workflow](./images/spc-mariadb-zabbix-workflow.png)

---
#### ▣ Pre-requisite
##### 1. Back-up dumpfile uploaded to minio-server in advance
> Refer to spc-zabbix-backup-upload chart

##### 2. Re-define values.yaml for each region
>** parameter values to check specially

- image registry
- image tag
- mariadb.config.bufferPoolSize
- mariadb.resources
- service.mariadb.external.nodePort
- service web nodePort
- zabbix.server.env (especially DB_SERVER_HOST)
- zabbix poller count (especially ZBX_STARTPOLLERS)
- zabbix.config.slack
- minio.name
- minio.url
- minio.user
- minio.password
- minio.bucket
- minio.region

---
#### ▣ mariadb-zabbix chart structure
It is located in the chart directory.
```bash
.
├── CHANGELOG.md
├── Chart.yaml
├── README.md
├── images
│   └── spc-mariadb-zabbix-workflow.png
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── configmap-mariadb.yaml
│   ├── configmap-zabbix-server-env.yaml
│   ├── cronjob-ipmi-collector.yaml
│   ├── cronjob-partition-table-watcher.yaml
│   ├── deploy-mariadb.yaml
│   ├── deploy-zabbix-server.yaml
│   ├── deploy-zabbix-web.yaml
│   ├── postjob-modify-zabbix.yaml
│   ├── postjob-restore-zabbix.yaml
│   ├── rbac.yaml
│   ├── secrets.yaml
│   ├── service-mariadb.yaml
│   ├── service-zabbix-server.yaml
│   └── service-zabbix-web.yaml
├── values
│   ├── values-ccd.yaml
│   ├── values-eu-1b.yaml
│   ├── values-eu-1c.yaml
│   ├── values-kr-1a.yaml
│   ├── values-kr-1c.yaml
│   ├── values-lab-eu.yaml
│   ├── values-sg-1b.yaml
│   ├── values-sg-1c.yaml
│   ├── values-us-1b.yaml
│   ├── values-us-1c.yaml
│   └── values-us.yaml
└── values.yaml
```
---
#### ▣ Register Helm chart (skip if the chart already exists)
- spc repository : spc-chart (http://bespin:global@203.248.18.120:8080/)
```
# repository 등록 (skip if repo already exist)
helm repo add spc-chart http://bespin:global@203.248.18.120:8080

# helm repo ls
NAME            URL
spc-chart       http://bespin:global@203.248.18.120:8080/

# chart 등록
helm package chart/
=> It will generate "spc-mariadb-zabbix-1.1.1.tgz"

helm push spc-mariadb-zabbix-1.1.1.tgz

helm repo update
```

##### 1. List 조회
> helm search repo spc-chart

```bash
NAME                                CHART VERSION   APP VERSION     DESCRIPTION
...
spc-chart/spc-mariadb-zabbix        1.1.1           1.0.0           This chart is a zabbix monitoring system for SP...
spc-chart/spc-zabbix-backup-upload  1.0.0           1.0.0           A Helm chart for back-up zabbix database and up...
...
```

##### 2. Install integrated chart (spc-mariadb-zabbix) for each region
```
namespace = spc-mariadb-zabbix

※ if you install other namespace, rename it.
```

###### 2.1 KR Region
▣ 1A az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-kr-1a.yaml

▣ 1C az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-kr-1c.yaml

###### 2.2 SG Region
▣ 1B az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-sg-1b.yaml

▣ 1C az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-sg-1b.yaml

###### 2.3 US Region
▣ 1B az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-us-1b.yaml

▣ 1C az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-us-1c.yaml

###### 2.4 EU Region
▣ 1B az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-eu-1b.yaml

▣ 1C az
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-eu-1c.yaml

###### 2.5 Lab-EU Region
> helm install "mariadb-zabbix" spc-chart/spc-mariadb-zabbix -n spc-mariadb-zabbix -f values/values-lab-eu.yaml


##### 3. Search deployed helm chart
helm ls -n "namespace"

---
#### ▣ Delete Helm chart
```
chart name = mariadb-zabbix
namespace = spc-mariadb-zabbix

※ if you install other namespace and use other chart-name, rename it.
```
##### 1. Deployed chart 조회

> helm ls -n spc-mariadb-zabbix

##### 2. Delete chart

> helm delete mariadb-zabbix -n spc-mariadb-zabbix

##### 3. Delete PersistPersistentVolumeClaims sparately after removing mariadb-zabbix chart
> kubectl get pvc -n spc-mariadb-zabbix
```
NAME                            STATUS   VOLUME                                     ....
data-mariadb-zabbix-db-0        Bound    pvc-a243e33a-fa4e-4784-a003-04719a6a22ca   ....
```
> kubectl delete pvc "data-mariadb-zabbix-db-0" -n spc-mariadb-zabbix

---
#### ▣ Sample to make values file
1. re-define single value
```
# if you want to change the storage class for mariadb.
# default value : "local-path"
# you want to change the value into "kadalu.replica3"

---
mariadb:
  persistence:
    storageClass: "kadalu.replica3"

```

2. re-define array value
```
# if you want to change the zabbix startPollers.
# default value : "50"
# you want to change the value into "400"

---
zabbix:
  server:
    env:
    - name: ZBX_STARTPOLLERS
      value: "400"                        # <=== 50 to 400

```
---
#### ▣ Troubleshooting

1. If you meet below error, add option --timeout. (default 5 minutes)
```
Error: failed post-install: timed out waiting for the condition
helm.go:75: [debug] failed post-install: timed out waiting for the condition
```

ex) helm install "Release-name" spc-chart/spc-mariadb-zabbix -n "target namespace" --timeout 10m


---
#### ▣ Reference parameters of the chart

| Parameter                                    | Description                                         | Default                                                           |
|----------------------------------------------|-----------------------------------------------------|-------------------------------------------------------------------|
| `image.mariadb.registry`                     | MariaDB image registry                              | `docker.io`                                                       |
| `image.mariadb.repository`                   | MariaDB image repository                            | `bitnami/maridb`                                                  |
| `image.mariadb.tag`                          | MariaDB image tag                                   | `10.4.13-debian-10.-r42`                                         |
| `image.mariadb.pullPolicy`                   | MariaDB image pull policy                           | `IfNotPresent`                                                    |
| `image.mariadbZabbixAgent.registry`          | Zabbix-agent image registory to monitor Mariadb     | `203.248.18.124`                                                  |
| `image.mariadbZabbixAgent.repository`        | Zabbix-agent image repository to monitor Mariadb    | `spc-global/zabbix-agent-debian`                                  |
| `image.mariadbZabbixAgent.tag`               | Zabbix-agent image tag to monitor Mariadb           | `4.4.1-r1`                                                        |
| `image.mariadbZabbixAgent.pullPolicy`        | Zabbix-agent image pull policy to monitor Mariadb   | `IfNotPresent`                                                    |
| `image.zabbixAgent.registry`                 | Zabbix-agent image registory                        | `203.248.18.124`                                                  |
| `image.zabbixAgent.repository`               | Zabbix-agent image repository                       | `spc-global/zabbix-agent`                                         |
| `image.zabbixAgent.tag`                      | Zabbix-agent image tag                              | `4.4.7-r1`                                                        |
| `image.zabbixAgent.pullPolicy`               | Zabbix-agent image pull policy                      | `IfNotPresent`                                                    |
| `image.zabbixServer.registry`                | Zabbix-server image registory                       | `203.248.18.124`                                                  |
| `image.zabbixServer.repository`              | Zabbix-server image repository                      | `spc-global/zabbix-server`                                        |
| `image.zabbixServer.tag`                     | Zabbix-server image tag                             | `4.4.7-r5`                                                        |
| `image.zabbixServer.pullPolicy`              | Zabbix-server image pull policy                     | `IfNotPresent`                                                    |
| `image.zabbixWeb.registry`                   | Zabbix-web image registory                          | `203.248.18.124`                                                  |
| `image.zabbixWeb.repository`                 | Zabbix-web image repository                         | `spc-global/zabbix-web`                                           |
| `image.zabbixWeb.tag`                        | Zabbix-web image tag                                | `4.4.7-r1`                                                        |
| `image.zabbixWeb.pullPolicy`                 | Zabbix-web image pull policy                        | `IfNotPresent`                                                    |
| `image.ipmiCollector.registry`               | IPMI Collector image registry                       | `203.248.18.124`                                                  |
| `image.ipmiCollector.repository`             | IPMI Collector image repository                     | `spc-global/ipmi-collector`                                       |
| `image.ipmiCollector.tag`                    | IPMI Collector image tag                            | `1.1.4`                                                           |
| `image.ipmiCollector.pullPolicy`             | IPMI Collector image pull policy                    | `IfNotPresent`                                                    |
| `image.mariadbClient.registry`               | Mariadb Client image registry                       | `203.248.18.124`                                                  |
| `image.mariadbClient.repository`             | Mariadb Client image repository                     | `spc-global/mysql`                                                |
| `image.mariadbClient.tag`                    | Mariadb Client image tag                            | `5.7`                                                             |
| `image.mariadbClient.pullPolicy`             | Mariadb Client image pull policy                    | `IfNotPresent`                                                    |
| `image.checkResource.registry`               | image registry to check whether target resource is running or completed       | `203.248.18.124`                        |
| `image.checkRestore.repository`              | image repository to check whether target resource is running or completed     | `spc-global/check-resource`             |
| `image.checkRestore.tag`                     | image tag to check whether target resource is running or completed            | `1.0-r1`                                |
| `image.checkRestore.pullPolicy`              | image pull policy to check whether target resource is running or completed    | `IfNotPresent`                          |
| `image.restoreZabbix.registry`               | image registry to restore zabbix database           | `203.248.18.124`                                                  |
| `image.restoreZabbix.repository`             | image repository to restore zabbix database         | `spc-global/restore-zabbix`                                       |
| `image.restoreZabbix.tag`                    | image tag to restore zabbix database                | `1.0-r1`                                                          |
| `image.restoreZabbix.pullPolicy`             | image pull policy to restore zabbix database        | `IfNotPresent`                                                    |
| `image.modifyZabbix.registry`                | image registry to modify zabbix database            | `203.248.18.124`                                                  |
| `image.modifyZabbix.repository`              | image repository to modify zabbix database          | `spc-global/modify-zabbix`                                        |
| `image.modifyZabbix.tag`                     | image tag to modify zabbix database                 | `1.0-r1`                                                          |
| `image.modifyZabbix.pullPolicy`              | image pull policy to modify zabbix database         | `IfNotPresent`                                                    |
| `securityContext.enabled`                    | Enable security context                             | `true`                                                            |
| `securityContext.fsGroup`                    | Group ID for the container                          | `1001`                                                            |
| `securityContext.runAsUser`                  | User ID for the container                           | `1001`                                                            |
| `database.name`                              | default database name                               | `my_db`                                                           |
| `database.root.name`                         | root user name                                      | `root`                                                            |
| `database.root.password`                     | root password                                       | `Bes9in2#4%`                                                      |
| `database.root.forcePassword`                | Force users to specify a password                   | `false`                                                           |
| `database.root.injectSecretsAsVolume`        | Mount admin user password as a file instead of using an environment variable | `false`                                  |
| `database.user.name`                         | user to monitor mariadb                             | `zbxmonitor`                                                      |
| `database.user.password`                     | user password to monitor mariadb                    | `zbxmonitor`                                                      |
| `database.user.forcePassword`                | Force users to specify a password                   | `false`                                                           |
| `database.user.injectSecretsAsVolume`        | Mount admin user password as a file instead of using an environment variable | `false`                                  |
| `database.backup.name`                       | mariadb backup user                                 | `mariabackup`                                                     |
| `database.backup.password`                   | mariadb backup user password                        | `mariabackup`                                                     |
| `database.backup.forcePassword`              | Force users to specify a password                   | `false`                                                           |
| `database.bespin.password`                   | bespin user password                                | `bestpracticeh0stw@y`                                             |
| `database.zabbix.password`                   | zabbix password                                     | `zabbix`                                                          |
| `database.zabbix.password`                   | zabbix password                                     | `zabbix`                                                          |
| `database.zabbix.database`                   | zabbix database                                     | `zabbix`                                                          |
| `mariadb.affinity`                           | mariadb affinity (in addition to master.antiAffinity when set)  | `{}`                                                  |
| `mariadb.antiAffinity`                       | mariadb pod anti-affinity policy                    | `soft`                                                            |
| `mariadb.nodeSelector`                       | mariadb node labels for pod assignment              | `{}`                                                              |
| `mariadb.tolerations`                        | List of node taints to tolerate                     | `[]`                                                              |
| `mariadb.updateStrategy.type`                | mariadb statefulset update strategy policy          | `RollingUpdate`                                                   |
| `mariadb.persistence.enabled`                | Enable persistence using PVC                        | `true`                                                            |
| `mariadb.persistence.mountPath`              | Path to mount the volume at                         | `/bitnami/mariadb`                                                |
| `mariadb.persistence.annotations`            | Persistent Volume Claim annotations                 | `{}`                                                              |
| `mariadb.persistence.accessModes`            | Persistent Volume Access Modes                      | `[ReadWriteOnce]`                                                 |
| `mariadb.persistence.size`                   | Persistent Volume Size                              | `200Gi`                                                           |
| `mariadb.persistence.storageClass`           | Persistent Volume Storage Class                     | `local-path` (Must rancher local-path installed)                  |
| `mariadb.config.bufferPoolSize`              | Innodb buffer pool size                             | `2G`                                                              |
| `mariadb.config.maxConnections`              | max_connections                                     | `1000`                                                            |
| `mariadb.config.waitTimeout`                 | wait_timeout                                        | `28800`                                                           |
| `mariadb.config.netReadTimeout`              | net_read_timeout                                    | `60`                                                              |
| `mariadb.config.netWriteTimeout`             | net_write_timeout                                   | `60`                                                              |
| `mariadb.config.maxAllowedPacket`            | max_allowed_packet                                  | `64M`                                                             |
| `mariadb.resources.requests.cpu`             | cpu resource requested when container creates       | `200m`                                                            |
| `mariadb.resources.requests.memory`          | memory resource requested when container creates    | `1Gi`                                                             |
| `mariadb.resources.limits.cpu`               | Maximum cpu resource container can take             | `200m`                                                            |
| `mariadb.resources.limits.memory`            | Maximum memory resource container can take          | `4Gi`                                                             |
| `mariadb.livenessProbe.enabled`              | Turn on and off liveness probe                      | `true`                                                            |
| `mariadb.livenessProbe.initialDelaySeconds`  | Delay before liveness probe is initiated            | `60`                                                              |
| `mariadb.livenessProbe.periodSeconds`        | How often to perform the probe                      | `10`                                                              |
| `mariadb.livenessProbe.timeoutSeconds`       | When the probe times out                            | `1`                                                               |
| `mariadb.livenessProbe.successThreshold`     | Minimum consecutive successes for the probe         | `1`                                                               |
| `mariadb.livenessProbe.failureThreshold`     | Minimum consecutive failures for the probe          | `3`                                                               |
| `mariadb.readinessProbe.enabled`             | Turn on and off readiness probe                     | `true`                                                            |
| `mariadb.readinessProbe.initialDelaySeconds` | Delay before readiness probe is initiated           | `30`                                                              |
| `mariadb.readinessProbe.periodSeconds`       | How often to perform the probe                      | `10`                                                              |
| `mariadb.readinessProbe.timeoutSeconds`      | When the probe times out                            | `1`                                                               |
| `mariadb.readinessProbe.successThreshold`    | Minimum consecutive successes for the probe         | `1`                                                               |
| `mariadb.readinessProbe.failureThreshold`    | Minimum consecutive failures for the probe          | `3`                                                               |
| `mariadb.podDisruptionBudget.enabled`        | If true, create a pod disruption budget for master pods. | `true`                                                       |
| `mariadb.podDisruptionBudget.minAvailable`   | Minimum number / percentage of pods that should remain scheduled | `1`                                                  |
| `mariadb.service.annotations`                | mariadb service annotations                         | `{}`                                                              |
| `service.mariadb.type`                                    | Kubernetes service type for mariadb                                         | `ClusterIP`                  |
| `service.mariadb.port`                                    | Mariadb  service port                                                       | `3306`                       |
| `service.mariadb.external.type`                           | Service type for accessing from external network                            | `NodePort`                   |
| `service.mariadb.external.port`                           | Internal service port to forward                                            | `3306`                       |
| `service.mariadb.external.nodePort`                       | service port to connect by external client                                  | `30306`                      |
| `service.zabbix.server.type`                              | Kubernetes service type variable                                            | `ClusterIP`                  |
| `service.zabbix.server.ports`                             | Zabbix Server service port list                                             | `[]`                         |
| `service.zabbix.server.ports[0].name`                     | Zabbix Server service port - array [0] 's name                              | `zabbix-trapper`             |
| `service.zabbix.server.ports[0].port`                     | Zabbix Server service port - array [0] 's port                              | `10051`                      |
| `service.zabbix.server.ports[0].targetPort`               | Zabbix Server service port - array [0] 's target port                       | `10051`                      |
| `service.zabbix.server.ports[1].name`                     | Zabbix Server service port - array [1] 's name                              | `zabbix-jmx`                 |
| `service.zabbix.server.ports[1].port`                     | Zabbix Server service port - array [1] 's port                              | `10052`                      |
| `service.zabbix.server.ports[1].targetPort`               | Zabbix Server service port - array [1] 's target port                       | `10052`                      |
| `service.zabbix.web.type`                                 | Kubernetes service type variable                                            | `NodePort`                   |
| `service.zabbix.web.ports`                                | Zabbix Web Front service port list                                          | `[]`                         |
| `service.zabbix.web.ports[0].name`                        | Zabbix Web Front service port - array [0] 's name                           | `https`                      |
| `service.zabbix.web.ports[0].port`                        | Zabbix Web Front service port - array [0] 's port                           | `443`                        |
| `service.zabbix.web.ports[0].targetPort`                  | Zabbix Web Front service port - array [0] 's target port                    | `443`                        |
| `service.zabbix.web.ports[0].nodePort`                    | Zabbix Web Front service port - array [0] 's NodePort                       | `30443`                      |
| `cronjob.ipmiCollector.concurrencyPolicy`                 | ConcurrencyPolicy for IPMICollector cronjob                                 | `Allow`                      |
| `cronjob.ipmiCollector.failedJobsHistoryLimit`            | FailedJobsHistoryLimit for IPMICollector cronjob                            | `1`                          |
| `cronjob.ipmiCollector.successfulJobsHistoryLimit`        | SuccessfulJobsHistoryLimit for IPMICollector cronjob                        | `7`                          |
| `cronjob.ipmiCollector.schedule`                          | Schedule (Daily) for IPMICollector cronjob                                  | `0 1 * * *`                  |
| `cronjob.ipmiCollector.user`                              | Admin user of Zabbix-server                                                 | `Admin`                      |
| `cronjob.ipmiCollector.password`                          | Admin password of Zabbix-server                                             | `zabbix`                     |
| `cronjob.partitioner.concurrencyPolicy`                   | ConcurrencyPolicy for Daily History table partitioning cronjob              | `Allow`                      |
| `cronjob.partitioner.failedJobsHistoryLimit`              | FailedJobsHistoryLimit for Daily History table partitioning cronjob         | `1`                          |
| `cronjob.partitioner.successfulJobsHistoryLimit`          | SuccessfulJobsHistoryLimit for Daily History table partitioning cronjob     | `7`                          |
| `cronjob.partitioner.schedule`                            | Schedule (Daily) for Daily History table partitioning cronjobb              | `0 1 * * *`                  |
| `postjob.modify.zabbix.port`                              | Zabbix web service port                                                     | `443`                        |
| `postjob.modify.zabbix.user`                              | Zabbix web login user                                                       | `Admin`                      |
| `postjob.modify.zabbix.pass`                              | Zabbix web login password                                                   | `zabbix`                     |
| `zabbix.agent.env`                                        | Zabbix agent environment info                                               | `[]`                         |
| `zabbix.agent.env[0].name`                                | Zabbix agent env - array [0]'s name                                         | `ZBX_HOSTNAMEITEM`           |
| `zabbix.agent.env[0].value`                               | Zabbix agent env - array [0]'s value                                        | `system.hostname`            |
| `zabbix.agent.env[1].name`                                | Zabbix agent env - array [1]'s name                                         | `ZBX_METADATAITEM`           |
| `zabbix.agent.env[1].value`                               | Zabbix agent env - array [1]'s value                                        | `system.uname`               |
| `zabbix.agent.env[2].name`                                | Zabbix agent env - array [2]'s name                                         | `ZBX_SERVER_HOST`            |
| `zabbix.agent.env[2].value`                               | Zabbix agent env - array [2]'s value                                        | `127.0.0.1`                  |
| `zabbix.agent.env[3].name`                                | Zabbix agent env - array [3]'s name                                         | `ZBX_HOSTNAME`               |
| `zabbix.agent.env[3].value`                               | Zabbix agent env - array [3]'s value                                        | `mariadb`                    |
| `zabbix.agent.env[4].name`                                | Zabbix agent env - array [4]'s name                                         | `ZBX_TIMEOUT`                |
| `zabbix.agent.env[4].value`                               | Zabbix agent env - array [4]'s value                                        | `15`                         |
| `zabbix.agent.env[5].name`                                | Zabbix agent env - array [5]'s name                                         | `ZBX_ACTIVE_ALLOW`           |
| `zabbix.agent.env[5].value`                               | Zabbix agent env - array [5]'s value                                        | `false`                      |
| `zabbix.web.replicas`                                     | Zabbix web replicas                                                         | `2`                          |
| `zabbix.server.debugLevel`                                | Debug level for Zabbix Server logging                                       | `3`                          |
| `minio.name`                                              | MINIO Server Name                                                           | `minio-local`                |
| `minio.url`                                               | MINIO Server URL                                                            | `http://203.248.18.126:8080` |
| `minio.user`                                              | MINIO AccessKey                                                             | `bespin`                     |
| `minio.password`                                          | MINIO SecretKey                                                             | `Bes9in2#4%`                 |
| `minio.bucket`                                            | Bucket name which manage dumpfile for restoring                             | `bucket`                     |
| `minio.region`                                            | Bucket sub directory. One of "KR" "US" "SG "EU"                             | `test`                       |
| `minio.filename`                                          | Dump file name to restore zabbix database                                   | ``                           |

