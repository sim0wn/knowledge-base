# Broken Access Control

## Web3 Sandbox

> Encontre um sandbox de código para escrever contratos em tempo real que foi implantado acidentalmente.

Esse desafio é bem simples e basta analisar o código-fonte da aplicação. Pesquisando `sandbox` no arquivo `main.js`, encontramos o caminho para acessar o sandbox.

<figure><img src="../../.gitbook/assets/ctfjuice_shopbacweb3_sandbox.png" alt=""><figcaption><p>Juice Shop - Web3 Sandbox</p></figcaption></figure>

## Admin Section

> Acesse a seção administrativa da loja.

Para resolver esse desafio, você pode tentar adivinhar onde está localizado o painel do administrador ou descobrir por força bruta usando uma _wordlist_. Nesse caso, o painel está localizado no endpoint `/administration`.

<figure><img src="../../.gitbook/assets/ctfjuice_shopbacadmin_dashboard.png" alt=""><figcaption><p>Juice Shop - Painel do administrador</p></figcaption></figure>

## View Basket

> Veja o carrinho de outro usuário.

Para esse desafio, podemos analisar as requisições que são realizadas ao acessar nosso carrinho.

<figure><img src="../../.gitbook/assets/ctfjuice_shopbacbasket_api_endpoint.png" alt=""><figcaption><p>Juice Shop - Requisição a API ao endpoint do carrinho</p></figcaption></figure>

Perceba a requisição realizada ao endpoint `/rest/basket/1`. É uma chamada à API que retorna os produtos contidos no carrinho do usuário. Se tentarmos alterar o `1` por outro valor (2, por exemplo) e reenviar a requisição, a aplicação não realiza a devida validação e somos capazes de visualizar o carrinho de outro usuário, obtendo sucesso no desafio.
