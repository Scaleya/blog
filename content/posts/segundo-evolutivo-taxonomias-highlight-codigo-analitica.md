+++
draft = true
date = "2019-10-18T12:33:32+02:00"
publishdate = "2019-10-18T12:33:32+02:00"

title = "#2 Taxonomía de tags, highlighting de código y analítica"

description = "Seguimos adelante, esta vez avanzamos un poco más con el interlinking creando una taxonomías de tags, además de server-side code highlighting y montamos todo lo necesario para recaudar los datos y analizarlos con Cloudflare, Google Analytics y Search Console."

summary = "Seguimos adelante, esta vez avanzamos un poco más con el interlinking creando una taxonomías de tags, además de server-side code highlighting y montamos todo lo necesario para recaudar los datos y analizarlos con Cloudflare, Google Analytics y Search Console."

tags = ['Evolutivo']

keywords = ['blog', 'desarrollo', 'tags', 'syntax highlighting', 'google analytics', 'syntax highlighting', 'search console']

[amp]
    elements = []

[author]
    name = "Asur"
    image = ""
    bio = ""
    homepage = ""

[image]
    src = ""
    title = ""
    author = ""
    link = ""
    license = ""
    licenseLink = ""

[ogp]
    title = ""
    url = ""
    description = ""
    image = ""
    site = ""

[twitter]
    title = ""
    url = ""
    description = ""
    image = ""
    site = ""

[sitemap]
  changefreq = "monthly"
  priority = 0.5
  filename = "sitemap.xml"

+++

# Taxonomía de tags, highlighting de código y analítica

Seguimos adelante, esta vez avanzamos un poco más con el interlinking creando una taxonomías de tags, además de server-side code highlighting y montamos todo lo necesario para recaudar los datos y analizarlos con Cloudflare, Google Analytics y Search Console.

## Highlighting de código ✨

Desde un principio esta funcionalidad ha estado en el punto de mira, me encanta escribir snippets en mis posts, porque *talk is cheap, show me the code* es un mantra con el que me identifico y me gusta ir de *hardcore programmer*.

Me gusta mucho como está implementado esto en Hugo, se puede considerar *server-side highlighting*, ya que a la hora de compilar tu contenido a HTML se utiliza [Chroma](https://github.com/alecthomas/chroma), una librería escrita en Go que analiza y aplica markup a tus snippets y luego a través de CSS puedes darles estilos.

En mi opinión esto es un acercamiento mucho más óptimo que al highlight en el cliente, como el de [highlight.js](https://highlightjs.org/) ya que además de no ocupar tiempo en el hilo de javascript analizando texto al vuelo, no habrá ningún parpadeo en el que el código salga sin estilos.

Para resaltar un trozo de código a la hora de escribir contenido tengo que usar un shortcode nativo de Hugo:

```
{{</* highlight php "linenos=table" */>}}

<?php

// ... code

function foo() {
    $bar = 'string';
}

{{</* /highlight */>}}
```

Que se verá de la siguiente manera:

{{< highlight php "linenos=table" >}}

<?php

// ... code

function foo() {
    $bar = 'string';
}

{{< / highlight >}}

Uno de mis objetivos era que se pudiese copiar y pegar el código resaltado sin incluir los números de línea, algo que pasa muy a menudo en muchas webs y me saca de quicio. Para hacerlo posible hay que incluir el atributo `"linenos=table"` al shortcode, de esta forma se utilizarán tablas HTML al maquetar la estructura de código y sus líneas independientemente, en vez de solo insertar un número al principio de estas últimas.

Como detalle final, la librería de Chroma es compatible con los temas de Pygments, otra librería similar pero escrita en Python, y esto abre un abanico enorme de [combinaciones de colores](https://xyproto.github.io/splash/docs/). Yo me he decantado por el tema *Github*, como no... Para aplicarlo tenemos que habilitar en la configuración de nuestro sitio `pygmentsUseClasses = true` e incluir el CSS generado por el comando `hugo gen chromastyles --style=github > syntax.css` a nuestro fichero de estilos.

## Taxonomía de tags 🏷️

Las taxonomías son maneras de agrupar elementos por características comunes. La primera implementación que voy a hacer al respecto es la de tags. Es una estrategia muy común a la hora de crear una estructura de links para SEO, también ayudará por supuesto a los usuarios a encontrar contenido que les interese.

Hugo ya incluye por defecto dos taxonomías: tags y categorías, además de poder crear las tuyas propias. He empezado por tags por mera comodidad, creo que las categorías las dejaré para cuando implemente unas migas de pan y lo haré todo junto.

La implementación, al estar ya soportado, no tiene mucha miga, para empezar añadimos las tags en la metadata del post de turno:

```
tags = ['Evolutivo', 'Data', 'Hugo']
```

Una vez tenemos los datos, pasan a formar parte del objeto del post y los podemos listar, como se puede ver encima del título de este post:

{{< highlight go-html-template "linenos=table" >}}

{{ $taxonomy := "tags" }}
<ul id="{{ $taxonomy }}">
    <li>Tags:</li>
    {{ range .Param $taxonomy }}
        {{ $name := . }}
        {{ with $.Site.GetPage (printf "/%s/%s" $taxonomy ($name | urlize)) }}
            <li><a href="{{ .Permalink }}">{{ $name }}</a></li>
        {{ end }}
    {{ end }}
</ul>

{{< / highlight >}}

Por último tenemos que crear una plantilla para las páginas de taxonomía. Es obligatorio que se llame `taxonomy.html` y es lo que se renderizará cuando se navegue al link que tiene cada tag. Ya tenemos accesibles todas las entradas pertenecientes a esa tag, por lo que el código se parece mucho al de la homepage, un título (que será el nombre de la tag) y un bucle para imprimir los links:

{{< highlight go-html-template "linenos=table" >}}

{{ define "main" }}
  <div>
    <h1>{{ .Title }}</h1>
    <ul>
        {{ range .Pages }}
            <li>
                <a href="{{ .Permalink }}">{{.PublishDate.Format "02-01-2006"}} | {{ .Title }}</a>
            </li>
        {{ end }}
    </ul>
  </div>
{{ end }}
{{< / highlight >}}

Finiquitado, ya tenemos una taxonomía básica montada.

## Analítica 📊

Soy un *nerd* para los números y las gráficas, no tengo ni idea de análisis ni representación de datos pero me encanta mirar gráficas, por lo que la parte de analítica iba a llegar más pronto que tarde. Por el momento he decidido utilizar tres fuentes de datos: Google Analytics, Search console y Cloudflare.

### Google Analytics

Google Analytics es una navaja suiza, con el conocimiento suficiente que espero ir adquiriendo con el tiempo puedes hacer prácticamente lo que quieras en lo que recolección de datos se refiere.

Para añadirlo en una página AMP tenemos que usar el componente `amp-analytics` de la siguiente manera:

{{< highlight html "linenos=table" >}}

<amp-analytics type="gtag" data-credentials="include">
  <script type="application/json">
    {
      "vars": {
        "gtag_id": "UA-128498798-1",
        "config": {
          "UA-128498798-1": { "groups": "default" }
        }
      },
      "triggers": {
        "trackPageview": {
          "on": "visible",
          "request": "pageview"
        }
      }
    }
  </script>
</amp-analytics>

{{< / highlight >}}

Los datos que más me interesan son:

 - Visitas
 - Adquisición
 - Tiempo en página de posts

**@TODO**

### Search console

Search console te muestra datos de la indexación de tu web en Google, así como las mejoras que ha detectado en tu web, como AMP, usabilidad móvil y datos estructurados.

El proceso de validación es a través de DNS, lo único que hay que añadir es un registro CNAME con el contenido que se nos indica al registrarnos y listo, ya tenemos acceso.

Las partes a las que prestaré más atención son:

 - Rendimiento
 - Cobertura
 - Mejoras

**@TODO**

### Cloudflare

Con Cloudflare no he tenido que hacer nada, ya estaba montado desde el principio como proxy de esta web, pero sigue teniendo una parte de analítica muy interesante, que está más orientada a los datos técnicos y son mucho más crudos que en el resto de plataformas ya que son recaudados a nivel DNS y proxy. 

Las partes más interesantes son:

 - Requests cacheados y no cacheados
 - Performance pies

**@TODO**

## Siguientes pasos 👣

Para el siguiente evolutivo ya tengo bastante claro en que va a consistir, ya he empezado a trabajar (con mucha ayuda) en una mejora del workflow de desarrollo y redacción de contenido, vamos a crear un flujo con Docker y Makefiles, además también me voy a apuntar a la beta abierta de Github Actions y haremos algo de despliegue continuo. *Stay tuned!* 😎

## Wayback Machine ⏰

Ver la [versión original de este post](# "Versión original del post").

Ver la [versión original de la homepage](# "Versión original de la homepage").