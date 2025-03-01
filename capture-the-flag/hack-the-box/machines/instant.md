# Instant

## Resumo

Essa máquina expõe uma aplicação web a qual disponibiliza o link para download de uma aplicação mobile. Essa aplicação mobile integra com uma API, porém as rotas e uma chave de acesso são expostas no código-fonte. Essa API possui uma rota para leitura de arquivos de log, a qual permite a leitura de arquivos locais, configurando uma vulnerabilidade de LFI. Por meio dessa vulnerabilidade de LFI, é possível recuperar a chave privada do usuário shirohige e ganhar acesso remoto à máquina. A partir disso, é possível escalar privilégios recuperando a senha do usuário root armazenada num backup da ferramenta Solar-PuTTY.

## Reconhecimento

Realizei uma varredura de portas com o [Nmap](https://nmap.org/) para identificar os serviços em execução e obtive os seguintes resultados:

```bash
# Nmap 7.92 scan initiated Tue Dec 10 18:49:28 2024 as: nmap -sC -sV --min-rate=1000 -T2 -vvv -p- -oA nmap/full 10.10.11.37
Packet capture filter (device tun0): dst host 10.10.14.169 and (icmp or icmp6 or ((tcp) and (src host 10.10.11.37)))
Nmap scan report for instant.htb (10.10.11.37)
Host is up, received reset ttl 63 (0.15s latency).
Scanned at 2024-12-10 18:49:29 -03 for 84s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 31:83:eb:9f:15:f8:40:a5:04:9c:cb:3f:f6:ec:49:76 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMM6fK04LJ4jNNL950Ft7YHPO9NKONYVCbau/+tQKoy3u7J9d8xw2sJaajQGLqTvyWMolbN3fKzp7t/s/ZMiZNo=
|   256 6f:66:03:47:0e:8a:e0:03:97:67:5b:41:cf:e2:c7:c7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL+zjgyGvnf4lMAlvdgVHlwHd+/U4NcThn1bx5/4DZYY
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.58
|_http-title: Instant Wallet
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read from /usr/bin/../share/nmap: nmap-payloads nmap-service-probes nmap-services.
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Dec 10 18:50:53 2024 -- 1 IP address (1 host up) scanned in 85.28 seconds
```

Na porta 80 estava em execução um servidor web, que ao ser acessado redireciona para o domínio `instant.htb`. Por conta disso, foi necessário adicionar uma entrada ao arquivo `/etc/hosts` para que esse domínio fosse resolvido para o servidor:

```bash
echo $'10.10.11.37\tinstant.htb' | sudo tee -a /etc/hosts
```

A seguinte página web foi retornada ao acessar o endereço `http://instant.htb`:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>HTB Instant — página inicial da aplicação</p></figcaption></figure>

A página sugere fazer o download de uma aplicação para realização de transações bancárias. Realizei uma varredura com o [ffuf](https://github.com/ffuf/ffuf) em busca de subdomínios e uma varredura com o [feroxbuster](https://github.com/epi052/feroxbuster) em busca de outros diretórios. Porém, em ambos os casos, não obtive nada muito relevante, então decidi realizar o download sugerido. Ao clicar em `Download Now`, é realizado o download de uma aplicação Android. Utilizando a ferramenta [jadx](https://github.com/skylot/jadx), descompilei a aplicação para analisar o código-fonte:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>HTB Instant — arquivos principais da aplicação mobile</p></figcaption></figure>

A princípio, pode-se notar que a aplicação implementa um [CRUD](https://developer.mozilla.org/pt-BR/docs/Glossary/CRUD) para autenticação, gerir as transações e o perfil de um usuário autenticado. Além disso, ela utiliza a biblioteca [okhttp3](https://square.github.io/okhttp/), uma biblioteca comumente utilizada para realizar requisições web em aplicações Android. Por conta disso, procurei por requisições que estivessem sendo realizadas, e encontrei o endpoint para uma API:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>HTB Instant — requisição à API exposta</p></figcaption></figure>

Nessa requisição, além do endpoint para a API, também é possível notar que a aplicação passa um token de acesso ao header `Authorization`, o que indica que essa API necessita de um token para ser acessada. Buscando por outras referências ao domínio `instant.htb` com a ferramenta de busca do jadx, encontrei uma referência em que o token de acesso é exposto:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>HTB Instant — token de acesso à API exposto</p></figcaption></figure>

Esse token permite acesso à API como usuário administrador:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption><p>HTB Instant — acesso de administrador à API</p></figcaption></figure>

Entretanto, nenhuma das rotas de API descobertas até então pareciam muito úteis. Explorando um pouco mais a aplicação, encontrei nos Resources um arquivo XML que indicava a quais fontes a aplicação poderia enviar requisições:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>HTB Instant — endpoint de um Swagger encontrado</p></figcaption></figure>

Foi possível encontrar outro subdomínio, o `swagger-ui.instant.htb`. O [Swagger](https://swagger.io/) é uma ferramenta utilizada para expor documentações interativas de APIs. Acessando essa página do Swagger, é possível autenticar-se utilizando o token descoberto anteriormente para realizar as consultas:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption><p>HTB Instant ­­— página do Swagger</p></figcaption></figure>

Além disso, descobri a rota `/api/v1/admin/read/log`. Essa rota espera um parâmetro `log_file_name`, o qual deveria ser o nome de um arquivo de log presente na máquina. Ao enviar a requisição informando a esse parâmetro o valor de exemplo (1.log), ela retorna o conteúdo do arquivo e o caminho absoluto dele (nesse caso, `/home/shirohige/logs/1.log`):

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption><p>HTB Instant — funcionalidade para leitura de logs </p></figcaption></figure>

## Exploração

Levando em consideração o caminho absoluto retornado, tentei voltar um diretório com `../` e acessar o arquivo `.ssh/id_rsa`, onde deveria estar contido a chave privada SSH do usuário `shirohige`. Como resultado, a chave privada do usuário foi retornada:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>HTB Instant — leitura arbitrária de arquivos</p></figcaption></figure>

Salvei esse resultado em JSON num arquivo `id_rsa.json` e processei ele utilizando o jq, o sed e o tr para salvar somente o conteúdo da chave:

```bash
jq -s '.[].["/home/shirohige/logs/../.ssh/id_rsa"] | .[]' id_rsa.json | sed 's/\\n//g' | tr -d '"' | tee id_rsa
```

Alterando as permissões da chave com `chmod 600 id_rsa`, foi possível acessar a máquina por como usuário `shirohige` por meio dessa chave:

```bash
ssh -i id_rsa shirohige@instant.htb
```

## Pós exploração

Durante o processo de enumeração após ganhar acesso inicial à máquina, encontrei um subdiretório `backups` no diretório `/opt` da máquina. Esse diretório continha um subdiretório `Solar-PuTTY` com um arquivo `sessions-backup.dat`. Pesquisando por esse tipo de arquivo, descobri que se tratava de um backup da ferramenta `Solar-PuTTY`, o qual ao ser recuperado pode revelar as credenciais que tinham sido salvas na aplicação. Pesquisando por uma forma de recuperar as credenciais desse arquivo, encontrei um [script](https://gist.github.com/xHacka/052e4b09d893398b04bf8aff5872d0d5) disponível publicamente no GitHub o qual faz essa tarefa. Utilizando o script, encontrei as credenciais para o usuário `root`:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption><p>HTB Instant — credenciais do usuário root</p></figcaption></figure>
