---
layout: post
lang: ru
ref: athena_web_page_to_pdf
title: "Athena: Точный экспорт веб-страниц в PDF"
comments: true
tags: [pdf, web, docker, athena]
redirect_from: "/posts/ru/athena_web_page_to_pdf/"
---

![](/images/athena.png){:.post-title}

Мне потребовалось выдать заказчику нашу документацию по API, сгенерированную из
[Open API](http://swagger.io/).

Когда я попробовал экспортировать сгенерированные в 
[bootprint-openapi](https://github.com/bootprint/bootprint-openapi) страницы с помощью 
[pandoc](http://pandoc.org/)
или через импорт в MS Office или Libre Office, на результат нельзя было
смотреть без слез - поехавшие или вообще исчезнувшие таблицы.
Возможно, это было из-за бага в [bootprint-openapi](https://github.com/bootprint/bootprint-openapi)
который я позднее нашел и поправил, теперь уже не выяснить.

Ковыряться в [bootprint-openapi](https://github.com/bootprint/bootprint-openapi),
чтобы попытаться создавать такой html, который будет конвертироваться без глюков,
казалось совершенно бесперспективным и тяжким по времени занятием
(я [форкнул bootprint-openapi](https://github.com/andgineer/bootprint-openapi)
для русификации, но одно дело перевести на русский handle bars, что заняло не
более получаса, и другое - ковыряться в html и css).

И тут я нашел совершенно изумительную вещь
[Athena Elegant PDF conversion](http://www.athenapdf.com/).

Изумительна она по двум причинам.

Во-1х она безукоризненно конвертирует веб-страницы в PDF.
Все выглядит точно, как в веб-браузере, что и не удивительно, потому что она
вызывает браузер и снимает картинку с него.

Во-2х ребята все сделали для того, чтобы их продукт было чрезвычайно легко использовать.

Хотите сервис, который конвертирует на лету - ок:
{% highlight bash %}
docker pull arachnysdocker/athenapdf-service
http://<docker-address>:8080/convert?auth=arachnys-weaver&url=http://blog.arachnys.com/
{% endhighlight %}

Мне было рациональнее конвертировать файлы заранее (athena работает достаточно быстро, но это
все-таки долгие 1-2 секунды на конвертацию каждого из моих файлов).

Поэтому я добавил в Makefile:
{% highlight bash %}
for API in docker/docs/target/*/; \
    do docker run --rm -v ${PWD}:/converted/ arachnysdocker/athenapdf athenapdf $${API}/index.html $${API}/$$(basename $${API}).pdf; \
done
{% endhighlight %}

Немного огорчает, что из нее сыпятся ошибки
{% highlight bash %}
Xlib:  extension "RANDR" missing on display ":99".
{% endhighlight %}

Но это никак не влияет на результат, поэтому я не стал разбираться, с чем это связано.

И еще надо учитывать вот этот баг в Electron, на котором написана Athena
[Duplication of thead, and other table related issues](https://github.com/arachnys/athenapdf/issues/68)

Я это делаю так:
{% highlight bash %}
sed -i "\$athead {display:table-row-group;}" /docs/$API/main.css
{% endhighlight %}

Не устаю восхищаться docker, который сделал это возможным - одна строка (две, с учетом
workaround для бага) и непростая задача решена!
