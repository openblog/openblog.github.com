---
title: Web Messaging API
layout: post
author: Vagner Santana
author_link: http://twitter.com/vagnervjs
author_profile: http://vagnersantana.com
image: img/posts/2013-02-25-message.jpg
tags: HTML5 javascript api
comments: true
keywords: >
  html5, javascript, messaging, api
resumo: >
  Já se deparou com a limitação de querer enviar dados de sua página para um `iframe` de um domínio diferente e não poder? Com essa <abbr title="Application Program Interface">API</abbr> isso se torna simples e fácil.
---

Provavelmente em algum projeto você já foi obrigado a utilizar um `iframe` para carregar conteúdo de uma página externa, e desde então só era possível visualizar o conteúdo do mesmo, e ele funciona como uma <i>sandbox</i> onde não existe comunicação com sua aplicação.
Nas especificações do W3C o <abbr title="HyperText Markup Language 5">HTML5</abbr> possui algumas <abbr title="Application Program Interface">APIs</abbr> para comunicação. Neste post trata especificamente uma delas: a HTML5 Web Messaging API que resolve o problema descrito.

## API

Essa é uma <abbr title="Application Program Interface">API</abbr> simples, que resolve o problema de comunicação entre aplicações hospedadas em diferentes origens.
Antes da existência dessa API, não era possível essa comunicação devido a "política da mesma origem", onde não é possivel uma programação client-side de uma origem acesse ou interfira em um documento de outra origem. Resumindo: um script hopedado em um lugar não consegue acessar o DOM de um documento de outra origem. 
A origem pode ser 3 tipos: protocolo, host e porta.
<br><br>
Exemplo: digamos que a origem de minha aplicação seja o URL: http://openblog.com.br/mensagem.php

<div class="tbl">
  <table>
    <tr>
      <th>URL</th>
      <th>Mesma origem?</th>
      <th>Razão</th>
    </tr>
    <tr>
      <td>http://openblog.com.br/recebe.php</td>
      <td>Sim</td>
      <td>-</td>
    </tr>
    <tr>
      <td>http://openblog.com.br</td>
      <td>Sim</td>
      <td>-</td>
    </tr>
    <tr>
      <td>https://openblog.com.br/auth.php</td>
      <td>Não</td>
      <td>Protocolo diferente</td>
    </tr>
    <tr>
      <td>http://openblog.com.br:9000/recebe.php</td>
      <td>Não</td>
      <td>Porta Diferente</td>
    </tr>
    <tr>
      <td>http://blog.openblog.com.br/recebe.php</td>
      <td>Não</td>
      <td>Host Diferente</td>
    </tr>
  </table>
</div>

##Método window.postMessage()

Este método é o responsável por criar uma mensagem a ser enviada para um objeto `window` em uma origem diferente. O padrão do tipo da mensagem é uma <i>string</i>.
O método recebe três parâmetros: dois obrigatórios e um opcional. 

{% highlight javascript %}
  window.postMessage(mensagem, destino, [portas])  
  //mensagem: string a ser enviada
  //destino: endereço a ser enviado
  //portas: array de portas válidas para o destino
{% endhighlight %}

O destino pode ser um URL absoluto, um caractere curinga (*) que servirá para qualquer destino ou um caractere barra (/) que adota a política da mesma origem (ou seja, apenas o mesmo host da página).

##Evento message

A <abbr title="Application Program Interface">API</abbr> prevê o evento `message` que é disparado no documento destino da mensagem. Quando disparado ele chama a seguinte função callback:

{% highlight javascript %}
  if (window.addEventListener) {
      window.addEventListener('message', receberMsg, false);
  } else {
      window.attachEvent("onmessage", receberMsg);
  };

  function receberMsg(e){
      //faça algo com a mensagem
  }
{% endhighlight %}

A função `receberMsg` como o nome já diz, recebe um parâmetro que retorna um objeto-evento com as propriedades:

{% highlight javascript %}
  e.data //text da mensagem
  e.origin //origem da mensagem
  e.lastEventId //string identificadora do último evento
  e.source //retorna WindowProxy do destino
  e.ports //array das portas enviadas
{% endhighlight %}

##Mãos na massa

Vamos criar a situação mencionada no início do post, onde um documento de origem A tenta se comunicar com um documento de origem B.
A idéia é postar uma mensagem e ela aparecer no `iframe`. Vamos lá...

- Documento A (http://openblog.github.com/demos/html5_msg_origin)
{% highlight javascript %}
  //Javascript
  <script>
    window.onload = function(){
      var objIframe = document.getElementsByTagName('iframe')[0];
      var btnEnviar =  document.getElementsByTagName('button')[0];

      btnEnviar.onclick = function(){
        var textoMsg = document.getElementsByTagName('input')[0].value;

        if(textoMsg == ''){
          alert('Digite uma mensagem!');
        } else{
          objIframe.contentWindow.postMessage(textoMsg, 'http://labs.vagnersantana.com');
        }
      }
    }
  </script>
{% endhighlight %}

{% highlight html %}
  <!-- HTML -->
  <section>
    <p>
      <label>Mensagem: <input type="text"></label>
      <button type="button">Enviar mensagem</button>
    </p>
    <iframe src="http://labs.vagnersantana.com/html5_msg_iframe.html"></iframe>
  </section>
{% endhighlight %}

- Documento B (http://labs.vagnersantana.com/html5_msg_iframe.html)
{% highlight javascript %}
  //Javascript 
  <script>
    if(window.addEventListener){
      window.addEventListener('message', receberMsg, false);
    } else{
      window.attachEvent("onmessage", receberMsg);
    };

    function receberMsg(e){
      var msg;
      var containerMsg = document.getElementById('recebe-msg');

      if(e.origin == 'http://openblog.github.com'){
        msg = 'Mensagem recebida: <br>';
        msg += 'Msg: ' + e.data + '<br>';
        msg += 'Origem: ' + e.origin;

        containerMsg.innerHTML = msg;
      } else{
        containerMsg.innerHTML = 'Origem não autorizada!';
      }
    }
  </script>
{% endhighlight %}

{% highlight html %}
  //HTML
  <p id="recebe-msg"></p>
{% endhighlight %}

O Web Messaging envia uma string, porém não precisa se desesperar e pensar em ir separando os dados por `|` ou `;` pois felizmente existe o maravilhoso `JSON` que oferece a funcionalidade de se transformar em string com o método `JSON.stringify` e recuperá-lo usando o `JSON.parse()`, desta forma podemos trocar e manipular dados de maneira fácil somando o Web Messaging com JSON.

##Can i use ?

<a href="http://caniuse.com/#search=messaging" target="_blank" alt="Can i use: Web Messaging API" title="Can i use: Web Messaging API">
  <img src="/img/posts/ciu_messaging.png" alt="Can i use table of Web Messaging API">
</a>

Para ver a <abbr title="Application Program Interface">API</abbr> em ação é só clicar no link abaixo, sinta-se à vontade em inspecionar o código e divirta-se.

<a href="/demos/html5_msg_origin/" alt="Demo" title="Demo" target="_blank">
  <button class="btn btn-demo">DEMO</button>
</a>

<h3>Referência</h3>
  <ul>
    <li>→<a href="http://livrohtml5.com.br/" alt="Livro HTML5 do Maujor" title="Livro HTML5 do Maujor">Livro HTML5 do Maujor, Capítulo 9</a></li>
    <li>→<a href="http://www.w3.org/TR/2010/WD-webmessaging-20101118/" alt="W3C Working Draft: HTML5 Web Messaging" title="W3C Working Draft: HTML5 Web Messaging">W3C Working Draft: HTML5 Web Messaging</a></li>
    <li>→<a href="https://github.com/DamonOehlman/talk-html5-messaging" alt="Damon Oehlman: Talk HTML5 Messaging" title="Damon Oehlman: Talk HTML5 Messaging">Damon Oehlman: Talk HTML5 Messaging</a></li>
  </ul>