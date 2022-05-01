```
                ____                  __          ____
               / __ \____  _____ ____/ /_  ____  / / /_
              / /_/ / __ `/ ___/ ___/ __ \/ __ \/ / __/
             / ____/ /_/ (__  |__  ) /_/ / /_/ / / /
            /_/    \__,_/____/____/_.___/\____/_/\__/
            Open source password manager for teams
```

# Passbolt

[Passbolt](https://www.passbolt.com/) is an open sourced password manager.

## Introduction

This chart bootstraps a Passbolt instance.

## Prerequisites

- Kubernetes 1.14+

## Installing the chart

To install the chart:

```bash
$ helm dependency update
$ helm install passbolt .
```

After startup of the main pod, you need to manually create the first administrator with the following command inside the container :

```bash
bin/cake passbolt register_user -u <email> -f <firstname> -l <lastname> -r admin
```

Then, follow the instructions printed on screen.

## Uninstalling the chart

To uninstall/delete the deployment:

```bash
$ helm ls
NAME           REVISION    UPDATED                     STATUS      CHART              NAMESPACE
passbolt    1           Sun May  1 21:27:43 2022    DEPLOYED    passbolt-1.0.0    passbolt
$ helm delete passbolt
```

## Configuration

The following table lists the configurable parameters of the Passbolt chart and their default values.

| Parameter                       | Description                                              | Default                     |
| ------------------------------- | -------------------------------------------------------- | --------------------------- |
| `image.repository`              | image repository                                         | `passbolt`                  |
| `image.tag`                     | `passbolt` image tag.                                    | `3.5.0-ce-non-root`         |
| `image.pullPolicy`              | Image pull policy                                        | `IfNotPresent`              |
| `imagePullSecrets`              | imagePullSecrets to use for private repositories         | `[]`                        |
| `nameOverride`                  | Override chart name                                      | ""                          |
| `fullnameOverride`              | Overrides Chart fullname                                 | ""                          |
| `podAnnotations`                | Custom annotations for pods                              | None                        |
| `podSecurityContext`            | Privilege and access control settings for pods           | `{}`                        |
| `securityContext`               | Privilege and access control settings for containers     | `{}`                        |
| `service.type`                  | Kubernetes service type                                  | `ClusterIP`                 |
| `service.port`                  | Kubernetes service port                                  | `80`                        |
| `podDisruptionBudget.enabled`   | Flag for enabling pod disruption budget                  | true                        |
| `podDisruptionBudget.rule.type` | Rule type of pdb                                         | `minAvailable`              |
| `podDisruptionBudget.rule.type` | Rule value of pdb                                        | `1`                         |
| `ingress.enabled`               | Flag for enabling ingress                                | false                       |
| `persistence.annotations`       | Kubernetes ingress annotations                           | `{}`                        |
| `ingress.labels`                | Ingress additional labels                                | `{}`                        |
| `ingress.hosts[0].host`         | Hostname to your Passbolt installation                   | `passbolt.organization.com` |
| `ingress.hosts[0].paths`        | Paths within the URL structure                           | `[]`                        |
| `ingress.tls`                   | Ingress secrets for TLS certificates                     | `[]`                        |
| `ingress.className`             | Ingress className for multi ingress env and kube v1.19   | `nil`                       |
| `istio.enabled`                 | Determine if using istio in cluster                      | `false`                     |
| `istio.gateway.host`            | Define gateway host for serving incoming traffic         | `nil`                       |
| `istio.gateway.tlsSecretName`   | Define secret where TLS certificate store                | `nil`                       |
| `istio.gateway.existingGateway` | Use existing gateway definition if cluster already have  | `nil`                       |
| `istio.host`                    | Define DNS for receiving incoming traffic                | `nil`                       |
| `email.host`                    | Email server host name                                   | `smtp.example.com`          |
| `email.port`                    | Email server port number                                 | `25`                        |
| `email.from`                    | Email from field                                         | `noreply@example.com`       |
| `email.username`                | Username for login to email server                       | `nil`                       |
| `email.password`                | Password for login to email server                       | `nil`                       |
| `email.timeout`                 | Email default connection timeout                         | `30`                        |
| `env`                           | Environment variables to attach to the pods              | `nil`                       |
| `secretsEnv`                    | Environment variables to attach to the pods from secrets | `nil`                       |
| `persistence.enabled`           | Flag for enabling persistent storage                     | false                       |
| `persistence.existingClaim`     | Do not create a new PVC but use this one                 | None                        |
| `persistence.storageClass`      | Storage class to be used                                 | ""                          |
| `persistence.accessMode`        | Volumes access mode to be set                            | `ReadWriteOnce`             |
| `persistence.size`              | Size of the volume                                       | 512Mi                       |
| `db.host`                       | Hostname of MariaDB                                      | `passbolt-mariadb`          |
| `db.name`                       | Database name used by Passbolt                           | `passbolt`                  |
| `mariadb.enabled`               | Flag for provisioning an embedded MariaDB instance       | true                        |
| `mariadb.fullnameOverride`      | Service name for accessing embedded MariaDB instance     | `passbolt-mariadb`          |
| `mariadb.auth.database`         | Default name for database to create                      | `passbolt`                  |
| `mariadb.auth.username`         | Default username for the database to create              | `passbolt`                  |
| `mariadb.auth.password`         | Default password for the database to create              | `password`                  |
| `passbolt.baseUrl`              | Base URL where Passbolt will be accessible               | `passbolt.organization.com` |
| `passbolt.pro`                  | Flag for enabling pro license                            | false                       |
| `resources`                     | Passbolt Pod resource requests & limits                  | `{}`                        |
| `affinity`                      | Node / Pod affinities                                    | `{}`                        |
| `nodeSelector`                  | Node labels for pod assignment                           | `{}`                        |
| `tolerations`                   | List of node taints to tolerate                          | `[]`                        |

### Connect to external MySQL / MariaDB database

1. Create `db-config.yaml` yaml file to bind local kubernetes dns `db-mysql.<namespace>.svc`(Optional) and credential to connect

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  externalName: mysqldb.organization.com
  type: ExternalName
---
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
data:
  username: <Username in Base64>
  password: <Password in Base64>
```

2. Apply configuration

```shell
$ kubectl apply -f db-config.yaml
```

3. Set the following values of the chart:

```yaml
db:
  host: mysql # or change to IP if possible
  name: passbolt
  existingSecret:
    name: db-creds
    usernameKey: username
    passwordKey: password

mariadb:
  enabled: false
```

### Enabling pro license

1. Create a yaml file `passbolt-key.yaml` with a secret that contains one or more keys to represent the certificates that you want including

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: passbolt-key
data:
  subscription_key.txt: "<Content of subscription_key.txt in Base64>"
```

2. Upload your `passbolt-key.yaml` to a secret in the cluster you are installing Passbolt to.

```shell
$ kubectl apply -f passbolt-key.yaml
```

3. Set the following values of the chart:

```yaml
passbolt:
  license:
    enabled: true
    existingSecret:
      name: passbolt-key
      licenseKey: subscription_key.txt
```
