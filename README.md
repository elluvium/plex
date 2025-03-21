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

To install to a specific namespace:

```bash
helm install plex ./plex-helm --set namespace=media
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
| `namespace`         | Namespace to deploy all resources                                            | `default`     |
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

| Name                             | Description                                             | Value         |
| -------------------------------- | ------------------------------------------------------- | ------------- |
| `persistence.config.enabled`     | Enable config persistence                                | `true`        |
| `persistence.config.size`        | Size of config volume                                   | `20Gi`        |
| `persistence.config.existingClaim` | Use existing PVC for config                          | `""`          |
| `persistence.transcode.enabled`  | Enable transcode directory persistence                  | `true`        |
| `persistence.transcode.size`     | Size of transcode volume                                | `20Gi`        |
| `persistence.media.enabled`      | Enable single media volume persistence                  | `true`        |
| `persistence.media.size`         | Size of single media volume                             | `200Gi`       |
| `persistence.media.mounts`       | List of mount paths within the single media volume      | See values.yaml |

### Node assignment parameters

| Name                              | Description                                            | Value       |
| --------------------------------- | ------------------------------------------------------ | ----------- |
| `nodeAssignment.enabled`          | Enable node assignment for the deployment              | `false`     |
| `nodeAssignment.nodeName`         | Node name to assign the pod to (e.g. "k8s-node-1")    | `""`        |
| `nodeAssignment.useNodeSelector`  | Use nodeSelector instead of nodeName                   | `false`     |
| `nodeAssignment.nodeSelector`     | Node selector labels to use if useNodeSelector=true    | `{}`        |

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

## Namespace Configuration

All resources created by this chart are deployed to the namespace specified in the `namespace` value. This allows for better organization and isolation of your Plex deployment.

To deploy to a specific namespace:

```yaml
namespace: media-apps
```

The namespace must exist before deployment, or you can create it first:

```bash
kubectl create namespace media-apps
helm install plex ./plex-helm --set namespace=media-apps
```

## Single PVC for Media

This chart uses a single PVC for all media with subpaths for different media types:

```yaml
persistence:
  media:
    enabled: true
    size: 200Gi
    mounts:
      - name: movies
        subPath: "movies"
        mountPath: /movies
      - name: tv
        subPath: "tv"
        mountPath: /tv
      # Add more media types as needed
```

This approach:
- Simplifies storage management
- Makes it easier to backup all media at once
- Allows for more efficient use of storage

## Node Assignment

The chart supports assigning the Plex pod to a specific node in two ways:

1. Using `nodeName` to directly assign to a specific node:
```yaml
nodeAssignment:
  enabled: true
  nodeName: "my-media-server-node"
```

2. Using `nodeSelector` for more flexible assignment:
```yaml
nodeAssignment:
  enabled: true
  useNodeSelector: true
  nodeSelector:
    kubernetes.io/hostname: my-node-name
    disktype: ssd
```

This is particularly useful when:
- Running Plex on a specific node with special hardware (like GPUs)
- Ensuring media files are local to the node for performance
- Isolating the media server to specific hardware

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