# Relatório de Resolução do Desafio: Força Bruta em Serviço FTP

Diante deste desafio que foi proposto, foram criadas dois ambientes de MV (Máquinas Virtuais), uma com o SO (Sistema Operacional) Kali Linux e a outra com o ambiente vulnerável proposital Metasploitable, como apresentado nos prints a seguir:

<img width="720" height="469" alt="Captura de tela 2026-11-12 210451" src="https://github.com/user-attachments/assets/f8081cc8-1ba6-4181-a8cf-d5ac2239940e" />


---

Após criadas é necessário descobrir o endereço de IP do ambiente vulnerável do Metasploitable utilizando o terminal com a seguinte linha de comando `ip a` ou utilizando o `ifconfig`, como representado nos prints a seguir:

```bash
ip a
```


---

Com estas informações podemos acessar a máquina vulnerável, mas inicialmente precisamos saber quais os serviços que estão ativos utilizando o terminal com os seguintes comandos:

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.101
```

---

Como verificado no print, o protocolo FTP consta como aberto, portanto é possível efetuar a conexão. Mas diante disto é necessário o nome de usuário e senha para efetuar a conexão. Mas seguindo os padrões reais estas credenciais não estão disponíveis com fácil acesso, mas é possível obter com vazamento de informações de empresas ou com arquivos com várias senhas e usuários padrões para serem efetuados ataques Brute Force, que seriam testes onde listas de usuários e senhas são criadas para serem feitos ataques testando cada combinação.

Criadas as listas de senhas e usuários, é possível fazer o ataque Brute Force utilizando a ferramenta Medusa com a seguinte linha de comando:

```bash
medusa -h 192.168.56.101 -U /root/UsersBruteForceFTP.txt -P /root/PasswordBruteForceFTP.txt -M ftp -v 4
```


---

Com o ataque feito e as credenciais validadas com a ferramenta, é possível efetuar o teste diretamente na máquina com o protocolo FTP utilizando a porta de rede para se chegar ao servidor:

```bash
ftp 192.168.56.101
```

Como de acordo com o print o acesso foi efetuado, e após é possível com o acesso fazer downloads, uploads de informações ou documentos sensíveis, como também renomear e excluir arquivos de acordo com a permissão do usuário.

