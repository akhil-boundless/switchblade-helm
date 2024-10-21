# Switchblade Helm Chart

This repository contains a Helm chart for deploying Switchblade to a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.0+

## Chart Details

This Helm chart deploys the following Kubernetes resources:
- Deployment for the Switchblade application
- Service to expose the application
- ServiceAccount for the pods
- Secret for AWS credentials (optional)

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
helm install my-release ./switchblade-helm
```

The command deploys Switchblade on the Kubernetes cluster with default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
helm delete my-release
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

The following table lists the configurable parameters of the Switchblade chart and their default values.

| Parameter                | Description             | Default        |
|--------------------------|-------------------------|----------------|
| `replicaCount`           | Number of replicas      | `1`            |
| `image.name`             | Container image name    | `818674127672.dkr.ecr.us-east-1.amazonaws.com/switchblade` |
| `image.tag`              | Container image tag     | `"latest"`     |
| `service.type`           | Kubernetes service type | `ClusterIP`    |
| `service.port`           | Service port            | `80`           |
| `ingress.enabled`        | Enable ingress          | `false`        |
| `autoscaling.enabled`    | Enable autoscaling      | `false`        |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
helm install my-release ./switchblade-helm --set replicaCount=2
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
helm install my-release ./switchblade-helm -f values.yaml
```

## AWS Credentials

This chart includes a `secret.yaml` file for AWS credentials. To use it:

1. Edit `secret.yaml` and add your base64-encoded AWS credentials:
   ```yaml
   data:
     AWS_ACCESS_KEY_ID: <base64-encoded-access-key>
     AWS_SECRET_ACCESS_KEY: <base64-encoded-secret-key>
   ```
2. Apply the secret to your cluster:
   ```bash
   kubectl apply -f secret.yaml
   ```

**Note:** It's recommended to use more secure methods like IAM roles for service accounts in production environments.

## Configuration and Installation Details

### Image

The `image` section in `values.yaml` allows you to specify the Switchblade image to use. By default, it's set to pull from an AWS ECR repository. Make sure your Kubernetes cluster has the necessary permissions to pull from this repository.

### Ingress

The chart includes an optional ingress resource. To enable it, set `ingress.enabled` to `true` and configure the `ingress` section in `values.yaml` according to your cluster setup.

### Autoscaling

Horizontal Pod Autoscaling can be enabled by setting `autoscaling.enabled` to `true`. Configure the `autoscaling` section in `values.yaml` to set the desired minimum and maximum replica counts and target CPU/memory utilization percentages.

## Upgrading the Chart

To upgrade the chart:

```bash
helm upgrade my-release ./switchblade-helm
```

## Support

For any issues or questions, please file an issue in the GitHub repository.
```

This README provides a comprehensive guide for users of your Helm chart. It includes sections on prerequisites, installation, configuration, and usage. The Parameters section gives an overview of the main configurable options, and there are additional sections covering AWS credentials, image configuration, ingress, and autoscaling.

Feel free to adjust any parts of this README to better fit your specific use case or to add any additional information you think would be helpful for users of your Helm chart.