sudo dnf update -y
sudo reboot

sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

getenforce
# Debe decir: Enforcing

sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=9345/tcp
sudo firewall-cmd --reload


curl -sfL https://get.rke2.io | sudo sh -

sudo mkdir -p /etc/rancher/rke2
sudo nano /etc/rancher/rke2/config.yaml

### config.yaml
token: MI_TOKEN_SUPER_SEGURO
tls-san:
  - rke2.ol10.local
  - 192.168.1.50   # IP del nodo
cni: canal
write-kubeconfig-mode: "0644"



sudo systemctl enable rke2-server --now
sudo systemctl status rke2-server

export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' | sudo tee /etc/profile.d/rke2.sh



kubectl get nodes
kubectl get pods -A




#########################
######  Agente

curl -sfL https://get.rke2.io | sudo sh -
server: https://IP_MASTER:9345
token: MI_TOKEN_SUPER_SEGURO
systemctl enable rke2-agent --now




#####################
###  RANCHER
#####################


curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash


helm version

## Cert Manager
kubectl create namespace cattle-system
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

kubectl get pods -n cert-manager


########  REPO RANCHER
helm repo add rancher https://releases.rancher.com/server-charts/stable
helm repo update

helm install rancher rancher/rancher \
  --namespace cattle-system \
  --set hostname=rancher.midominio.com \
  --set replicas=1 \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=admin@midominio.com

### OPCIONAL  --set ingress.tls.source=secret


kubectl -n cattle-system rollout status deploy/rancher

kubectl get secret --namespace cattle-system bootstrap-secret \
  -o go-template='{{.data.bootstrapPassword | base64decode}}{{"\n"}}'


https://rancher.midominio.com
