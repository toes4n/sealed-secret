# sealed-secret

### 1. Install the Sealed Secrets Controller
Install the server-side controller and CRD into cluster (namespace: kube-system):


kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.32.2/controller.yaml
This deploys the controller and sets up the CRD.​

### 2. Install kubeseal CLI (Client-Side)
Download, extract, and install the ARM64 binary:

wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.33.1/kubeseal-0.33.1-linux-arm64.tar.gz
tar -xvzf kubeseal-0.33.1-linux-arm64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
kubeseal --version   # Ensure it runs on ARM64
This ensures you avoid "Exec format error" and run the correct version for architecture.​
[seal-secrets-releases-official](https://github.com/bitnami-labs/sealed-secrets/releases)

### 3. Create a Kubernetes Secret Manifest
Create a file named secret.yaml with secret data, using base64 encoding:

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  username: dXNlcg==
  password: cGFzc3dvcmQ=

Replace username and password values with own base64-encoded secrets.​

### 4. Seal Secret Using kubeseal
Convert Secret manifest into a SealedSecret:

kubeseal --format yaml < secret.yaml > sealedsecret.yaml
This generates a SealedSecret manifest with encrypted data fields, safe for version control.​​

### 5. Apply the SealedSecret to Cluster
Apply sealed secret:

kubectl apply -f sealedsecret.yaml
The Sealed Secrets controller will decrypt it and create a standard Kubernetes Secret in the correct namespace.​

### 6. Use the Deployed Secret
Reference the secret in Pod/Deployment definitions as usual:

envFrom:
- secretRef:
    name: my-secret
    
Application will access the secret as if it was any standard Kubernetes Secret.​

7. Store Only SealedSecret Manifests in Git
Keep only SealedSecret YAML files (sealedsecret.yaml) in source control.

Never commit plain Kubernetes Secret files.​

### Optional: Compile from Source (if alternate method needed)
If a required binary is missing, you can compile using Go:

go install github.com/bitnami-labs/sealed-secrets/cmd/kubeseal@v0.32.2
$(go env GOPATH)/bin/kubeseal --version
This binary matches build system and always works for ARM64.
