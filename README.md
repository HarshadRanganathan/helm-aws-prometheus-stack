# helm-aws-prometheus-stack

Helm chart for setting up:
- The Prometheus Operator
- Highly available Prometheus
- Thanos Sidecar
- Highly available Alertmanager
- Prometheus node-exporter
- kube-state-metrics

Chart Reference - https://github.com/prometheus-operator/kube-prometheus

## Pre-requisites

### Namespace

Create a new namespace `platform` where we will install the `kube-prometheus-stack`.

```bash
kubectl create namespace platform
```

### IAM

We will be using IRSA (IAM Roles for Service Accounts) to give the required permissions to the Kube Prometheus Stack.

`Note: You need to create an OIDC provider for your cluster to make use of IRSA. Refer - https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html`

1. Create a new IAM policy `prometheus-pol` with the policy document at `iam/policy.json`

Replace `${bucket_name}` placeholder with the S3 bucket where you want the prometheus metrics to be stored

2. Create a new IAM role `prometheus-rol` and attach the IAM policy `prometheus-pol`

3. Update the trust relationship of the IAM role `prometheus-rol` as below replacing the `account_id`, `eks_cluster_id` and `region` with the appropriate values.

This trust relationship allows pods with serviceaccount `prometheus` in `platform` namespace to assume the role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account_id>:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/<eks_cluster_id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<eks_cluster_id>:sub": "system:serviceaccount:platform:prometheus"
        }
      }
    }
  ]
}
```

### Service Account

Create a new service account in the `platform` namespace and associate it with the IAM role which we had created earlier.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: platform
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/prometheus-rol
EOF
```

### Config Updates

1. Update `stages/prod/prod-values.yaml` file with appropriate values.

| | |
|--|--|
|hosts |Hostname to which the rules apply  |
|external-dns.alpha.kubernetes.io/hostname |Hostname to be used for DNS records |
|alb.ingress.kubernetes.io/certificate-arn |ACM certificate for TLS connection |
|externalLabels |External labels to be exported along with the metrics - Update with the EKS cluster name |

## Install/Upgrade Chart

Run below commands to set up prometheus stack:

```
# create config secret
kubectl -n platform create secret generic thanos-objstore-config --from-file=thanos.yaml=stages/prod/thanos-config.yaml

# helm install/upgrade
helm upgrade -i prometheus . -n platform --values=stages/shared-values.yaml --values=stages/prod/prod-values.yaml
```

## Accessing AlertManager UI

To visit the Alertmanager UI you can run

```
kubectl --namespace platform  port-forward svc/prometheus-alertmanager 9093
```

then visit localhost:9093

## Reload AlertManager Config

If the prometheus operator does not reload the config automatically you can manually initiate the config reload by hitting below endpoint after port forwarding:

```
curl -X POST http://127.0.0.1:9093/-/reload
```
