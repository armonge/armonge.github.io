---
layout: post
title:  "Configurando django, gunicorn y supervisord en webfaction"
date:   2012-06-11 21:42:29
tags: python django webfaction
redirect_from: "/blog/1/django-gunicorn-y-supervisord-en-webfaction/"
---
En este mi primer blog hablaré de como configurar django usando gunicorn y supervisord en webfaction. Antes que nada quiero decir que webfaction es de los mejores hosting que he probado, sin embargo, ultimamente he estado probando gunicorn y debo decir que lo siento mucho más comodo que apache.

En este tutorial asumo que tenemos una cuenta de webfaction nueva y con el home limpio. Además usaremos el siguiente proyecto de Django como ejemplo [https://bitbucket.org/armonge/webfactiontut](https://bitbucket.org/armonge/webfactiontut "https://bitbucket.org/armonge/webfactiontut").

**Paso 1: Instalar pip y virtualenv**

{% highlight bash %}
mkdir -p ~/lib/python2.7
easy_install-2.7 pip
pip install virtualenv
{% endhighlight %}

**Paso 2: Instalar Django**

Para instalar nuestra aplicación Django vamos al [panel de administración de webfaction](http://my.webfaction.com "") y creamos 3 aplicaciones nuevas

- webfactiontut: Esta es una aplicación del tipo "Custom app(listening on port)" y sera donde instalemos nuestra aplicación Django
- webfactiontut_static: Esta aplicación es del tipo "Static only(no .htaccess)" y servira para nuestros archivos estáticos
- webfactiontut_media: Esta es del tipo "Static only(no .htaccess)" y servira para nuestros archivos de media

También es necesario crear una base de datos para el proyecto

Teniendo todo lo anterior nos dirigimos a /home/user/webapps/webfactiontut y ejecutamos

{% highlight bash %}
cd /home/user/webapps/webfactiontut
git clone https://bitbucket.org/armonge/webfactiontut.git webfactiontut
virtualenv --distribute --system-site-packages env
source env/bin/activate
pip install -r webfactiontut/requirements.txt
{% endhighlight %}

Con esto listo tenemos que cambiar lo siguiente en nuestra configuración de Django

{% highlight python %}
STATIC_ROOT = '/home/user/webapps/webfactiontut_static'
MEDIA_ROOT = '/home/user/webapps/webfactiontut_media'
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'user_webfactiontut',                      # Or path to database file if using sqlite3.
        'USER': 'user_webfactiontut',                      # Not used with sqlite3.
        'PASSWORD': 'SOME VERY DIFFICULT PASSWORD',                  # Not used with sqlite3.
        'HOST': '',                      # Set to empty string for localhost. Not used with sqlite3.
        'PORT': '',                      # Set to empty string for default. Not used with sqlite3.
    }
}
{% endhighlight %}

Y por último teniendo ya configurado el proyecto

{% highlight bash %}
python manage.py syncdb
python manage.py collectstatic
{% endhighlight %}

Y por último teniendo ya configurado el proyecto

{% highlight bash %}
python manage.py syncdb
python manage.py collectstatic
{% endhighlight %}

**Paso 3: Configurando supervisor**

[Supervisord](http://supervisord.org/ "") es un proceso que prefiero instalar a nivel global y no en el virtualenv, esto por que normalmente sólo tendremos un proceso de supervisor para muchas aplicaciones, dicho esto:

{% highlight bash %}
deactivate
cd 
pip install supervisor
cp lib/python2.7/supervisor/skel/sample.conf supervisord.conf
{% endhighlight %}

Con lo anterior tenemos supervisord instalado y un archivo de configuración que sólo falta personalizar, lo primero es cambiar todas las rutas para que apunten a direcciones dentro de nuestro home.

{% highlight ini %}
[unix_http_server]
file=/home/user/supervisor.sock   ; (the path to the socket file)

[supervisord]
logfile=/home/user/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/home/user/supervisord.pid ; (supervisord pidfile;default supervisord.pid)

[supervisorctl]
serverurl=unix:///home/user/supervisor.sock ; use a unix:// URL  for a unix socket
{% endhighlight %}

Y por último añadimos una sección [program:x] para nuestra aplicación, acá tenemos que cambiar 8000 por el puerto que el panel de webfaction nos indique.

{% highlight ini %}
[program:webfactiontut]
command=/home/user/webapps/webfactiontut/env/bin/python manage.py run_gunicorn -b :8000 --name=%(program_name)s --log-level=DEBUG
directory=/home/user/webapps/webfactiontut/webfactiontut/
autostart=true
autorestart=true
stdout-logfile=/home/user/logs/user/webfactiontut.log
stderr_logfile=/home/user/logs/user/webfactiontut.log
{% endhighlight %}

Si todo esta configurado correctamente entonces podemos echar a andar nuestra aplicación de la siguiente manera

{% highlight bash %}
supervisord -c supervisord.conf
{% endhighlight %}

Y podemos chequear el estado de nuestra aplicación de la siguiente manera
<s>
{% highlight bash %}
supervisorctl webfactiontut status
{% endhighlight %}

</s>

{% highlight bash %}
supervisorctl status webfactiontut
{% endhighlight %}

**Paso 4: Configurando el website**

Ahora que ya tenemos configuradas las aplicaciones sólo falta configurar el website en el panel de webfaction.

Para esto creamos un nuevo website de nombre webfactiontut y le añadimos las 3 aplicaciones que creamos de la siguiente manera

- webfactiontut en /
- webfactiontut_static en /static
- webfactiontut_media en /media


Y con esto se concluye el tutorial
