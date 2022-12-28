# Configuração da interface ens192

**Anteriormente, é necessário ter realizado as configurações da VPN no terminal linux para seguir essa etapa.**

Acesse a VPN e altere o arquivo /etc/netplan/00-installer-config.yaml

Comando para alterar o arquivo: 

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

![Visualização do arquivo /etc/netplan/00-installer-config.yaml](https://user-images.githubusercontent.com/98924290/209741608-ec42c4aa-4850-44a5-b690-21dd281a50c6.png)


Após isso, aplique as alterações.

```bash
sudo netplan apply
```

![Aplicando as alterações no arquivo pelo comando: sudo netplan apply](https://user-images.githubusercontent.com/98924290/209741922-710eb30e-d424-4c57-8074-066493431daf.png)

Vizualização da interface ens192 configurada pelo comando:

```bash
ifconfig -a
```
![Comando ifconfig -a](https://user-images.githubusercontent.com/98924290/209742130-19ef7704-b3b3-4391-8b88-4d12a8e8ae69.png)

## Definição dos nomes 

Através do comando abaixo defina os nomes com base na tabela Nomes das Vm´s.

```bash
hostnamectl set-hostname nome_da_tabela
```
*Exemplo:* 

```bash
hostnamectl set-hostname gw.grupo1.turma924.ifalara.local
```

![Definição dos nomes da VPN](https://user-images.githubusercontent.com/98924290/209742431-9d993dbf-310e-4d38-9a58-56b1be9fe0bf.png)

* [Voltar ao roteiro]()
