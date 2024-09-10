# Unvalidated Redirects

## Outdated Allowlist

> Faça-nos te redirecionar para um de nossos endereços de criptomoedas que não têm mais suporte.

Esse desafio envolve análise estática de código. Para isso, podemos abrir o código-fonte do arquivo `main.js` e procurar por `payment` em busca de como a aplicação lida com os métodos de pagamento. Navegando pelos resultados, é possível encontrar alguns funções com nomes de criptomoedas.

{% code title="main.js" overflow="wrap" lineNumbers="true" %}
```javascript
showBitcoinQrCode() {
  this.dialog.open(
    le, {
      data: {
        data: 'bitcoin:1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm',
        url: './redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm',
        address: '1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm',
        title: 'TITLE_BITCOIN_ADDRESS'
      }
    }
  )
}
showDashQrCode() {
  this.dialog.open(
    le, {
      data: {
        data: 'dash:Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW',
        url: './redirect?to=https://explorer.dash.org/address/Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW',
        address: 'Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW',
        title: 'TITLE_DASH_ADDRESS'
      }
    }
  )
}
showEtherQrCode() {
  this.dialog.open(
    le, {
      data: {
        data: '0x0f933ab9fCAAA782D0279C300D73750e1311EAE6',
        url: './redirect?to=https://etherscan.io/address/0x0f933ab9fcaaa782d0279c300d73750e1311eae6',
        address: '0x0f933ab9fCAAA782D0279C300D73750e1311EAE6',
        title: 'TITLE_ETHER_ADDRESS'
      }
    }
  )
}
```
{% endcode %}

Dentro dessas funções, é chamada uma função `this.dialog.open`, que recebem 2 parâmetros e um deles é um objeto. Perceba que dentro desse objeto é definido um objeto `data` em que é informada uma URL[^1] dentre seus atributos. Essa URL indica um redirecionamento para realizar o pagamento com uma carteira de criptomoedas. Se copiarmos essa URL e a informarmos no endereço raiz da aplicação, somos redirecionados e o desafio é resolvido.

[^1]: Uniform Resource Locator
