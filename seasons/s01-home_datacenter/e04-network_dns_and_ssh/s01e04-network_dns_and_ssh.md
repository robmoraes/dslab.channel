# Network, DNS and SSH

Agora já com o Ubuntu Server instalado já podemos
precisamos configurar o IP fixo, o name server e ativar o
SSH.

## 1. Network

### 1.1. Descobrir configuração de rede

[docs/discovery_network_configs.md](./../../../docs/discovery_network_configs.md)

### 1.2. Fixar IP

```bash
ls /etc/netplan
# p.ex.: 50-cloud-init.yaml

sudo nano /etc/netplan/50-cloud-init.yaml
```

Configurar no arquivo yaml o `IP` para o primeiro `control-plane`:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Aplicar a alteração

```bash
sudo netplan apply
```

## 2. DNS

### 2.1. Definir Hostname

Aplicar o hostname `cp1` ao servidor.

```bash
sudo hostnamectl set-hostname cp1
```

Editar o arquivo `/etc/hosts`

```bash
sudo nano /etc/hosts
```

Colocar as linhas abaixo que ainda não estiverem no arquivo

```text
127.0.0.1      localhost
127.0.1.1      cp1
192.168.1.50   cp1.kube.dslab cp1
```

### 2.2. Configurar DNSs no `etc/hosts` do node

Ainda no `etc/hosts` já deixar preconfiguradas as linhas
correspondentes a todos os nodes do cluster.

```text
192.168.1.51    cp2.kube.dslab
192.168.1.52    cp3.kube.dslab
192.168.1.60    w1.kube.dslab
192.168.1.61    w2.kube.dslab
```

Verificar

```bash
hostname
# deve retornar cp1

ping cp1
```

### 2.2. Configurar DNSs no `etc/hosts` do host

Fora da VM, no laptop que será usado como workstation também
escrever no `/etc/hosts` os IPs e nameservers:

```text
192.168.1.50    cp1.kube.dslab
192.168.1.51    cp2.kube.dslab
192.168.1.52    cp3.kube.dslab
192.168.1.60    w1.kube.dslab
192.168.1.61    w2.kube.dslab
```

## 3. SSH

### 3.1. Habilitar e iniciar o serviço SSH no node

Na instalação do Ubuntu Server deve ter sido incluído
o pacote `OpenSSH`. Nesse passo vamos apenas habilitar e ativar.

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

### 3.2. Criar par de chaves SSH no host

Agora no host (fora da VM) que será o workstation, abrir um
terminal e criar um par de chaves SSH para acessar os nodes.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_dslab -C "kube_dslab"
# passphrase empty

# verificar chaves geradas
ls -lha ~/.ssh | grep id_dslab
```

### 3.3. Copiar a chave pública para o node

```bash
ssh-copy-id -i ~/.ssh/kubelab.pub dslab@cp1.kube.dslab
```

### 3.4. Conectar ao node usando a chave

A partir do host, via terminal, acessar o node com SSH
e a chave criada.

```bash
ssh i ~/.ssh/id_dslab dslab@cp1.kube.dslab
```

A partir daqui toda operação realizada no node será por SSH.
