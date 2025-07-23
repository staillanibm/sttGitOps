# Create the vault namespace
kubectl create ns vault

# Create the TLS secret used by the Ingress for TLS termination
kubectl create secret tls tls-cert -n vault \
    --key="${TLS_PRIVATEKEY_FILE_PATH}" \
    --cert="${TLS_PUBLICKEY_FILE_PATH}"

# Apply the manifests
kubectl apply -k ./k8s

# Initialization of vault
kubectl exec -n vault vault-0 -- vault operator init -key-shares=1 -key-threshold=1

# Unseal the vault (you need to set the UNSEAL_KEY)
kubectl exec -n vault vault-0 -- vault operator unseal $UNSEAL_KEY

# Create the vault token secret
kubectl create secret generic vault-token -n vault \
	--from-literal=token=${VAULT_TOKEN}

kubectl apply -f - <<EOF
apiVersion: external-secrets.io/v1
kind: ClusterSecretStore
metadata:
  name: vault-backend-integration
spec:
  provider:
    vault:
      server: "http://vault.vault:8200"
      path: "integration"
      version: "v2"
      auth:
        tokenSecretRef:
          name: vault-token
          key: token
          namespace: vault
EOF