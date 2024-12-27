# Photon Lockdown

> Encontramos a localização do adversário e agora devemos acessar o Terminal de Rede Óptico deles para desabilitar a conexão à Internet. Felizmente, obtivemos uma cópia do firmware do dispositivo, o qual suspeitamos conter as credenciais embutidas em algum arquivo. Você é capaz de extrair a senha dele?

Podemos baixar os arquivos do desafio e extrair. Com isso, obtemos três arquivos:

* fwu\_ver
* hw\_ver
* rootfs

Os primeiros dois arquivos listados são arquivos de texto e aparentam conter, respectivamente, a versão do firmware e a versão do hardware. Já o `rootfs` é um arquivo comprimido com [Squashfs](https://www.kernel.org/doc/html/latest/filesystems/squashfs.html), um sistema de arquivos comumente utilizado em sistemas embutidos para transferir o sistema através de um único arquivo comprimido. É possível descompactar esse arquivo utilizando o [unsquashfs](https://man.archlinux.org/man/extra/squashfs-tools/unsquashfs.1.en) da seguinte forma:

<pre class="language-bash"><code class="lang-bash"><strong>mkdir root
</strong><strong>unsquashfs -d ./root rootfs
</strong></code></pre>

Nos comandos acima, primeiro criamos um diretório root e então especificamos que o conteúdo do arquivo `rootfs` deve ser extraído no diretório criado. A partir disso, temos acesso a todo o sistema de arquivos compactado, e agora basta uma busca minuciosa para encontrar a senha. Se você tem conhecimento do sistema de arquivos Linux, vai encontrar rapidamente numa busca manual pelos arquivos de configuração no diretório `/etc`. Mesmo assim, é possível automatizar esse processo usando um grep no diretório extraído:

```bash
grep -ir "password\|HTB" . 2>/dev/null
```

No comando acima, a opção `i` indica que a busca deve ser case-insensitive (isto é, não diferencia maiúsculas das minúsculas) e a opção `r` indica que o grep deve realizar uma busca recursiva (ou seja, pesquisar em todos os arquivos presentes nos subdiretórios). Informamos o termo da busca usando regex com o `"password\|HTB"` para indicar que devem ser retornados os resultados que contenham `password` ou `HTB` (que é o padrão das flags do Hack The Box). Por fim, é realizado um redirecionamento com `2>/dev/null` para impedir que possíveis erros sejam impressos no terminal, e com isso podemos encontrar a senha/flag.
