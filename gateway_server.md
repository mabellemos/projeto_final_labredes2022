# Introdução

   * O servidor de gateway pode através do NAT (Network Address Translation) fazer o roteamento todos as maquinas de uma rede interna (Ex.: Rede de MV no Virtualbox) para uma rede externa (Ex. Internet) utilizando somente um endereço de rede de saída. Para isso o NAT reescreve os enderecos da fonte (source address) de todos os pacotes de saída para o endereço da interface externa do gateway.

   * As definições de rede da rede externa ao gateway server estão exemplificadas na Tabela 1.

<p><center> Tabela 1: Definições da rede externa</center></p>

| DESCRIÇÃO   |        IP         |
|:------------|:------------------|
| rede        | 10.9.24.0         |
| máscara     | 255.255.255.0     |
| VirtualBox(gateway)| 10.9.24.118|
| Broadcast   | 10.9.24.255/24    |

   * As definições de rede da rede interna ao gateway server estão exemplificadas na Tabela 2.

<p><center> Tabela 2: Definições da rede interna</center></p>

| DESCRIÇÃO   | IP            |
|:------------|:------------- |
| rede        | 10.9.24.0     |
| máscara     | 255.255.255.0 |
| Gateway     | 10.9.24.118   |
| Broadcast   | 10.9.24.255/24|
| NameServer1 | 10.9.24.120   |
| NameServer2 | 10.9.24.110   |
| samba       | 10.9.24.103   |




# Configuração do servidor Gateway como NAT

## Configuração do firewall/NAT

   * Para configurar um servidor como gateway de rede é necessário configurar o firewall do linux (iptables). 
   * as regras do iptables podem ser digitadas no terminal ou podem ser executadas em um script.
   * Com um script, pode-se inicializar as regras do firewall todas as vezes que a máquina for reinicializada.

### habilitar o firewall 
   * Vamos serguir os passos descritos em [1]:
   
   1. habilitar o firewall e permitir o acesso ssh:
```bash

 $ sudo ufw enable
 $ sudo ufw allow ssh
```
![WhatsApp Image 2022-12-28 at 13 40 57](https://user-images.githubusercontent.com/103062784/209848944-9a435042-fab2-4453-958d-a1ae100db003.jpeg)

   2. habilitar o encaminhamento de pacotes das interfaces WAN para LAN, ajustando-se os parâmetros no arquivo **/etc/ufw/sysctl.conf**, removendo-se a marca de comentário (#) da seguinte linha _# net/ipv4/ip_forwarding=1_

```bash
$ sudo nano /etc/ufw/sysctl.conf
``` 
```
...
net/ipv4/ip_forwarding=1
...
```
![WhatsApp Image 2022-12-28 at 13 41 58](https://user-images.githubusercontent.com/103062784/209848923-c8b4a1f1-a319-4731-8132-0ae52a8be6ba.jpeg)


   3. confira o nome das interfaces de rede
```bash
$ ifconfig -a
```
```
WAN interface: enp0s3 
LAN interface: enp0s8
```

   4. Configurar as interfaces de rede (netplan) 

```bash
$ sudo nano /etc/netplan/50-cloud-init.yaml 
```

```
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [10.9.24.118/24]
            dhcp4: false              
    version: 2
```
![image](https://user-images.githubusercontent.com/103062784/209866762-91aa53f1-b946-4a7a-a3d8-a2c61abbf114.png)

```bash
$ sudo netplan apply
$ ifconfig -a
```

   5. no ubuntu 18.04 o arquivo /etc/rc.local não existe mais. Então é necessário recriá-lo.
```bash
$ sudo nano /etc/rc.local
```

   6. A seguir, adicione o seguinte script no arquivo [/etc/rc.local](rc.local)

---
```bash
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.
# Politica padrão para bloquear (drop) todos os pacotes de entrada
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
# Aceita pacotes de entrada a partir das interfaces localhost e the LAN.
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i enp0s8 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
# Aceita pacotes de entrada a partir da WAN se o roteador iniciou a conexao
iptables -A INPUT -i enp0s3 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
# Encaminha os pacotes da LAN para a WAN
iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
# Encaminha os pacotes WAN para a LAN se a LAN inicar a conexao.
iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
# Trafego NAT sai pela interface WAN
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# rc.local needs to exit with 0
# rc.local precisa sair com 0
exit 0
```
![WhatsApp Image 2022-12-28 at 13 50 10](https://user-images.githubusercontent.com/103062784/209849124-48fb8c8a-55bb-441e-98c0-d5876fd7b791.jpeg)

---
   7. converte o arquivo em executável e o torna inicializável no boot
```bash
$ sudo chmod 755 /etc/rc.local
```
   8. verificar se o firewall está funcionando
```bash
$ sudo ufw status
```
![WhatsApp Image 2022-12-28 at 13 51 34](https://user-images.githubusercontent.com/103062784/209849181-71578ac1-a157-441f-a499-38d7d6944337.jpeg)
ou
```bash
$ systemctl status ufw.service
```

   9.  reiniciar a máquina
```bash
$ sudo reboot
```
   10. Nas máquinas SAMBA, NS1 e NS2 ativar o gateway (gateway4: 10.9.24.1) na interface de rede:
```bash
$ sudo nano /etc/netplan/50-cloud-init.yaml
```
```
network:
    ethernets:
        enp0s3:
            addresses: [10.9.24.118/24]
            gateway4: 10.9.24.1
            dhcp4: false
            nameservers:
             addresses:
                - 10.9.24.120
                - 10.9.24.110
                search: [grupo1.turma924.ifalara.local]

    version: 2
```
![image](https://user-images.githubusercontent.com/103062784/209867713-d6b0b5e5-9288-49e2-9064-38e87de5e8b6.png)


```bash
$ sudo netplan apply
$ ifconfig -a
```

  11. Encaminhamento de portas para acesso externo à serviços da rede interna.
  
  * Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente, adicione as informações do IPTABLES sobre portas, IP e Interface no arquivo /etc/rc.local conforme o exemplo abaixo, depois reinicie a máquina:
  
   a. SAMBA: Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente:
        * Portas: 445 e 139
        * Interface Externa aqui é a WAN: enp0s3
        * IP do servidor = 10.0.0.100
        
```bash
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i enp0s3 -p tcp --dport 445 -j DNAT --to 10.0.0.100:445
iptables -A FORWARD -p tcp -d 10.0.0.100 --dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i enp0s3 -p tcp --dport 139 -j DNAT --to 10.0.0.100:139
iptables -A FORWARD -p tcp -d 10.0.0.100 --dport 139 -j ACCEPT
```
![image](https://user-images.githubusercontent.com/103062784/209868442-64ab0113-df98-4bef-842f-731a18bf7964.png)


   b. DNS: Para permitir que o serviço de resolução de nomes (DNS) esteja disponível externamente:
        * Porta: 53
        * Interface Externa aqui é a WAN: enp0s3
        * IP do servidor nameserver1 = 10.0.0.10
        
```bash
#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master interno na porta 53
iptables -A PREROUTING -t nat -i enp0s3 -p udp --dport 53 -j DNAT --to 10.0.0.10:53
iptables -A FORWARD -p udp -d 10.0.0.10 --dport 53 -j ACCEPT
```

# Exercícios

   1. Faça login no *gw* e **ping** para as máquinas *ns1*, *ns2*, e *dh1*.
 
  ![ping_ns1](https://user-images.githubusercontent.com/98924290/209842906-b2dd808b-464d-4b66-8c08-37ef3bb07bba.png)
  ![ping_ns2](https://user-images.githubusercontent.com/98924290/209842919-f122d675-3831-454d-99ae-262d0f5c70cc.png)
  ![ping_samba](https://user-images.githubusercontent.com/98924290/209842934-7f030dec-4146-4fdc-97b9-a4c47ad91a16.png)

   3. Faça login no *ns1* e **ping** para as máquinas *ns2*, *gw*, e *dh1*.
 
 ![ping_ns2_ns1](https://user-images.githubusercontent.com/98924290/209844826-fc5bc1d9-44b1-46c3-972c-11e03e5848db.png)
![ping_gw_ns1](https://user-images.githubusercontent.com/98924290/209844849-9612e734-5be4-4879-ab98-2ccd039100fa.png)
![ping_samba_ns1](https://user-images.githubusercontent.com/98924290/209844873-c8238284-489c-40c2-a908-0924a787fbaf.png)

   5. Faça login no *ns2* e **ping** para as máquinas *ns1*, *gw*, e *dh1*.


   7. Faça login no *samba-srv* e **ping** para as máquinas *gw*, *ns1* e *ns2*.
![ping_gw_samba](https://user-images.githubusercontent.com/98924290/209845005-a90f2ee9-3148-4a95-8074-a2f0be1a29eb.png)
![ping_ns1_samba](https://user-images.githubusercontent.com/98924290/209845027-92d79309-6790-4609-8b3a-e23cbefeea6d.png)

   9. Faça login no *gw* e **nslookup** para *ns1*, *ns2*, e *dh1*.
   10. Faça login no *ns1* e **nslookup** para as máquinas *ns2*, *gw*, e *dh1*.
   11. Faça login no *ns2* e **nslookup** para as máquinas *ns1*, *gw*, e *dh1*.
   12. Faça login no *dh1* e **nslookup** para as máquinas *gw*, *ns1* e *ns2*.
   13. Faça login no *gw* e **dig** para *ns1*, *ns2*, e *dh1*.
   14. Faça login no *ns1* e **dig** para as máquinas *ns2*, *gw*, e *dh1*.
   15. Faça login no *ns2* e **dig** para as máquinas *ns1*, *gw*, e *dh1*.
   16. Faça login no *dh1* e **dig** para as máquinas *gw*, *ns1* e *ns2*.
   17. Faça login no *gw* e **dig -x** para os IPs de *ns1*, *ns2*, e *dh1*.
   18. Faça login no *ns1* e **dig -x** para os IPs de *ns2*, *gw*, e *dh1*.
   19. Faça login no *ns2* e **dig -x** para os IPs de *ns1*, *gw*, e *dh1*.
   20. Faça login no *dh1* e **dig -x** para os IPs de *gw*, *ns1* e *ns2*.

