# Descobrir configuração de rede

Para configurar corretamente a rede das VMs vamos
precisar desativar o DHCP automático e configurar manualemente.

A forma mais objetiva de obter as configurações é rodar alguns
comandos no Host na mesma rede que as VMs serão instaladas.

Essa pesquisa precisa ser feita só uma vez.

```bash
# 1. Descobrir o IP da rede do Host. 
# Como vamos usar a rede Wifi, então buscamos pela 
# rede wlp0 (pode variar)

ip a | grep wlp0

# resultado tipico:
# 3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
#     inet 192.168.1.106/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp0s20f3
#          ----------------

# 2. Descobrir o Gateway

ip route | grep wlp0

# resultado típico:
# default via 192.168.1.1 dev wlp0s20f3 proto dhcp src 192.168.1.106 metric 600 
# 192.168.1.0/24 dev wlp0s20f3 proto kernel scope link src 192.168.1.106 metric 600 
# --------------
```

## Interpretando o resultado da pesquisa

IP do Host

    inet 192.168.1.106/24

Isso significa:

- Rede: 192.168.1.0/24
- Máscara: 255.255.255.0
- Range: 192.168.1.1 - 192.168.1.254

Gateway

    default via 192.168.1.1

Para DNS vamos simplesmente usar:

- 8.8.8.8
- 1.1.1.1

Essa pesquina nos dá informação para determinar o range de IPs
do cluster, por exemplo para o control-plane-1 vamos usar final 50:

```bash
# file /etc/netplan/xxx.yaml
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