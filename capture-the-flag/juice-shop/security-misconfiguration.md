# Security Misconfiguration

## Error handling

> Ocasione em um erro que não é apresentável e nem tratado com consistência.

Esse desafio pode ser solucionado de diversas formas, e muito provavelmente será solucionado enquanto você navega e explora a aplicação.

## Deprecated Interface

> Utilize uma interface B2B obsoleta que não foi encerrada corretamente.

Para esse desafio, precisamos nos aproveitar de uma funcionalidade da aplicação que foi descontinuada. Analisando o código fonte, me deparei com o seguinte trecho de código:

{% code title="main.js" overflow="wrap" lineNumbers="true" %}
```javascript
[
    'ng2FileSelect',
    '',
    'id',
    'file',
    'type',
    'file',
    'accept',
    '.pdf,.zip',
    'aria-label',
    // Observa a linha abaixo
    'Input area for uploading a single invoice PDF or XML B2B order file or a ZIP archive containing multiple invoices or orders<!---->',
    2,
    'margin-left',
    '10px',
    3,
    'uploader'
],
```
{% endcode %}

Acima são descritas as propriedades de um campo de _upload_ de arquivos. Na linha destacada, é citado a possibilidade de enviar um arquivo [XML](https://www.freecodecamp.org/portuguese/news/o-que-e-um-arquivo-xml-como-abrir-arquivos-xml-e-quais-sao-os-melhores-editores-de-xml/) contendo os pedidos de uma empresa. Se já tivermos navegado pelas funcionalidades da aplicação, tivemos contato com um campo de _upload_ no formulário de envio de reclamações. Portanto, podemos navegar até lá e tentar fazer o _upload_ de um arquivo XML. Ao enviar o formulário com o arquivo XML, o desafio é resolvido.
