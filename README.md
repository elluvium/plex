# Plex Media Server Helm Chart

This Helm chart deploys Plex Media Server on a Kubernetes cluster using the LinuxServer.io Docker image.

## Introduction

This chart bootstraps a Plex Media Server deployment on a Kubernetes cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure (for persistent storage)
- (Optional) GPU support for hardware transcoding

## Installing the Chart

To install the chart with the release name `plex`:

```bash
helm install plex ./plex-helm
```

The command deploys Plex Media Server on the Kubernetes cluster with default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `plex` deployment:

```bash
helm uninstall plex
```

## Parameters

### Common parameters

| Name                | Description                                                                  | Value         |
| ------------------- | ---------------------------------------------------------------------------- | ------------- |
| `nameOverride`      | String to partially override plex.fullname template                          | `""`          |
| `fullnameOverride`  | String to fully override plex.fullname template                              | `""`          |
| `replicaCount`      | Number of Plex replicas to deploy                                            | `1`           |

### Image parameters

| Name                | Description                                                  | Value                        |
| ------------------- | ------------------------------------------------------------ | ---------------------------- |
| `image.repository`  | Plex image repository                                        | `lscr.io/linuxserver/plex`  |
| `image.tag`         | Plex image tag (immutable tags are recommended)              | `latest`                     |
| `image.pullPolicy`  | Plex image pull policy                                       | `IfNotPresent`               |

### Environment variables

| Name                | Description                                                  | Value             |
| ------------------- | ------------------------------------------------------------ | ----------------- |
| `env.PUID`          | User ID for permissions                                      | `1000`            |
| `env.PGID`          | Group ID for permissions                                     | `1000`            |
| `env.TZ`            | Timezone                                                     | `Etc/UTC`         |
| `env.VERSION`       | Version of Plex to use                                       | `docker`          |
| `env.PLEX_CLAIM`    | Plex claim token from https://plex.tv/claim                  | `""`              |

### Service parameters

| Name                       | Description                                                   | Value          |
| -------------------------- | ------------------------------------------------------------- | -------------- |
| `service.type`             | Plex service type                                             | `ClusterIP`    |
| `service.hostNetwork`      | Whether to use host network mode                              | `false`        |
| `service.webUIPort`        | Port for the Plex WebUI                                       | `32400`        |

### Persistence parameters

| Name                                | Description                                      | Value         |
| ----------------------------------- | ------------------------------------------------ | ------------- |
| `persistence.config.enabled`        | Enable config persistence                         | `true`        |
| `persistence.config.size`           | Size of config volume                            | `20Gi`        |
| `persistence.config.existingClaim`  | Use existing PVC for config                      | `""`          |
| `persistence.transcode.enabled`     | Enable transcode directory persistence           | `true`        |
| `persistence.transcode.size`        | Size of transcode volume                         | `20Gi`        |
| `persistence.media[*].enabled`      | Enable media volume persistence                  | `true`        |
| `persistence.media[*].size`         | Size of media volumes                            | `100Gi`       |

### Hardware Acceleration parameters

| Name                                | Description                                      | Value       |
| ----------------------------------- | ------------------------------------------------ | ----------- |
| `hardwareAcceleration.intel.enabled` | Enable Intel GPU passthrough                     | `false`     |
| `hardwareAcceleration.nvidia.enabled` | Enable NVIDIA GPU support                        | `false`     |
| `hardwareAcceleration.nvidia.runtime` | NVIDIA runtime to use                           | `nvidia`    |

### Ingress parameters

| Name                         | Description                                            | Value       |
| ---------------------------- | ------------------------------------------------------ | ----------- |
| `ingress.enabled`            | Enable ingress controller resource                     | `false`     |
| `ingress.className`          | IngressClass that will be used                         | `""`        |
| `ingress.hosts[0].host`      | Hostname to your Plex installation                     | `plex.local` |
| `ingress.hosts[0].paths`     | Path within the URL structure                         | See values.yaml |

## Hardware Acceleration

### Intel/AMD GPU

To use Intel/AMD GPU hardware acceleration:

1. Set `hardwareAcceleration.intel.enabled=true`
2. Make sure the node has the GPU available
3. The container will automatically use the GPU for transcoding

### NVIDIA GPU

To use NVIDIA GPU hardware acceleration:

1. Install the [NVIDIA device plugin](https://github.com/NVIDIA/k8s-device-plugin)
2. Set `hardwareAcceleration.nvidia.enabled=true` 
3. Set `hardwareAcceleration.nvidia.runtime=nvidia`
4. Set `hardwareAcceleration.nvidia.visibleDevices=all` (or specify a GPU UUID)

## Upgrading

### From any previous version

If values have changed, make sure to update your values file and apply the changes:

```bash
helm upgrade plex ./plex-helm -f your-values.yaml
```

## Notes

- For the first startup, you should claim your Plex server within 4 minutes using a claim token from https://plex.tv/claim
- You may need to restart the pod after claiming your server
- For best performance, try to use hostNetwork mode when possible
- For transcoding, consider using a memory-backed volume for better performance