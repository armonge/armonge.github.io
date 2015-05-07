---
layout: post
title:  "Generador de codigos QR con asyncio"
date:   2014-07-16 16:28:32
tags: asyncio python
redirect-from: "/blog/2/generador-qr-asyncio-python/"
---
Ultimamente he estado interesado en aprender [asynncio](https://docs.python.org/3/library/asyncio.html "asyncio") y luego de ver un peguesito en odesk  donde pedían  crear un generador de codigos QR me dije que seria interesante hacer un servidor de codigos QR.

Aca mi implementación:

<script src="https://gist.github.com/armonge/d6d28ad99a00e2149181.js"></script>

Para correrlo solamente es necesario instalar las librerias en el requirements.txt y luego


{% highlight bash %}
$ python3 qrio.py
{% endhighlight %}

Asumiendo que tengamos el programa [httpie](https://github.com/jakubroztocil/httpie "httpie") instalado lo podemos probar de esta forma

{% highlight bash %}
$ http http://127.0.0.1:8000/  data=test  -d
HTTP/1.1 200 OK
CONNECTION: keep-alive
CONTENT-DISPOSITION: inline; filename="qr.png"
CONTENT-TYPE: img/png
DATE: Wed, 16 Jul 2014 22:36:28 GMT
SERVER: Python/3.4 aiohttp/0.8.2
TRANSFER-ENCODING: chunked

Downloading to "qr.png"
Done. 446.00 B in 0.01467s (29.70 kB/s)
{% endhighlight %}

Y el resultado es: ![test](https://drive.google.com/uc?export=download&id=0B92CMs3kfvLAV2JmTUF3RlBRTGc "Test QR Code")
