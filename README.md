# project-1
its my first project 18/08/2025
# Scalable Static Website with AWS S3 + CloudFront + Route 53 + Jenkins (on Kubernetes)

## 0) Prereqs
- AWS account, AWS CLI configured: `aws configure`
- A domain in Route 53 (optional but recommended)
- Kubernetes cluster (minikube/kind/EKS) + `kubectl`
- Docker (only if you plan to build a custom Jenkins image)
- Git + Jenkins setup

## 1) Create & Configure S3
```bash
aws s3 mb s3://your-s3-bucket-name --region ap-south-1

# Enable static website hosting (do this in Console -> S3 -> Properties).
# Upload your site:
aws s3 sync ./website s3://your-s3-bucket-name
```

### (Optional) Public Read Bucket Policy
`aws/bucket-policy.json` â€“ replace `your-s3-bucket-name`, then apply:
```bash
aws s3api put-bucket-policy --bucket your-s3-bucket-name --policy file://aws/bucket-policy.json
```

> If you serve via **CloudFront**, you can keep the bucket private by using an Origin Access Control (OAC).

## 2) Create CloudFront Distribution
- Origin: your S3 bucket (website endpoint or S3 origin with OAC)
- Default root object: `index.html`
- Copy the **Distribution ID** and **Domain name**

## 3) Route 53 (Domain â†’ CloudFront)
- Create an **A record (Alias)** for `yourdomain.com` â†’ **Alias to CloudFront distribution**.

## 4) Deploy Jenkins on Kubernetes
Install Nginx Ingress Controller (if your cluster supports LoadBalancers):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Apply Jenkins manifests:
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
# optional if you have public Ingress:
kubectl apply -f k8s/ingress.yaml
```

Access Jenkins:
- With NodePort (minikube): `minikube service -n devops jenkins --url`
- Or browser: `http://<node-ip>:32080`

Initial admin password:
```bash
kubectl exec -n devops deploy/jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword
```

## 5) Jenkins Credentials & Pipeline
Create **Secret Text** credentials:
- `aws-region-text`  â†’ e.g., `ap-south-1`
- `aws-access-key-id`
- `aws-secret-access-key`

Attach IAM policy (`aws/iam-policy-for-jenkins.json`) to a user whose keys you store in Jenkins.

Create a Multibranch or Pipeline job using this repo; Jenkinsfile deploys:
1. Checkout
2. Validate
3. `aws s3 sync`
4. CloudFront invalidation

## 6) Update & Deploy
- Edit files under `website/`
- Push to Git â†’ Jenkins runs â†’ S3 updated â†’ CloudFront invalidated

## 7) Optional TLS for Jenkins Ingress
- Create a TLS secret in `devops` namespace and uncomment the TLS section in `k8s/ingress.yaml`.

## Troubleshooting
- **AccessDenied** to S3 â†’ fix IAM policy or bucket name.
- **403 from CloudFront** â†’ check OAC / bucket policy.
- **Ingress not reachable** â†’ verify LoadBalancer IP and DNS.
- **EmptyDir data loss** â†’ use a PVC for persistent Jenkins home in production.
```