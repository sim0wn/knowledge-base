# Sensitive Data Exposure

## Confidential Document

> Acesse um documento confidencial.

Acessando o menu de navegação lateral, é possível encontrar a página "Sobre nós". Nela, é possível observar um link para acesso aos "termos de uso" da aplicação.

<figure><img src="../../.gitbook/assets/ctfjuice_shopdata_exposureceabout_us.png" alt=""><figcaption><p>Juice Shop - Sobre nós</p></figcaption></figure>

Ao abrir ele, somos encaminhados para um diretório `/ftp`:

<figure><img src="../../.gitbook/assets/ctfjuice_shopdata_exposurelegal_md.png" alt=""><figcaption><p>Juice Shop - Termos de Uso</p></figcaption></figure>

Se voltarmos um diretório do arquivo aberto (isto é, acessarmos o diretório `/ftp`), é possível ver a listagem de arquivos desse diretório. Para solucionar o desafio, basta abrir o arquivo `aquisitions.md`.

## Exposed Metrics

> Encontre o endpoint que serve dados de uso para serem analisados por um [sistema de monitoramento popular](https://github.com/prometheus/prometheus).

Esse desafio pode ser resolvido por adivinhação manual ou ataque de força bruta, utilizando uma _wordlist_. Em ambos os casos, basta acessar o endpoint `/metrics`.

## Login MC SafeSearch

> Faça login com o usuário do MC SafeSearch sem usar SQL injection ou qualquer outro bypass.

Para esse desafio, podemos acessar o painel de administrador e obter o e-mail do MC SafeSearch. A partir disso, podemos pesquisar por ele e vamos encontrar o [videoclipe](https://www.youtube.com/watch?v=v59CX2DiX0Y) dele falando sobre como garantir a segurança de suas senhas. Entretanto, ao longo da música, ele acaba acidentalmente expondo sua senha. Por fim, podemos interceptar a requisição de login (visto que o formulário não reconhece a senha) e usar suas credenciais para acessar a conta.
