# openshift-langfuse

Minimal instructions for getting langfuse running on OpenShift

- https://github.com/langfuse/langfuse-k8s/blob/main/examples/minimal-installation/README.md
- https://langfuse.com/self-hosting
- https://langfuse.com/blog/2025-06-04-open-sourcing-langfuse-product

```bash
oc new-project langfuse

oc apply -f- <<'EOF'
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: langfuse
  namespace: langfuse
stringData:
  salt: PUSBWRJmBns3IBIMNb6q+/n+3EHyQ987uVXpbgXiF/Q=  # openssl rand -base64 32
  encryption-key: 42b2b176228e7c9c34379dd000a35b292d10617b4a3fe8ba9b9cacc9508ccb30  # Generate with openssl rand -hex 32
  nextauth-secret: password
  postgresql-password: password
  clickhouse-password: password
  redis-password: password
  s3-user: user
  s3-password: password
EOF

export SALT=PUSBWRJmBns3IBIMNb6q+/n+3EHyQ987uVXpbgXiF/Q=

wget https://raw.githubusercontent.com/langfuse/langfuse-k8s/refs/heads/main/examples/minimal-installation/values.yaml

helm repo add langfuse https://langfuse.github.io/langfuse-k8s
helm repo update

helm install langfuse langfuse/langfuse -n langfuse -f values.yaml --debug

oc create route edge --service= langfuse-web --insecure-policy=Redirect
```

if you want some default login values (else use the create user) you can do this

```bash
oc set env deployment/langfuse-web \
  LANGFUSE_INIT_ORG_ID="acme-org" \
  LANGFUSE_INIT_ORG_NAME="ACME ORG" \
  LANGFUSE_INIT_ORG_CLOUD_PLAN="Team" \
  LANGFUSE_INIT_PROJECT_ID="7a88fb47-b4e2-43b8-a06c-a5ce950dc53a" \
  LANGFUSE_INIT_PROJECT_NAME="llm" \
  LANGFUSE_INIT_PROJECT_PUBLIC_KEY="pk-lf-1234567890" \
  LANGFUSE_INIT_PROJECT_SECRET_KEY="pk-lf-1234567890" \
  LANGFUSE_INIT_USER_EMAIL="admin@acme.com" \
  LANGFUSE_INIT_USER_NAME="admin" \
  LANGFUSE_INIT_USER_PASSWORD="password123"
```
