<!--- app-name: Apache ZooKeeper -->

# Bitnami package for Apache ZooKeeper

Apache ZooKeeper provides a reliable, centralized register of configuration data and services for distributed applications.

[Overview of Apache ZooKeeper](https://zookeeper.apache.org)

Trademarks: This software listing is packaged by Bitnami. The respective trademarks mentioned in the offering are owned by the respective companies, and use of them does not imply any affiliation or endorsement.

## TL;DR

```console
helm install my-release oci://registry-1.docker.io/bitnamicharts/zookeeper
```

Looking to use Apache ZooKeeper in production? Try [VMware Tanzu Application Catalog](https://bitnami.com/enterprise), the commercial edition of the Bitnami catalog.

## ⚠️ Important Notice: Upcoming changes to the Bitnami Catalog

Beginning August 28th, 2025, Bitnami will evolve its public catalog to offer a curated set of hardened, security-focused images under the new [Bitnami Secure Images initiative](https://news.broadcom.com/app-dev/broadcom-introduces-bitnami-secure-images-for-production-ready-containerized-applications). As part of this transition:

- Granting community users access for the first time to security-optimized versions of popular container images.
- Bitnami will begin deprecating support for non-hardened, Debian-based software images in its free tier and will gradually remove non-latest tags from the public catalog. As a result, community users will have access to a reduced number of hardened images. These images are published only under the “latest” tag and are intended for development purposes
- Starting August 28th, over two weeks, all existing container images, including older or versioned tags (e.g., 2.50.0, 10.6), will be migrated from the public catalog (docker.io/bitnami) to the “Bitnami Legacy” repository (docker.io/bitnamilegacy), where they will no longer receive updates.
- For production workloads and long-term support, users are encouraged to adopt Bitnami Secure Images, which include hardened containers, smaller attack surfaces, CVE transparency (via VEX/KEV), SBOMs, and enterprise support.

These changes aim to improve the security posture of all Bitnami users by promoting best practices for software supply chain integrity and up-to-date deployments. For more details, visit the [Bitnami Secure Images announcement](https://github.com/bitnami/containers/issues/83267).

## Introduction

This chart bootstraps a [ZooKeeper](https://github.com/bitnami/containers/tree/main/bitnami/zookeeper) deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8.0+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release oci://REGISTRY_NAME/REPOSITORY_NAME/zookeeper
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

These commands deploy ZooKeeper on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Configuration and installation details

### Resource requests and limits

Bitnami charts allow setting resource requests and limits for all containers inside the chart deployment. These are inside the `resources` value (check parameter table). Setting requests is essential for production workloads and these should be adapted to your specific use case.

To make this process easier, the chart contains the `resourcesPreset` values, which automatically sets the `resources` section according to different presets. Check these presets in [the bitnami/common chart](https://github.com/bitnami/charts/blob/main/bitnami/common/templates/_resources.tpl#L15). However, in production workloads using `resourcesPreset` is discouraged as it may not fully adapt to your specific needs. Find more information on container resource management in the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).

### Update credentials

Bitnami charts configure credentials at first boot. Any further change in the secrets or credentials require manual intervention. Follow these instructions:

- Update the user password following [the upstream documentation](https://zookeeper.apache.org/documentation.html)
- Update the password secret with the new values (replace the SECRET_NAME, CLIENT_PASSWORD and SERVER_PASSWORD placeholders)

```shell
kubectl create secret generic SECRET_NAME --from-literal=client-password=CLIENT_PASSWORD --from-literal=server-password=SERVER_PASSWORD --dry-run -o yaml | kubectl apply -f -
```

### Prometheus metrics

This chart can be integrated with Prometheus by setting `metrics.enabled` to `true`. This will expose Zookeeper native Prometheus endpoint and a `metrics` service configurable under the `metrics.service` section. It will have the necessary annotations to be automatically scraped by Prometheus.

#### Prometheus requirements

It is necessary to have a working installation of Prometheus or Prometheus Operator for the integration to work. Install the [Bitnami Prometheus helm chart](https://github.com/bitnami/charts/tree/main/bitnami/prometheus) or the [Bitnami Kube Prometheus helm chart](https://github.com/bitnami/charts/tree/main/bitnami/kube-prometheus) to easily have a working Prometheus in your cluster.

#### Integration with Prometheus Operator

The chart can deploy `ServiceMonitor` objects for integration with Prometheus Operator installations. To do so, set the value `metrics.serviceMonitor.enabled=true`. Ensure that the Prometheus Operator `CustomResourceDefinitions` are installed in the cluster or it will fail with the following error:

```text
no matches for kind "ServiceMonitor" in version "monitoring.coreos.com/v1"
```

Install the [Bitnami Kube Prometheus helm chart](https://github.com/bitnami/charts/tree/main/bitnami/kube-prometheus) for having the necessary CRDs and the Prometheus Operator.

### [Rolling vs Immutable tags](https://techdocs.broadcom.com/us/en/vmware-tanzu/application-catalog/tanzu-application-catalog/services/tac-doc/apps-tutorials-understand-rolling-tags-containers-index.html)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

### Configure log level

You can configure the ZooKeeper log level using the `ZOO_LOG_LEVEL` environment variable or the parameter `logLevel`. By default, it is set to `ERROR` because each use of the liveness probe and the readiness probe produces an `INFO` message on connection and a `WARN` message on disconnection, generating a high volume of noise in your logs.

In order to remove that log noise so levels can be set to 'INFO', two changes must be made.

First, ensure that you are not getting metrics via the deprecated pattern of polling 'mntr' on the ZooKeeper client port. The preferred method of polling for Apache ZooKeeper metrics is the ZooKeeper metrics server. This is supported in this chart when setting `metrics.enabled` to `true`.

Second, to avoid the connection/disconnection messages from the probes, you can set custom values for these checks which direct them to the ZooKeeper Admin Server instead of the client port. By default, an Admin Server will be started that listens on `localhost` at port `8080`. The following is an example of this use of the Admin Server for probes:

```yaml
livenessProbe:
  enabled: false
readinessProbe:
  enabled: false
customLivenessProbe:
  exec:
    command: ['/bin/bash', '-c', 'curl -s -m 2 http://localhost:8080/commands/ruok | grep ruok']
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 6
customReadinessProbe:
  exec:
    command: ['/bin/bash', '-c', 'curl -s -m 2 http://localhost:8080/commands/ruok | grep error | grep null']
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 6
```

You can also set the log4j logging level and what log appenders are turned on, by using `ZOO_LOG4J_PROP` set inside of conf/log4j.properties as zookeeper.root.logger by default to

```console
zookeeper.root.logger=INFO, CONSOLE
```

the available appender is

- CONSOLE
- ROLLINGFILE
- RFAAUDIT
- TRACEFILE

### Backup and restore

To back up and restore Helm chart deployments on Kubernetes, you need to back up the persistent volumes from the source deployment and attach them to a new deployment using [Velero](https://velero.io/), a Kubernetes backup/restore tool. Find the instructions for using Velero in [this guide](https://techdocs.broadcom.com/us/en/vmware-tanzu/application-catalog/tanzu-application-catalog/services/tac-doc/apps-tutorials-backup-restore-deployments-velero-index.html).

## Persistence

The [Bitnami ZooKeeper](https://github.com/bitnami/containers/tree/main/bitnami/zookeeper) image stores the ZooKeeper data and configurations at the `/bitnami/zookeeper` path of the container.

Persistent Volume Claims are used to keep the data across deployments. This is known to work in GCE, AWS, and minikube. See the [Parameters](#parameters) section to configure the PVC or to disable persistence.

If you encounter errors when working with persistent volumes, refer to our [troubleshooting guide for persistent volumes](https://docs.bitnami.com/kubernetes/faq/troubleshooting/troubleshooting-persistence-volumes/).

### Adjust permissions of persistent volume mountpoint

As the image run as non-root by default, it is necessary to adjust the ownership of the persistent volume so that the container can write data into it.

By default, the chart is configured to use Kubernetes Security Context to automatically change the ownership of the volume. However, this feature does not work in all Kubernetes distributions.
As an alternative, this chart supports using an initContainer to change the ownership of the volume before mounting it in the final destination.

You can enable this initContainer by setting `volumePermissions.enabled` to `true`.

### Configure the data log directory

You can use a dedicated device for logs (instead of using the data directory) to help avoiding competition between logging and snaphots. To do so, set the `dataLogDir` parameter with the path to be used for writing transaction logs. Alternatively, set this parameter with an empty string and it will result in the log being written to the data directory (Zookeeper's default behavior).

When using a dedicated device for logs, you can use a PVC to persist the logs. To do so, set `persistence.enabled` to `true`. See the [Persistence Parameters](#persistence-parameters) section for more information.

### Set pod affinity

This chart allows you to set custom pod affinity using the `affinity` parameter. Find more information about pod affinity in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

As an alternative, you can use any of the preset configurations for pod affinity, pod anti-affinity, and node affinity available at the [bitnami/common](https://github.com/bitnami/charts/tree/main/bitnami/common#affinities) chart. To do so, set the `podAffinityPreset`, `podAntiAffinityPreset`, or `nodeAffinityPreset` parameters.

## Parameters

### Global parameters

| Name                                                  | Description                                                                                                                                                                                                                                                                                                                                                         | Value   |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `global.imageRegistry`                                | Global Docker image registry                                                                                                                                                                                                                                                                                                                                        | `""`    |
| `global.imagePullSecrets`                             | Global Docker registry secret names as an array                                                                                                                                                                                                                                                                                                                     | `[]`    |
| `global.defaultStorageClass`                          | Global default StorageClass for Persistent Volume(s)                                                                                                                                                                                                                                                                                                                | `""`    |
| `global.storageClass`                                 | DEPRECATED: use global.defaultStorageClass instead                                                                                                                                                                                                                                                                                                                  | `""`    |
| `global.security.allowInsecureImages`                 | Allows skipping image verification                                                                                                                                                                                                                                                                                                                                  | `false` |
| `global.compatibility.openshift.adaptSecurityContext` | Adapt the securityContext sections of the deployment to make them compatible with Openshift restricted-v2 SCC: remove runAsUser, runAsGroup and fsGroup and let the platform use their allowed default IDs. Possible values: auto (apply if the detected running cluster is Openshift), force (perform the adaptation always), disabled (do not perform adaptation) | `auto`  |

### Common parameters

| Name                     | Description                                                                                  | Value           |
| ------------------------ | -------------------------------------------------------------------------------------------- | --------------- |
| `kubeVersion`            | Override Kubernetes version                                                                  | `""`            |
| `nameOverride`           | String to partially override common.names.fullname template (will maintain the release name) | `""`            |
| `fullnameOverride`       | String to fully override common.names.fullname template                                      | `""`            |
| `clusterDomain`          | Kubernetes Cluster Domain                                                                    | `cluster.local` |
| `extraDeploy`            | Extra objects to deploy (evaluated as a template)                                            | `[]`            |
| `commonLabels`           | Add labels to all the deployed resources                                                     | `{}`            |
| `commonAnnotations`      | Add annotations to all the deployed resources                                                | `{}`            |
| `namespaceOverride`      | Override namespace for ZooKeeper resources                                                   | `""`            |
| `usePasswordFiles`       | Mount credentials as files instead of using environment variables                            | `true`          |
| `diagnosticMode.enabled` | Enable diagnostic mode (all probes will be disabled and the command will be overridden)      | `false`         |
| `diagnosticMode.command` | Command to override all containers in the statefulset                                        | `["sleep"]`     |
| `diagnosticMode.args`    | Args to override all containers in the statefulset                                           | `["infinity"]`  |

### ZooKeeper chart parameters

| Name                          | Description                                                                                                                | Value                       |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| `image.registry`              | ZooKeeper image registry                                                                                                   | `REGISTRY_NAME`             |
| `image.repository`            | ZooKeeper image repository                                                                                                 | `REPOSITORY_NAME/zookeeper` |
| `image.digest`                | ZooKeeper image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag                  | `""`                        |
| `image.pullPolicy`            | ZooKeeper image pull policy                                                                                                | `IfNotPresent`              |
| `image.pullSecrets`           | Specify docker-registry secret names as an array                                                                           | `[]`                        |
| `image.debug`                 | Specify if debug values should be set                                                                                      | `false`                     |
| `auth.client.enabled`         | Enable ZooKeeper client-server authentication. It uses SASL/Digest-MD5                                                     | `false`                     |
| `auth.client.clientUser`      | User that will use ZooKeeper clients to auth                                                                               | `""`                        |
| `auth.client.clientPassword`  | Password that will use ZooKeeper clients to auth                                                                           | `""`                        |
| `auth.client.serverUsers`     | Comma, semicolon or whitespace separated list of user to be created                                                        | `""`                        |
| `auth.client.serverPasswords` | Comma, semicolon or whitespace separated list of passwords to assign to users when created                                 | `""`                        |
| `auth.client.existingSecret`  | Use existing secret (ignores previous passwords)                                                                           | `""`                        |
| `auth.quorum.enabled`         | Enable ZooKeeper server-server authentication. It uses SASL/Digest-MD5                                                     | `false`                     |
| `auth.quorum.learnerUser`     | User that the ZooKeeper quorumLearner will use to authenticate to quorumServers.                                           | `""`                        |
| `auth.quorum.learnerPassword` | Password that the ZooKeeper quorumLearner will use to authenticate to quorumServers.                                       | `""`                        |
| `auth.quorum.serverUsers`     | Comma, semicolon or whitespace separated list of users for the quorumServers.                                              | `""`                        |
| `auth.quorum.serverPasswords` | Comma, semicolon or whitespace separated list of passwords to assign to users when created                                 | `""`                        |
| `auth.quorum.existingSecret`  | Use existing secret (ignores previous passwords)                                                                           | `""`                        |
| `tickTime`                    | Basic time unit (in milliseconds) used by ZooKeeper for heartbeats                                                         | `2000`                      |
| `initLimit`                   | ZooKeeper uses to limit the length of time the ZooKeeper servers in quorum have to connect to a leader                     | `10`                        |
| `syncLimit`                   | How far out of date a server can be from a leader                                                                          | `5`                         |
| `preAllocSize`                | Block size for transaction log file                                                                                        | `65536`                     |
| `snapCount`                   | The number of transactions recorded in the transaction log before a snapshot can be taken (and the transaction log rolled) | `100000`                    |
| `maxClientCnxns`              | Limits the number of concurrent connections that a single client may make to a single member of the ZooKeeper ensemble     | `60`                        |
| `maxSessionTimeout`           | Maximum session timeout (in milliseconds) that the server will allow the client to negotiate                               | `40000`                     |
| `heapSize`                    | Size (in MB) for the Java Heap options (Xmx and Xms)                                                                       | `1024`                      |
| `fourlwCommandsWhitelist`     | A list of comma separated Four Letter Words commands that can be executed                                                  | `srvr, mntr, ruok`          |
| `minServerId`                 | Minimal SERVER_ID value, nodes increment their IDs respectively                                                            | `1`                         |
| `listenOnAllIPs`              | Allow ZooKeeper to listen for connections from its peers on all available IP addresses                                     | `false`                     |
| `zooServers`                  | ZooKeeper space separated servers list. Leave empty to use the default ZooKeeper server names.                             | `""`                        |
| `autopurge.snapRetainCount`   | The most recent snapshots amount (and corresponding transaction logs) to retain                                            | `10`                        |
| `autopurge.purgeInterval`     | The time interval (in hours) for which the purge task has to be triggered                                                  | `1`                         |
| `logLevel`                    | Log level for the ZooKeeper server. ERROR by default                                                                       | `ERROR`                     |
| `jvmFlags`                    | Default JVM flags for the ZooKeeper process                                                                                | `""`                        |
| `dataLogDir`                  | Dedicated data log directory                                                                                               | `""`                        |
| `configuration`               | Configure ZooKeeper with a custom zoo.cfg file                                                                             | `""`                        |
| `existingConfigmap`           | The name of an existing ConfigMap with your custom configuration for ZooKeeper                                             | `""`                        |
| `extraEnvVars`                | Array with extra environment variables to add to ZooKeeper nodes                                                           | `[]`                        |
| `extraEnvVarsCM`              | Name of existing ConfigMap containing extra env vars for ZooKeeper nodes                                                   | `""`                        |
| `extraEnvVarsSecret`          | Name of existing Secret containing extra env vars for ZooKeeper nodes                                                      | `""`                        |
| `command`                     | Override default container command (useful when using custom images)                                                       | `["/scripts/setup.sh"]`     |
| `args`                        | Override default container args (useful when using custom images)                                                          | `[]`                        |

### Statefulset parameters

| Name                                                | Description                                                                                                                                                                                                       | Value            |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `replicaCount`                                      | Number of ZooKeeper nodes                                                                                                                                                                                         | `1`              |
| `revisionHistoryLimit`                              | The number of old history to retain to allow rollback                                                                                                                                                             | `10`             |
| `containerPorts.client`                             | ZooKeeper client container port                                                                                                                                                                                   | `2181`           |
| `containerPorts.tls`                                | ZooKeeper TLS container port                                                                                                                                                                                      | `3181`           |
| `containerPorts.follower`                           | ZooKeeper follower container port                                                                                                                                                                                 | `2888`           |
| `containerPorts.election`                           | ZooKeeper election container port                                                                                                                                                                                 | `3888`           |
| `containerPorts.adminServer`                        | ZooKeeper admin server container port                                                                                                                                                                             | `8080`           |
| `containerPorts.metrics`                            | ZooKeeper Prometheus Exporter container port                                                                                                                                                                      | `9141`           |
| `livenessProbe.enabled`                             | Enable livenessProbe on ZooKeeper containers                                                                                                                                                                      | `true`           |
| `livenessProbe.initialDelaySeconds`                 | Initial delay seconds for livenessProbe                                                                                                                                                                           | `30`             |
| `livenessProbe.periodSeconds`                       | Period seconds for livenessProbe                                                                                                                                                                                  | `10`             |
| `livenessProbe.timeoutSeconds`                      | Timeout seconds for livenessProbe                                                                                                                                                                                 | `5`              |
| `livenessProbe.failureThreshold`                    | Failure threshold for livenessProbe                                                                                                                                                                               | `6`              |
| `livenessProbe.successThreshold`                    | Success threshold for livenessProbe                                                                                                                                                                               | `1`              |
| `livenessProbe.probeCommandTimeout`                 | Probe command timeout for livenessProbe                                                                                                                                                                           | `3`              |
| `readinessProbe.enabled`                            | Enable readinessProbe on ZooKeeper containers                                                                                                                                                                     | `true`           |
| `readinessProbe.initialDelaySeconds`                | Initial delay seconds for readinessProbe                                                                                                                                                                          | `5`              |
| `readinessProbe.periodSeconds`                      | Period seconds for readinessProbe                                                                                                                                                                                 | `10`             |
| `readinessProbe.timeoutSeconds`                     | Timeout seconds for readinessProbe                                                                                                                                                                                | `5`              |
| `readinessProbe.failureThreshold`                   | Failure threshold for readinessProbe                                                                                                                                                                              | `6`              |
| `readinessProbe.successThreshold`                   | Success threshold for readinessProbe                                                                                                                                                                              | `1`              |
| `readinessProbe.probeCommandTimeout`                | Probe command timeout for readinessProbe                                                                                                                                                                          | `2`              |
| `startupProbe.enabled`                              | Enable startupProbe on ZooKeeper containers                                                                                                                                                                       | `false`          |
| `startupProbe.initialDelaySeconds`                  | Initial delay seconds for startupProbe                                                                                                                                                                            | `30`             |
| `startupProbe.periodSeconds`                        | Period seconds for startupProbe                                                                                                                                                                                   | `10`             |
| `startupProbe.timeoutSeconds`                       | Timeout seconds for startupProbe                                                                                                                                                                                  | `1`              |
| `startupProbe.failureThreshold`                     | Failure threshold for startupProbe                                                                                                                                                                                | `15`             |
| `startupProbe.successThreshold`                     | Success threshold for startupProbe                                                                                                                                                                                | `1`              |
| `customLivenessProbe`                               | Custom livenessProbe that overrides the default one                                                                                                                                                               | `{}`             |
| `customReadinessProbe`                              | Custom readinessProbe that overrides the default one                                                                                                                                                              | `{}`             |
| `customStartupProbe`                                | Custom startupProbe that overrides the default one                                                                                                                                                                | `{}`             |
| `lifecycleHooks`                                    | for the ZooKeeper container(s) to automate configuration before or after startup                                                                                                                                  | `{}`             |
| `resourcesPreset`                                   | Set container resources according to one common preset (allowed values: none, nano, micro, small, medium, large, xlarge, 2xlarge). This is ignored if resources is set (resources is recommended for production). | `micro`          |
| `resources`                                         | Set container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                 | `{}`             |
| `podSecurityContext.enabled`                        | Enabled ZooKeeper pods' Security Context                                                                                                                                                                          | `true`           |
| `podSecurityContext.fsGroupChangePolicy`            | Set filesystem group change policy                                                                                                                                                                                | `Always`         |
| `podSecurityContext.sysctls`                        | Set kernel settings using the sysctl interface                                                                                                                                                                    | `[]`             |
| `podSecurityContext.supplementalGroups`             | Set filesystem extra groups                                                                                                                                                                                       | `[]`             |
| `podSecurityContext.fsGroup`                        | Set ZooKeeper pod's Security Context fsGroup                                                                                                                                                                      | `1001`           |
| `containerSecurityContext.enabled`                  | Enabled containers' Security Context                                                                                                                                                                              | `true`           |
| `containerSecurityContext.seLinuxOptions`           | Set SELinux options in container                                                                                                                                                                                  | `{}`             |
| `containerSecurityContext.runAsUser`                | Set containers' Security Context runAsUser                                                                                                                                                                        | `1001`           |
| `containerSecurityContext.runAsGroup`               | Set containers' Security Context runAsGroup                                                                                                                                                                       | `1001`           |
| `containerSecurityContext.runAsNonRoot`             | Set container's Security Context runAsNonRoot                                                                                                                                                                     | `true`           |
| `containerSecurityContext.privileged`               | Set container's Security Context privileged                                                                                                                                                                       | `false`          |
| `containerSecurityContext.readOnlyRootFilesystem`   | Set container's Security Context readOnlyRootFilesystem                                                                                                                                                           | `true`           |
| `containerSecurityContext.allowPrivilegeEscalation` | Set container's Security Context allowPrivilegeEscalation                                                                                                                                                         | `false`          |
| `containerSecurityContext.capabilities.drop`        | List of capabilities to be dropped                                                                                                                                                                                | `["ALL"]`        |
| `containerSecurityContext.seccompProfile.type`      | Set container's Security Context seccomp profile                                                                                                                                                                  | `RuntimeDefault` |
| `automountServiceAccountToken`                      | Mount Service Account token in pod                                                                                                                                                                                | `false`          |
| `hostAliases`                                       | ZooKeeper pods host aliases                                                                                                                                                                                       | `[]`             |
| `podLabels`                                         | Extra labels for ZooKeeper pods                                                                                                                                                                                   | `{}`             |
| `podAnnotations`                                    | Annotations for ZooKeeper pods                                                                                                                                                                                    | `{}`             |
| `podAffinityPreset`                                 | Pod affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                               | `""`             |
| `podAntiAffinityPreset`                             | Pod anti-affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                          | `soft`           |
| `nodeAffinityPreset.type`                           | Node affinity preset type. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                         | `""`             |
| `nodeAffinityPreset.key`                            | Node label key to match Ignored if `affinity` is set.                                                                                                                                                             | `""`             |
| `nodeAffinityPreset.values`                         | Node label values to match. Ignored if `affinity` is set.                                                                                                                                                         | `[]`             |
| `affinity`                                          | Affinity for pod assignment                                                                                                                                                                                       | `{}`             |
| `nodeSelector`                                      | Node labels for pod assignment                                                                                                                                                                                    | `{}`             |
| `tolerations`                                       | Tolerations for pod assignment                                                                                                                                                                                    | `[]`             |
| `topologySpreadConstraints`                         | Topology Spread Constraints for pod assignment spread across your cluster among failure-domains. Evaluated as a template                                                                                          | `[]`             |
| `podManagementPolicy`                               | StatefulSet controller supports relax its ordering guarantees while preserving its uniqueness and identity guarantees. There are two valid pod management policies: `OrderedReady` and `Parallel`                 | `Parallel`       |
| `priorityClassName`                                 | Name of the existing priority class to be used by ZooKeeper pods, priority class needs to be created beforehand                                                                                                   | `""`             |
| `schedulerName`                                     | Kubernetes pod scheduler registry                                                                                                                                                                                 | `""`             |
| `updateStrategy.type`                               | ZooKeeper statefulset strategy type                                                                                                                                                                               | `RollingUpdate`  |
| `updateStrategy.rollingUpdate`                      | ZooKeeper statefulset rolling update configuration parameters                                                                                                                                                     | `{}`             |
| `extraVolumes`                                      | Optionally specify extra list of additional volumes for the ZooKeeper pod(s)                                                                                                                                      | `[]`             |
| `extraVolumeMounts`                                 | Optionally specify extra list of additional volumeMounts for the ZooKeeper container(s)                                                                                                                           | `[]`             |
| `sidecars`                                          | Add additional sidecar containers to the ZooKeeper pod(s)                                                                                                                                                         | `[]`             |
| `initContainers`                                    | Add additional init containers to the ZooKeeper pod(s)                                                                                                                                                            | `[]`             |
| `pdb.create`                                        | Deploy a pdb object for the ZooKeeper pod                                                                                                                                                                         | `true`           |
| `pdb.minAvailable`                                  | Minimum available ZooKeeper replicas                                                                                                                                                                              | `""`             |
| `pdb.maxUnavailable`                                | Maximum unavailable ZooKeeper replicas. Defaults to `1` if both `pdb.minAvailable` and `pdb.maxUnavailable` are empty.                                                                                            | `""`             |
| `enableServiceLinks`                                | Whether information about services should be injected into pod's environment variable                                                                                                                             | `true`           |
| `dnsPolicy`                                         | Specifies the DNS policy for the zookeeper pods                                                                                                                                                                   | `""`             |
| `dnsConfig`                                         | allows users more control on the DNS settings for a Pod. Required if `dnsPolicy` is set to `None`                                                                                                                 | `{}`             |

### Traffic Exposure parameters

| Name                                        | Description                                                                             | Value       |
| ------------------------------------------- | --------------------------------------------------------------------------------------- | ----------- |
| `service.type`                              | Kubernetes Service type                                                                 | `ClusterIP` |
| `service.ports.client`                      | ZooKeeper client service port                                                           | `2181`      |
| `service.ports.tls`                         | ZooKeeper TLS service port                                                              | `3181`      |
| `service.ports.follower`                    | ZooKeeper follower service port                                                         | `2888`      |
| `service.ports.election`                    | ZooKeeper election service port                                                         | `3888`      |
| `service.nodePorts.client`                  | Node port for clients                                                                   | `""`        |
| `service.nodePorts.tls`                     | Node port for TLS                                                                       | `""`        |
| `service.disableBaseClientPort`             | Remove client port from service definitions.                                            | `false`     |
| `service.sessionAffinity`                   | Control where client requests go, to the same pod or round-robin                        | `None`      |
| `service.sessionAffinityConfig`             | Additional settings for the sessionAffinity                                             | `{}`        |
| `service.clusterIP`                         | ZooKeeper service Cluster IP                                                            | `""`        |
| `service.loadBalancerIP`                    | ZooKeeper service Load Balancer IP                                                      | `""`        |
| `service.loadBalancerSourceRanges`          | ZooKeeper service Load Balancer sources                                                 | `[]`        |
| `service.externalTrafficPolicy`             | ZooKeeper service external traffic policy                                               | `Cluster`   |
| `service.annotations`                       | Additional custom annotations for ZooKeeper service                                     | `{}`        |
| `service.extraPorts`                        | Extra ports to expose in the ZooKeeper service (normally used with the `sidecar` value) | `[]`        |
| `service.headless.annotations`              | Annotations for the Headless Service                                                    | `{}`        |
| `service.headless.publishNotReadyAddresses` | If the ZooKeeper headless service should publish DNS records for not ready pods         | `true`      |
| `service.headless.servicenameOverride`      | String to partially override headless service name                                      | `""`        |
| `networkPolicy.enabled`                     | Specifies whether a NetworkPolicy should be created                                     | `true`      |
| `networkPolicy.allowExternal`               | Don't require client label for connections                                              | `true`      |
| `networkPolicy.allowExternalEgress`         | Allow the pod to access any range of port and all destinations.                         | `true`      |
| `networkPolicy.extraIngress`                | Add extra ingress rules to the NetworkPolicy                                            | `[]`        |
| `networkPolicy.extraEgress`                 | Add extra ingress rules to the NetworkPolicy                                            | `[]`        |
| `networkPolicy.ingressNSMatchLabels`        | Labels to match to allow traffic from other namespaces                                  | `{}`        |
| `networkPolicy.ingressNSPodMatchLabels`     | Pod labels to match to allow traffic from other namespaces                              | `{}`        |

### Other Parameters

| Name                                          | Description                                                            | Value   |
| --------------------------------------------- | ---------------------------------------------------------------------- | ------- |
| `serviceAccount.create`                       | Enable creation of ServiceAccount for ZooKeeper pod                    | `true`  |
| `serviceAccount.name`                         | The name of the ServiceAccount to use.                                 | `""`    |
| `serviceAccount.automountServiceAccountToken` | Allows auto mount of ServiceAccountToken on the serviceAccount created | `false` |
| `serviceAccount.annotations`                  | Additional custom annotations for the ServiceAccount                   | `{}`    |

### Persistence parameters

| Name                                   | Description                                                                    | Value               |
| -------------------------------------- | ------------------------------------------------------------------------------ | ------------------- |
| `persistence.enabled`                  | Enable ZooKeeper data persistence using PVC. If false, use emptyDir            | `true`              |
| `persistence.existingClaim`            | Name of an existing PVC to use (only when deploying a single replica)          | `""`                |
| `persistence.storageClass`             | PVC Storage Class for ZooKeeper data volume                                    | `""`                |
| `persistence.accessModes`              | PVC Access modes                                                               | `["ReadWriteOnce"]` |
| `persistence.size`                     | PVC Storage Request for ZooKeeper data volume                                  | `8Gi`               |
| `persistence.annotations`              | Annotations for the PVC                                                        | `{}`                |
| `persistence.labels`                   | Labels for the PVC                                                             | `{}`                |
| `persistence.selector`                 | Selector to match an existing Persistent Volume for ZooKeeper's data PVC       | `{}`                |
| `persistence.dataLogDir.size`          | PVC Storage Request for ZooKeeper's dedicated data log directory               | `8Gi`               |
| `persistence.dataLogDir.existingClaim` | Provide an existing `PersistentVolumeClaim` for ZooKeeper's data log directory | `""`                |
| `persistence.dataLogDir.selector`      | Selector to match an existing Persistent Volume for ZooKeeper's data log PVC   | `{}`                |

### Volume Permissions parameters

| Name                                                        | Description                                                                                                                                                                                                                                           | Value                      |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| `volumePermissions.enabled`                                 | Enable init container that changes the owner and group of the persistent volume                                                                                                                                                                       | `false`                    |
| `volumePermissions.image.registry`                          | Init container volume-permissions image registry                                                                                                                                                                                                      | `REGISTRY_NAME`            |
| `volumePermissions.image.repository`                        | Init container volume-permissions image repository                                                                                                                                                                                                    | `REPOSITORY_NAME/os-shell` |
| `volumePermissions.image.digest`                            | Init container volume-permissions image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag                                                                                                                     | `""`                       |
| `volumePermissions.image.pullPolicy`                        | Init container volume-permissions image pull policy                                                                                                                                                                                                   | `IfNotPresent`             |
| `volumePermissions.image.pullSecrets`                       | Init container volume-permissions image pull secrets                                                                                                                                                                                                  | `[]`                       |
| `volumePermissions.resourcesPreset`                         | Set container resources according to one common preset (allowed values: none, nano, micro, small, medium, large, xlarge, 2xlarge). This is ignored if volumePermissions.resources is set (volumePermissions.resources is recommended for production). | `nano`                     |
| `volumePermissions.resources`                               | Set container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                                                     | `{}`                       |
| `volumePermissions.containerSecurityContext.enabled`        | Enabled init container Security Context                                                                                                                                                                                                               | `true`                     |
| `volumePermissions.containerSecurityContext.seLinuxOptions` | Set SELinux options in container                                                                                                                                                                                                                      | `{}`                       |
| `volumePermissions.containerSecurityContext.runAsUser`      | User ID for the init container                                                                                                                                                                                                                        | `0`                        |

### Metrics parameters

| Name                                       | Description                                                                           | Value       |
| ------------------------------------------ | ------------------------------------------------------------------------------------- | ----------- |
| `metrics.enabled`                          | Enable Prometheus to access ZooKeeper metrics endpoint                                | `false`     |
| `metrics.service.type`                     | ZooKeeper Prometheus Exporter service type                                            | `ClusterIP` |
| `metrics.service.port`                     | ZooKeeper Prometheus Exporter service port                                            | `9141`      |
| `metrics.service.annotations`              | Annotations for Prometheus to auto-discover the metrics endpoint                      | `{}`        |
| `metrics.serviceMonitor.enabled`           | Create ServiceMonitor Resource for scraping metrics using Prometheus Operator         | `false`     |
| `metrics.serviceMonitor.namespace`         | Namespace for the ServiceMonitor Resource (defaults to the Release Namespace)         | `""`        |
| `metrics.serviceMonitor.interval`          | Interval at which metrics should be scraped.                                          | `""`        |
| `metrics.serviceMonitor.scrapeTimeout`     | Timeout after which the scrape is ended                                               | `""`        |
| `metrics.serviceMonitor.additionalLabels`  | Additional labels that can be used so ServiceMonitor will be discovered by Prometheus | `{}`        |
| `metrics.serviceMonitor.selector`          | Prometheus instance selector labels                                                   | `{}`        |
| `metrics.serviceMonitor.relabelings`       | RelabelConfigs to apply to samples before scraping                                    | `[]`        |
| `metrics.serviceMonitor.metricRelabelings` | MetricRelabelConfigs to apply to samples before ingestion                             | `[]`        |
| `metrics.serviceMonitor.honorLabels`       | Specify honorLabels parameter to add the scrape endpoint                              | `false`     |
| `metrics.serviceMonitor.jobLabel`          | The name of the label on the target service to use as the job name in prometheus.     | `""`        |
| `metrics.serviceMonitor.scheme`            | The explicit scheme for metrics scraping.                                             | `""`        |
| `metrics.serviceMonitor.tlsConfig`         | TLS configuration used for scrape endpoints used by Prometheus                        | `{}`        |
| `metrics.prometheusRule.enabled`           | Create a PrometheusRule for Prometheus Operator                                       | `false`     |
| `metrics.prometheusRule.namespace`         | Namespace for the PrometheusRule Resource (defaults to the Release Namespace)         | `""`        |
| `metrics.prometheusRule.additionalLabels`  | Additional labels that can be used so PrometheusRule will be discovered by Prometheus | `{}`        |
| `metrics.prometheusRule.rules`             | PrometheusRule definitions                                                            | `[]`        |

### TLS/SSL parameters

| Name                                      | Description                                                                                                                                                                                                               | Value                                                                 |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `tls.client.enabled`                      | Enable TLS for client connections                                                                                                                                                                                         | `false`                                                               |
| `tls.client.auth`                         | SSL Client auth. Can be "none", "want" or "need".                                                                                                                                                                         | `none`                                                                |
| `tls.client.autoGenerated`                | Generate automatically self-signed TLS certificates for ZooKeeper client communications                                                                                                                                   | `false`                                                               |
| `tls.client.existingSecret`               | Name of the existing secret containing the TLS certificates for ZooKeeper client communications                                                                                                                           | `""`                                                                  |
| `tls.client.existingSecretKeystoreKey`    | The secret key from the tls.client.existingSecret containing the Keystore.                                                                                                                                                | `""`                                                                  |
| `tls.client.existingSecretTruststoreKey`  | The secret key from the tls.client.existingSecret containing the Truststore.                                                                                                                                              | `""`                                                                  |
| `tls.client.keystorePath`                 | Location of the KeyStore file used for Client connections                                                                                                                                                                 | `/opt/bitnami/zookeeper/config/certs/client/zookeeper.keystore.jks`   |
| `tls.client.truststorePath`               | Location of the TrustStore file used for Client connections                                                                                                                                                               | `/opt/bitnami/zookeeper/config/certs/client/zookeeper.truststore.jks` |
| `tls.client.passwordsSecretName`          | Existing secret containing Keystore and truststore passwords                                                                                                                                                              | `""`                                                                  |
| `tls.client.passwordsSecretKeystoreKey`   | The secret key from the tls.client.passwordsSecretName containing the password for the Keystore.                                                                                                                          | `""`                                                                  |
| `tls.client.passwordsSecretTruststoreKey` | The secret key from the tls.client.passwordsSecretName containing the password for the Truststore.                                                                                                                        | `""`                                                                  |
| `tls.client.keystorePassword`             | Password to access KeyStore if needed                                                                                                                                                                                     | `""`                                                                  |
| `tls.client.truststorePassword`           | Password to access TrustStore if needed                                                                                                                                                                                   | `""`                                                                  |
| `tls.quorum.enabled`                      | Enable TLS for quorum protocol                                                                                                                                                                                            | `false`                                                               |
| `tls.quorum.auth`                         | SSL Quorum Client auth. Can be "none", "want" or "need".                                                                                                                                                                  | `none`                                                                |
| `tls.quorum.autoGenerated`                | Create self-signed TLS certificates. Currently only supports PEM certificates.                                                                                                                                            | `false`                                                               |
| `tls.quorum.existingSecret`               | Name of the existing secret containing the TLS certificates for ZooKeeper quorum protocol                                                                                                                                 | `""`                                                                  |
| `tls.quorum.existingSecretKeystoreKey`    | The secret key from the tls.quorum.existingSecret containing the Keystore.                                                                                                                                                | `""`                                                                  |
| `tls.quorum.existingSecretTruststoreKey`  | The secret key from the tls.quorum.existingSecret containing the Truststore.                                                                                                                                              | `""`                                                                  |
| `tls.quorum.keystorePath`                 | Location of the KeyStore file used for Quorum protocol                                                                                                                                                                    | `/opt/bitnami/zookeeper/config/certs/quorum/zookeeper.keystore.jks`   |
| `tls.quorum.truststorePath`               | Location of the TrustStore file used for Quorum protocol                                                                                                                                                                  | `/opt/bitnami/zookeeper/config/certs/quorum/zookeeper.truststore.jks` |
| `tls.quorum.passwordsSecretName`          | Existing secret containing Keystore and truststore passwords                                                                                                                                                              | `""`                                                                  |
| `tls.quorum.passwordsSecretKeystoreKey`   | The secret key from the tls.quorum.passwordsSecretName containing the password for the Keystore.                                                                                                                          | `""`                                                                  |
| `tls.quorum.passwordsSecretTruststoreKey` | The secret key from the tls.quorum.passwordsSecretName containing the password for the Truststore.                                                                                                                        | `""`                                                                  |
| `tls.quorum.keystorePassword`             | Password to access KeyStore if needed                                                                                                                                                                                     | `""`                                                                  |
| `tls.quorum.truststorePassword`           | Password to access TrustStore if needed                                                                                                                                                                                   | `""`                                                                  |
| `tls.resourcesPreset`                     | Set container resources according to one common preset (allowed values: none, nano, micro, small, medium, large, xlarge, 2xlarge). This is ignored if tls.resources is set (tls.resources is recommended for production). | `nano`                                                                |
| `tls.resources`                           | Set container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                         | `{}`                                                                  |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
helm install my-release \
  --set auth.clientUser=newUser \
    oci://REGISTRY_NAME/REPOSITORY_NAME/zookeeper
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

The above command sets the ZooKeeper user to `newUser`.

> NOTE: Once this chart is deployed, it is not possible to change the application's access credentials, such as usernames or passwords, using Helm. To change these application credentials after deployment, delete any persistent volumes (PVs) used by the chart and re-deploy it, or use the application's built-in administrative tools if available.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
helm install my-release -f values.yaml oci://REGISTRY_NAME/REPOSITORY_NAME/zookeeper
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.
> **Tip**: You can use the default [values.yaml](https://github.com/bitnami/charts/tree/main/bitnami/zookeeper/values.yaml)

## Troubleshooting

Find more information about how to deal with common errors related to Bitnami's Helm charts in [this troubleshooting guide](https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues).

## Upgrading

### To 13.7.0

This version introduces image verification for security purposes. To disable it, set `global.security.allowInsecureImages` to `true`. More details at [GitHub issue](https://github.com/bitnami/charts/issues/30850).

### To 13.0.0

This major bump changes the following security defaults:

- `runAsGroup` is changed from `0` to `1001`
- `readOnlyRootFilesystem` is set to `true`
- `resourcesPreset` is changed from `none` to the minimum size working in our test suites (NOTE: `resourcesPreset` is not meant for production usage, but `resources` adapted to your use case).
- `global.compatibility.openshift.adaptSecurityContext` is changed from `disabled` to `auto`.

This could potentially break any customization or init scripts used in your deployment. If this is the case, change the default values to the previous ones.

### To 12.0.0

This new version of the chart includes the new ZooKeeper major version 3.9.x. For more information, please refer to [Zookeeper 3.9.0 Release Notes](https://zookeeper.apache.org/doc/r3.9.0/releasenotes.html)

### To 11.0.0

This major version removes `commonAnnotations` and `commonLabels` from `volumeClaimTemplates`. Now annotations and labels can be set in volume claims using `persistence.annotations` and `persistence.labels` values. If the previous deployment has already set `commonAnnotations` and/or `commonLabels` values, to ensure a clean upgrade from previous version without loosing data, please set `persistence.annotations` and/or `persistence.labels` values with the same content as the common values.

### To 10.0.0

This new version of the chart adds support for server-server authentication.
The chart previously supported client-server authentication, to avoid confusion, the previous parameters have been renamed from `auth.*` to `auth.client.*`.

### To 9.0.0

This new version of the chart includes the new ZooKeeper major version 3.8.0. Upgrade compatibility is not guaranteed.

### To 8.0.0

This major release renames several values in this chart and adds missing features, in order to be inline with the rest of assets in the Bitnami charts repository.

Affected values:

- `allowAnonymousLogin` is deprecated.
- `containerPort`, `tlsContainerPort`, `followerContainerPort` and `electionContainerPort` have been regrouped under the `containerPorts` map.
- `service.port`, `service.tlsClientPort`, `service.followerPort`, and  `service.electionPort` have been regrouped under the `service.ports` map.
- `updateStrategy` (string) and `rollingUpdatePartition` are regrouped under the `updateStrategy` map.
- `podDisruptionBudget.*` parameters are renamed to `pdb.*`.

### To 7.0.0

This new version renames the parameters used to configure TLS for both client and quorum.

- `service.tls.disable_base_client_port` is renamed to `service.disableBaseClientPort`
- `service.tls.client_port` is renamed to `service.tlsClientPort`
- `service.tls.client_enable` is renamed to `tls.client.enabled`
- `service.tls.client_keystore_path` is renamed to `tls.client.keystorePath`
- `service.tls.client_truststore_path` is renamed to `tls.client.truststorePath`
- `service.tls.client_keystore_password` is renamed to `tls.client.keystorePassword`
- `service.tls.client_truststore_password` is renamed to `tls.client.truststorePassword`
- `service.tls.quorum_enable` is renamed to `tls.quorum.enabled`
- `service.tls.quorum_keystore_path` is renamed to `tls.quorum.keystorePath`
- `service.tls.quorum_truststore_path` is renamed to `tls.quorum.truststorePath`
- `service.tls.quorum_keystore_password` is renamed to `tls.quorum.keystorePassword`
- `service.tls.quorum_truststore_password` is renamed to `tls.quorum.truststorePassword`

### To 6.1.0

This version introduces `bitnami/common`, a [library chart](https://helm.sh/docs/topics/library_charts/#helm) as a dependency. More documentation about this new utility could be found [here](https://github.com/bitnami/charts/tree/main/bitnami/common#bitnami-common-library-chart). Please, make sure that you have updated the chart dependencies before executing any upgrade.

### To 6.0.0

[On November 13, 2020, Helm v2 support was formally finished](https://github.com/helm/charts#status-of-the-project), this major version is the result of the required changes applied to the Helm Chart to be able to incorporate the different features added in Helm v3 and to be consistent with the Helm project itself regarding the Helm v2 EOL.

### To 5.21.0

A couple of parameters related to Zookeeper metrics were renamed or disappeared in favor of new ones:

- `metrics.port` is renamed to `metrics.containerPort`.
- `metrics.annotations` is deprecated in favor of `metrics.service.annotations`.

### To 3.0.0

This new version of the chart includes the new ZooKeeper major version 3.5.5. Note that to perform an automatic upgrade
of the application, each node will need to have at least one snapshot file created in the data directory. If not, the
new version of the application won't be able to start the service. Please refer to [ZOOKEEPER-3056](https://issues.apache.org/jira/browse/ZOOKEEPER-3056)
in order to find ways to workaround this issue in case you are facing it.

### To 2.0.0

Backwards compatibility is not guaranteed unless you modify the labels used on the chart's statefulsets.
Use the workaround below to upgrade from versions previous to 2.0.0. The following example assumes that the release name is `zookeeper`:

```console
kubectl delete statefulset zookeeper-zookeeper --cascade=false
```

### To 1.0.0

Backwards compatibility is not guaranteed unless you modify the labels used on the chart's deployments.
Use the workaround below to upgrade from versions previous to 1.0.0. The following example assumes that the release name is zookeeper:

```console
kubectl delete statefulset zookeeper-zookeeper --cascade=false
```

## License

Copyright &copy; 2025 Broadcom. The term "Broadcom" refers to Broadcom Inc. and/or its subsidiaries.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.