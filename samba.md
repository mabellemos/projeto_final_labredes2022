# Configuração do Samba

## Objetivo:
 
   * Configurar um servidor compartilhamento de arquivos usando o serviço Samba no linux
   * Acessar o **Gateway Server** via Putty no Windows e depois acessar os servidores **samba**.


## Nome da máquina

```
Tabela 1: Exemplo de nomes dos servidores
-------------------------------------------------------------------------
|    Nome da VM     |                    NOME                           |
-------------------------------------------------------------------------
| Gateway (gw)      | gw.izabel_924.labredes.ifalarapiraca.local        |
| Samba-SRV.        | samba.nycolli_924.labredes.ifalarapiraca.local    |
| NameServer1 (ns1) | ns1.marta_924.labredes.ifalarapiraca.local        |
| NameServer2 (ns2) | ns2.gatinho_924.labredes.ifalarapiraca.local       |
-------------------------------------------------------------------------
```

Conferir os nomes das MV conforme a Tabela 1. Editar o nome da máquina

```bash
$ sudo hostnamectl set-hostname samba-srv
$ reboot
```
OBS: após o reboot o nome da máquina aparecerá no prompt do shell

```
    1. Definir o IP da rede interna para o Samba-SRV

```bash
$ sudo nano /etc/netplan/00-installer-config.yaml
```
![WhatsApp Image 2022-12-23 at 16 51 01](https://user-images.githubusercontent.com/103062733/209842982-0889fac3-a233-4545-b445-550b5b6538f7.jpeg)


```
network:
    ethernets:
        enp0s3:
            addresses: [10.9.24.103/24]
            gateway4: 10.9.24.1
            dhcp4: false 
    version: 2
```

```bash
$ sudo netplan apply
$ ifconfig -a
$ ping 10.9.24.103
```
![WhatsApp Image 2022-12-23 at 15 29 51](https://user-images.githubusercontent.com/103062733/209842436-95365bf1-fed7-4320-ba60-9a9b6eae9dc4.jpeg)

![WhatsApp Image 2022-12-23 at 15 31 27](https://user-images.githubusercontent.com/103062733/209842494-a80a1482-9a4f-46b9-87b2-465ad38ee46f.jpeg)


   2. Na máquina Host faça login via ssh (Use Putty no Windows ou o Terminal no Linux)

Exemplo: $ ssh usuário@ipremoto

```bash
$ ssh administrador@10.9.24.103
```
![WhatsApp Image 2022-12-23 at 15 35 35](https://user-images.githubusercontent.com/103062733/209842701-5b8487e6-3597-4912-9fe7-d56162b952dd.jpeg)


   3. instalar o servidor samba na MV samba-srv

```bash
$ sudo apt update
$ sudo apt install samba
```
![WhatsApp Image 2022-12-23 at 16 45 02](https://user-images.githubusercontent.com/103062733/209842756-c2b84a70-4643-4be6-876c-f93eec38bccc.jpeg)
![WhatsApp Image 2022-12-23 at 16 45 40](https://user-images.githubusercontent.com/103062733/209842804-4e135e94-7327-4464-8f70-1c01b0cd4a17.jpeg)

   
   4. Verfificar se o samba está rodando

```bash
$ whereis samba
samba: /usr/sbin/samba /usr/lib/x86_64-linux-gnu/samba /etc/samba /usr/share/samba /usr/share/man/man8/samba.8.gz /usr/share/man/man7/samba.7.gz


$ sudo systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-03-22 23:07:17 UTC; 1h 26min ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 691 ExecStartPre=/usr/share/samba/update-apparmor-samba-profile (code=exited, status=0/SUCCESS)
   Main PID: 697 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 460)
     Memory: 17.5M
     CGroup: /system.slice/smbd.service
             ├─697 /usr/sbin/smbd --foreground --no-process-group
             ├─737 /usr/sbin/smbd --foreground --no-process-group
             ├─738 /usr/sbin/smbd --foreground --no-process-group
             └─739 /usr/sbin/smbd --foreground --no-process-group

```
![WhatsApp Image 2022-12-23 at 16 58 00](https://user-images.githubusercontent.com/103062733/209843196-b2e697ec-4d6c-4f27-bb44-52d7dc0e2a94.jpeg)


```bash
$ netstat -an | grep LISTEN
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN   
```
![WhatsApp Image 2022-12-23 at 16 59 26](https://user-images.githubusercontent.com/103062733/209843243-a5903260-c4d1-4781-9f5a-7f4a6750f9aa.jpeg)


    5. Faça o backup do arquivo de configuração do samba e cria um arquivo novo somente com os comandos necessários.
    
```bash
$ sudo cp /etc/samba/smb.conf{,.backup}
$ ls -la
-rw-r--r--  1 root root 8942 Mar 22 20:55 smb.conf
-rw-r--r--  1 root root 8942 Mar 23 01:42 smb.conf.backup

$
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
```
![WhatsApp Image 2022-12-23 at 17 02 23](https://user-images.githubusercontent.com/103062733/209843506-6c9d68ae-f3f2-4e43-9813-3188f85c2c15.jpeg)
```bash
$ sudo nano /etc/samba/smb.conf
```

```
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba, Ubuntu)
   interfaces = ens160
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n>
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
 [public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody

   

![image](https://user-images.githubusercontent.com/103062784/210071502-c02d4e25-bc17-4913-9f03-8f6014972c17.png)


  
  
  6. Edite o arquivo de configuração /etc/samba/smb.conf

	* adicione as interfaces da sua máquina na linha "interfaces = 10.9.24.103/8 enp0s3", separando os nomes das interfaces por espaços.
  
```bash
$ sudo nano /etc/samba/smb.conf
```
![WhatsApp Image 2022-12-28 at 13 46 17](https://user-images.githubusercontent.com/103062733/209845075-0daee891-5224-470d-abb7-ab72be3e7ed1.jpeg)


```
[global]
   workgroup = WORKGROUP
   netbios name = samba-srv
   security = user
   server string = %h server (Samba, Ubuntu)
   interfaces = 10.9.24.103/8 enp0s3
   bind interfaces only = yes
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody
   force create mode = 0777
   force directory mode = 0777
```
    * Renicie o serviço smbd
    
```bash
$ sudo systemctl restart smbd
```

   * modifica a pasta /samba/public para acesso a somente usuários do grupo sambashare
   
```
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = no
   valid users = @sambashare
   #guest only = yes
   #force user = nobody
   #force create mode = 0777
   #force directory mode = 0777
```

    * Crie um usuário do S.O para que possa utilizar o compartilhamento samba:
    * usuário: aluno
    * senha: alunoifal
    
```bash
$ sudo adduser aluno
Adding user `aluno' ...
Adding new group `aluno' (1001) ...
Adding new user `aluno' (1001) with group `aluno' ...
Creating home directory `/home/aluno' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for aluno
Enter the new value, or press ENTER for the default
	Full Name []: Aluno de SRED no IFAL Arapiraca
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
```
![WhatsApp Image 2022-12-28 at 13 51 23](https://user-images.githubusercontent.com/103062733/209845746-ca4e5081-801a-4e7e-8e60-9f7b298656d3.jpeg)

    * É necessário vincular o usuário do S.O. ao Serviço Samba. Repita a senha de aluno ou crie uma senha nova somente para acessar o compartilhamento de arquivo. Neste caso repetiremos a senha do usuário aluno
    
```bash
$ sudo smbpasswd -a aluno
New SMB password:
Retype new SMB password:
Added user aluno.

```
![WhatsApp Image 2022-12-28 at 14 15 39](https://user-images.githubusercontent.com/103062733/209849027-d700f5e3-c00e-4da7-ad9c-beb10545c35b.jpeg)

```bash

$ sudo usermod -aG sambashare aluno

```
    
    * O Samba já está instalado, agora precisamos criar um diretório para compartilhá-lo em rede.
   
```bash
$ mkdir /home/<username>/sambashare/
$ sudo mkdir -p /samba/public
```
    * configure as permissões para que qualquer um possa acessar o compartilhamento público.

```bash
sudo chown -R nobody:nogroup /samba/public
sudo chmod -R 0775 /samba/public
sudo chgrp sambashare /samba/public

```
![WhatsApp Image 2022-12-28 at 14 00 35](https://user-images.githubusercontent.com/103062733/209846816-544c7392-725b-4b6e-ab1b-d267f796a950.jpeg)

* [Voltar ao roteiro](https://github.com/mabellemos/projeto_final_labredes2022/blob/main/README.md)

