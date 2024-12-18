# README: Deploying Switchblade with AWS License Manager (LCM)

This guide outlines the steps required to deploy Switchblade with AWS License Manager (LCM) integration for licensing validation.

## Prerequisites

1. **AWS Setup**:
   - Ensure you have the necessary AWS credentials configured.
   - The following permissions are required:
     - `license-manager:CheckoutLicense`
     - `sts:GetCallerIdentity`

2. **AWS License Manager Configuration**:
   - Create the necessary service-linked role for AWS License Manager:
     ```bash
     aws iam create-service-linked-role --aws-service-name license-manager.amazonaws.com
     ```
   - Verify the service-linked role exists:
     ```bash
     aws iam list-roles | grep AWSServiceRoleForLicenseManager
     ```

3. **Kubernetes Setup**:
   - A running Kubernetes cluster (e.g., k3s, EKS).
   - `kubectl` configured to access the cluster.

4. **Docker Image**:
   - Ensure you have the Switchblade Docker image available.
   - Tag and push the image to a container registry accessible by the Kubernetes cluster.

5. **License Manager Product**:
   - Confirm your product is configured in AWS License Manager and associated entitlements are created.

## Deployment Steps

### 1. Create a Namespace
Create a namespace for Switchblade:
```bash
kubectl create namespace operators
```

### 2. Configure Secrets

#### AWS Credentials (Optional)
If not using IAM roles or metadata, create a Kubernetes secret for AWS credentials:
```bash
kubectl create secret generic aws-credentials \
  --from-literal=AWS_ACCESS_KEY_ID=<your-access-key> \
  --from-literal=AWS_SECRET_ACCESS_KEY=<your-secret-key> \
  -n operators
```

### 3. Apply the Deployment Manifest
Use the provided Kubernetes deployment manifest for Switchblade. Update the placeholders with your values:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: switchblade
  namespace: operators
spec:
  replicas: 1
  selector:
    matchLabels:
      app: switchblade
  template:
    metadata:
      labels:
        app: switchblade
    spec:
      containers:
      - name: switchblade
        image: <your-docker-image>
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: AWS_ACCESS_KEY_ID
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: AWS_SECRET_ACCESS_KEY
        - name: LICENSE_KEY
          value: <your-license-key>
        - name: PRODUCT_CODE
          value: <your-product-code>
        - name: ECS_CONTAINER_METADATA_URI_V4
          value: "http://169.254.169.254/latest/meta-data/"
```
Apply the manifest:
```bash
kubectl apply -f switchblade-deployment.yaml
```

### 4. Verify the Deployment
Check the deployment status:
```bash
kubectl get pods -n operators
```

View the logs to ensure the application is running as expected:
```bash
kubectl logs -f <pod-name> -n operators
```

### 5. Validate Licensing
Switchblade will automatically call AWS License Manager to validate licensing. Look for the following log messages to confirm:
- **License activated**
- **Calling AWS checkOutLicense API**
- **Successfully called AWS checkOutLicense API**

## Troubleshooting

1. **Service Role Missing**:
   - If you see `AccessDeniedException: Service role not found`, ensure the `AWSServiceRoleForLicenseManager` role exists.

2. **No Entitlements Allowed**:
   - Check your License Manager configuration and ensure entitlements are correctly associated with your product.

3. **Invalid AWS Credentials**:
   - Ensure the credentials provided are valid and have the necessary permissions.
   - Test using:
     ```bash
     aws sts get-caller-identity
     ```

4. **Metadata Fetching Issues**:
   - If not running on ECS, skip metadata fetching by ensuring `ECS_CONTAINER_METADATA_URI_V4` is not set.

## Additional Notes
- The deployment can be scaled by updating the `replicas` field in the manifest.
- Ensure proper monitoring and logging are in place for production environments.

For further assistance, contact your administrator or refer to the Switchblade documentation.
