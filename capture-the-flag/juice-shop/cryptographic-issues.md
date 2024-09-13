# Cryptographic Issues

## Weird Crypto

> Informe a loja sobre um algoritmo ou biblioteca que não deveria ser usado como estão usando.

Para esse desafio, se você já resolveu o de SQL injection, não tem segredo. Basta se recordar que quando inserimos apenas um apóstrofo no formulário de login, ele retorna um erro e a query comparando a senha com o que parece ser um [hash](https://www.kaspersky.com.br/blog/hash-o-que-sao-e-como-funcionam/2773/) MD5. Considerando que, por segurança, as senhas devem ser armazenadas criptografadas usando um hash e um salt, ela não está sendo armazenada da forma mais segura no banco de dados. Portanto, se irmos até a página de feedback e enviar um comentário dizendo que a aplicação não deveria armazenar as senhas usando `MD5` da forma que está, o desafio é resolvido.
