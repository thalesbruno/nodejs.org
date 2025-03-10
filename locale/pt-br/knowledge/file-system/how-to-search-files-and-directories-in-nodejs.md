---
title: Como eu procuro arquivos e diretórios?
date: '2011-08-26T10:08:50.000Z'
tags:
  - filesystem
difficulty: 1
layout: knowledge-post.hbs
---

Suponha que você queira listar todos os arquivos no seu diretório atual.  Uma abordagem é utilizar a função builtin `fs.readdir` [method](/how-do-i-read-files-in-node-js). Isto lhe retornará um array com todos os arquivos e diretórios do caminho especificado:

    fs = require('fs');

    fs.readdir(process.cwd(), function (err, files) {
      if (err) {
        console.log(err);
        return;
      }
      console.log(files);
    });


Infelizmente, se você quuiser fazer uma lista recursiva dos arquivos, então as coisas rapidamente ficarão muito mais complicadas. Para evitar toda essa complexidade assustadora, esta é uma das circustâncias onde uma biblioteca feita por usuários salvar o dia. [Node-findit](https://github.com/substack/node-findit), pela SubStack, é um módulo auxiliar para facilitar a busca por arquivos.  Ele tem interfaces para permitir você trabalhar com callbacks, eventos, ou simplesmente da velha forma síncrona (o que na maioria das vezes não é uma boa ideia).

Para instalar `node-findit`, simplemente use npm:

    npm install findit

Na mesma pasta, crie um arquivo chamado `example.js`, e então adicione este código.  Execute-o com `node example.js`.  Este exemplo usa o `node-findit` com a interface baseada em eventos.

    //Isto instancia o localizador de arquivos
    var finder = require('findit').find(__dirname);

    //Isto fica escutando por diretórios encontrados
    finder.on('directory', function (dir) {
      console.log('Directory: ' + dir + '/');
    });

    //Isto fica escutando por arquivos encontrados
    finder.on('file', function (file) {
      console.log('File: ' + file);
    });
