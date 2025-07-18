kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create secret tls tls-cert \
  --cert=$HOME/tls/sttlab.local.crt --key=$HOME/tls/sttlab.local.key \
  -n argocd

kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - argocd.sttlab.local
      secretName: tls-cert
  rules:
    - host: argocd.sttlab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 443
EOF

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

kubectl create namespace argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/install.yaml

kubectl create secret tls tls-cert \
  --cert=$HOME/tls/sttlab.local.crt --key=$HOME/tls/sttlab.local.key \
  -n argo

kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argowf
  namespace: argo
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - argowf.sttlab.local
      secretName: tls-cert
  rules:
    - host: argowf.sttlab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argo-server
                port:
                  number: 2746
EOF

---

# If argocd sync is blocked
argocd app terminate-op msr-contact-management
