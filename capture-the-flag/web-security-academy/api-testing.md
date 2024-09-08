# Teste de API

## Utilizando a documentação para explorar um endpoint de API

> Para solucionar esse laboratório, encontre a documentação da API e remova o `carlos`. Você pode autenticar-se utilizando as credenciais `wiener:peter`.

Esse laboratório é bem simples, e basta apenas um pouco de noção sobre API e navegar pela aplicação.

<figure><img src="../../.gitbook/assets/ctfwsaapi_testinglab_Iindex.png" alt=""><figcaption><p>Laboratório I - Página inicial</p></figcaption></figure>

Conforme orientado na descrição, podemos abrir o laboratório e fazer login acessando "My account" e informando as credenciais `wiener:peter`.

<figure><img src="../../.gitbook/assets/ctfwsaapi_testinglab_Idashboard.png" alt=""><figcaption><p>Laboratório I - Dashboard</p></figcaption></figure>

Após a autenticação, a aplicação apresenta um formulário em que é possível alterar o e-mail. Nesse caso, eu estava interceptando o tráfego com o Burp Suite e realizei uma troca de e-mail. Com isso, foi possível observar uma requisição com método PATCH ao endpoint `/api/user/wiener` passando um JSON com o parâmetro e-mail e o valor que informei no formulário.

<figure><img src="../../.gitbook/assets/ctfwsaapi_testinglab_Idelete_endpoint.png" alt=""><figcaption><p>Laboratório I - Removendo o usuário "carlos"</p></figcaption></figure>

Enviei essa requisição para o Burp Repeater, removi o corpo da requisição (nesse caso, os dados JSON), alterei o método para DELETE e substituí o `wiener` por `carlos`. Ao enviar a requisição modificada, a aplicação não realiza nenhum controle e permite a exclusão do usuário `carlos`, concluindo a resolução desse laboratório.
