# Kubernetes bootstrap

Sequencia de comandos para preparar o servidor linux
para rodar Kubernets.

```bash
# desabilitar swap (obrigatório para kubernetes)
sudo swapoff -a
sudo nano /etc/fstab
# comentar a linha do swap no fstab

# confirmar desativação da swap
free -h

# modulos de kernel necessarios
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# parâmetros de rede - sysctl para kubernets
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# instalar containerd (runtime de containers)
sudo apt update
sudo apt install -y containerd

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo nano /etc/containerd/config.toml
# SystemdCgroup = true
sudo systemctl restart containerd
sudo systemctl enable containerd

# Instalar Kubernetes packages kubeadm, kubelet, kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```