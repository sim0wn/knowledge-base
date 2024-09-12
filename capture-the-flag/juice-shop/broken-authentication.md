# Broken Authentication

## Password Strength

> Faça login com as credenciais do usuário administrador sem antes alterá-las ou utilizando injeção de SQL.

Considerando que já tenha acessado o usuário administrador [anteriormente](https://logs.sim0wn.com.br/capture-the-flag/juice-shop/injection#login-admin) usando SQL injection, sabemos que o e-mail do usuário administrador é `admin@juice-sh.op`. Portanto, podemos ir até o formulário de login e tentar adivinhar a senha dele, supondo que seja uma senha fraca. Depois de algumas tentativas, consegui acessar a conta dele com a senha `admin123`.
