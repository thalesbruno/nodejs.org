---
title: Como eu posso proteger meu código?
date: null
tags:
  - filesystem
  - security
difficulty: 3
layout: knowledge-post.hbs
---

Às vezes, você pode querer deixar usuários lerem ou escreverem arquivos em seu servidor. Por exemplo, talvez você queira escrever um software de fórum sem usar um banco de dados de verdade. O problema é que você não quer que seus usuários possam modificar ou ler todo e qualquer arquivo em seu servidor, e por vezes há formas de contornar restrições que deveriam impedí-los. Continue lendo para ver como você pode proteger seu código contra atacantes malvados tentando bagunçar seus arquivos.

Poison Null Bytes
=================
Poison null bytes é um modo de enganar o seu código para que ele veja outro nome de arquivo em vez do que realmente está sendo aberto. Isto pode, em muitos casos, ser usado para contornar proteções a directory traversal, para enganar servidores a entregar arquivos com tipos errados e para contornar restrições nos nomes de arquivos que podem ser usados. [Aqui está uma descrição mais detalhada.](http://groups.google.com/group/nodejs/browse_thread/thread/51f66075e249d767/85f647474b564fde) Sempre use código como este quando estiver acessando arquivos com nomes fornecidos pelo usuário:

    if (filename.indexOf('\0') !== -1) {
      return respond('That was evil.');
    }

Whitelisting
============
Você sem sempre será capaz de usar whitelisting, mas se você for, use-a - é muito fácil de implementar e difícil de dar errado. Por exemplo, se você souber que todos os nomes de arquivos são strings alfanuméricas em caixa baixa:

    if (!/^[a-z0-9]+$/.test(filename)) {
      return respond('illegal character');
    }

Contudo, note que whitelisting sozinha não será mais suficiente assim que  você permitir pontos e barras - pessoas poderiam entrar com coisas como `../../etc/passwd` a fim de obter arquivos de fora da pasta permitida.

Prevenindo Directory Traversal
==============================
Directory traversal significa que um atacante tenta acessar arquivos fora do diretório que você quer permitir que ele acesse. Você pode prevenir isto usando o módulo "path" nativo do nodes. **Não reimplementar as coisas no módulo path por conta própria** - por exemplo, quando alguém executa seu código em um servidor windows, não manipular barras invertidas como barras normais permitrá que atacantes façam directory traversal.

Este exemplo assume que você já checou a variável `userSuppliedFilename`  como descrito na seção acima "Poison Null Bytes".

    var rootDirectory = '/var/www/';

Garanta que você tem uma barra no final do nome dos diretórios permitidos - você não quer que pessoas sejam capazes de acessar `/var/www-secret/`, não é?.

    var path = require('path');
    var filename = path.join(rootDirectory, userSuppliedFilename);

Agora `filename` contém um caminho absoluto e não contém mais sequências `..` - `path.join` cuida disso. Entretanto, pode ser algo como `/etc/passwd` agora, aí você tem que checar se começa com `rootDirectory`:

    if (filename.indexOf(rootDirectory) !== 0) {
      return respond('trying to sneak out of the web root?');
    }

Agora a varável `filename` deve conter o nome do arquivo ou  diretório que está dentro do diretório permitido (a menos que ele não exista).
