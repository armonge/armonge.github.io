---
layout: post
title:  "Retry Decorator"
date:   2014-08-13 08:29:24
tags: python decorator
redirect_from: "/blog/3/retry-decorator/"
---
Recientemente tuve un problema con un webapi, resulta que estaba llegando a los limites de uso del API y muchas peticiones estaban fallando. Buscando una manera sencilla de reintentar las tareas se me ocurrio crear un decorador y aca esta el resultado.

<script src="https://gist.github.com/armonge/b1b9b54a7add1c3fbd24.js"></script>

La idea es usarlo de la siguiente manera

{% highlight python %}
@retry(errors.HttpError)
def execute_request(request):
    return request.execute()
{% endhighlight %}

El decorador va a capturar las excepciones que reciba como parametro y reintentar la función cada cierto tiempo hasta llegar a un maximo. Probablemente hay mucho que mejorar, quizas definir la cantidad de intentos con un parametro, debería de usar [functools.wraps](https://docs.python.org/2/library/functools.html#functools.wraps "functools.wraps") o incluso otras cosas.

Pero bueno eso queda de ejercicio al lector
