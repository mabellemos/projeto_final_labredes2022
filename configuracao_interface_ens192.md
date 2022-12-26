# Configuração da interface ens192

Altere o arquivo /etc/netplan/00-installer-config.yaml

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

```bash
# This is the network config written by 'subiquity'
network:
  renderer: networkd
  ethernets:
    ens160:
      dhcp4: false
      addresses: [10.9.24.110/24]           # IP de acesso à VM (Não alterar)
      gateway4: 10.9.24.1                         # IP do gateway (Não alterar)
      nameservers:
         addresses:
           - 10.9.24.1                                      # IP do nameserver (Não alterar)
      #     - 
      #   search: [ ]

    ens192:
      dhcp4: true                                         # alterar para false
      #addresses: [ ]                                   # adicionar o IP/Mascara de acordo com a Planilha de Acomp.
      #gateway4:                                         # IP do gateway da subrede de grupo (Não alterar ainda)
      #nameservers:
      #   addresses:
      #     -                                                     # IP do nameserver do grupo (Não alterar ainda)
      #     - 
      #   search: [ ]
  version: 2

```
