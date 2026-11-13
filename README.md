# Relatório de Resolução do Desafio: Força Bruta em Serviço FTP

Diante deste desafio que foi proposto, foram criadas dois ambientes de MV (Máquinas Virtuais), uma com o SO (Sistema Operacional) Kali Linux e a outra com o ambiente vulnerável proposital Metasploitable, como apresentado nos prints a seguir:

<img width="720" height="469" alt="Captura de tela 2026-11-12 210451" src="https://github.com/user-attachments/assets/f8081cc8-1ba6-4181-a8cf-d5ac2239940e" />

<img width="869" height="659" alt="Captura de tela 2026-11-12 210656" src="https://github.com/user-attachments/assets/119e3149-9b0a-4c29-85e8-e562db913fa8" />

---

Após criadas é necessário descobrir o endereço de IP do ambiente vulnerável do Metasploitable utilizando o terminal com a seguinte linha de comando `ip a` ou utilizando o `ifconfig`, como representado nos prints a seguir:

```bash
ip a
```
<img width="724" height="472" alt="Captura de tela 2026-11-12 210920" src="https://github.com/user-attachments/assets/0fbbc641-e496-4a13-ae80-87f7f1a1061f" />

---

Com estas informações podemos acessar a máquina vulnerável, mas inicialmente precisamos saber quais os serviços que estão ativos utilizando o terminal com o seguinte comando:

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.101
```
<img width="868" height="729" alt="Captura de tela 2026-11-12 211201" src="https://github.com/user-attachments/assets/4bb22dfe-cf7e-4688-a261-f00efbd934e1" />

---

Como verificado no print, o protocolo FTP consta como aberto, portanto é possível efetuar a conexão. Mas diante disto é necessário o nome de usuário e senha para efetuar a conexão. Mas seguindo os padrões reais estas credenciais não estão disponíveis com fácil acesso, mas é possível obter com vazamento de informações de empresas ou com arquivos com várias senhas e usuários padrões para serem efetuados ataques Brute Force, que seriam testes onde listas de usuários e senhas são criadas para serem feitos ataques testando cada combinação.
Criadas as listas de senhas e usuários, é possível fazer o ataque Brute Force utilizando a ferramenta Medusa com a seguinte linha de comando:

```bash
medusa -h 192.168.56.101 -U /root/UsersBruteForceFTP.txt -P /root/PasswordBruteForceFTP.txt -M ftp -t 6
```
<img width="1919" height="341" alt="Captura de tela 2026-11-12 213023" src="https://github.com/user-attachments/assets/f5ea9459-c9eb-4647-a39b-a2ed2e65d206" />


<img width="873" height="729" alt="Captura de tela 2026-11-12 211909" src="https://github.com/user-attachments/assets/8ea3154e-4c01-4508-b46f-fa1a4afd95ca" />


<img width="1918" height="1031" alt="Captura de tela 2026-11-12 212314" src="https://github.com/user-attachments/assets/6b397c1d-2785-4039-9c80-000f9c6f4318" />

---

Com o ataque feito e as credenciais validadas com a ferramenta, é possível efetuar o teste diretamente na máquina com o protocolo FTP utilizando a porta de rede para se chegar ao servidor:

```bash
ftp 192.168.56.101
```
<img width="1919" height="1032" alt="Captura de tela 2026-11-12 212522" src="https://github.com/user-attachments/assets/9c735307-1997-4cdc-b96e-105bb07ca45b" />

Como de acordo com o print o acesso foi efetuado com as credenciais validadas pelo ataque BruteForce, e após é possível com isto efetuar downloads, uploads de informações ou documentos sensíveis, como também renomear e excluir arquivos de acordo com a permissão do usuário.

