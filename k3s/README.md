curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.29.4+k3s1" K3S_TOKEN=Start12345! sh -s - server --cluster-init --bind-address 10.0.50.11 --write-kubeconfig-mode 644 --disable traefik --disable servicelb --prefer-bundled-bin --node-name k3s-master01

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.29.4+k3s1" K3S_TOKEN=Start12345! sh -s - server --cluster-init --write-kubeconfig-mode 644 --disable traefik --disable servicelb --prefer-bundled-bin --server https://10.0.50.11:6443 --node-name k3s-master02

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.29.4+k3s1" K3S_TOKEN=Start12345! sh -s - server --cluster-init --write-kubeconfig-mode 644 --disable traefik --disable servicelb --prefer-bundled-bin --server https://10.0.50.11:6443 --node-name k3s-master03

---------------
scp g3n3zyp@10.0.50.11:/etc/rancher/k3s/k3s.yaml /Users/adam/.kube/config

--------------- HELM

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add jetstack https://charts.jetstack.io


--------------- MetalLB LoadBalancer

helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb
kubectl apply -f k3s/metalLB-config.yaml

--------------- Traefik Ingress

helm repo add traefik https://traefik.github.io/charts
helm install --values=k3s/traefik-values.yaml traefik traefik/traefik

--------------- Cert Manager

helm install \
 cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

kubectl apply -f k3s/cloudflare-clusterissuer.yaml

--------------- Longhorn Storage

helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.7.1
kubectl apply -f k3s/longhorn-storageclass-configmap.yaml
kubectl apply -f k3s/longhorn-storageclass.yaml
kubectl apply -f k3s/longhorn-ingress.yaml

