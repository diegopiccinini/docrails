**NO LEAS ESTE FICHERO EN GITHUB, LOS MANUALES ESTÁN PUBLICADOS EN http://guides.rubyonrails.org.**

Comenzando con Rails
==========================

Esté manual cubre el comienzo y la ejecución de Ruby on Rails.

Después de leer este manual, tu conocerás:

* Como instalar Rails, crear una nueva aplicación Rails, y conectar tu aplicación a una base de datos.
* La disposición general de una aplicación Rails.
* Los principios básicos del patrón MVC (Modelo, Vista, Controlador) y el diseño RESTful.
* Como rapidamente generar los componentes iniciales de una aplicación Rails.

--------------------------------------------------------------------------------

Supuestos del Manual
--------------------

Este manual está diseñado para principiantes quienes quieren comenzar con Rails garabateando con el código. Esto no supone que tu tienes una experiencia previa con Rails.
Sin embargo, para sacar el mejor partido del aprendizaje, tendrás que contar con ciertos prerequisitos de instalación:

* El lenguaje [Ruby](https://www.ruby-lang.org/en/downloads) versión 1.9.3 o superior.
* El sistema de paquetes [RubyGems](https://rubygems.org), el cual se instala con Ruby versiones 1.9 o posteriores. Para aprender más acerca de RubyGems, por favor leer [Manuales de RubyGems](http://guides.rubygems.org).
* Una instalación funcionando de [SQLite3 Database](https://www.sqlite.org).

Rails es un framework para aplicaciones web, que corre sobre el lenguaje de programación Ruby.
Si no tienes experiencia previa con Ruby, podrás encontrar muy empinada la curva de aprendizaje.
Aquí hay un listado de muchos sitios donde puedes aprender en línea Ruby:

* [La página oficial del Lenguaje de programación Ruby](https://www.ruby-lang.org/en/documentation/)
* [reSRC's un listado de libros de programación gratuitos](http://resrc.io/list/10/list-of-free-programming-books/#ruby)

Tenga en cuenta que algunos recursos, mientras siguen siendo excelentes, cubren versiones de Ruby tan antiguas como la
1.6, y comunmente la 1.8, y podrían no incluir alguna de la sintaxis que verás en tu día a día de desarrollo en Rails.

¿Porqué Rails?
--------------


Rails es un framework de desarrollo de aplicaciones web escrito en el lenguaje Ruby.
Está diseñado para simplificar la programación de aplicaciones web, asumiendo que todos los desarrolladores necesitan empezar. Este diseño hace que se tenga que escribir menos código que en otros frameworks.
Experimentados desarrolladores Rails dicen que el desarrollo de la aplicaciones web se vuelve mucho más divertido.

Rails es un software obstinado. Esto hace suponer que existe una mejor manera de hacer las cosas, y está diseñado para alentar este camino -y en algunos casos desalentar alternativas. Si tu aprendes el "Camino Rails" probablemente descubrirás un tremendo incremento en la productividad. Si tu persistes en traer viejos hábitos de otros lenguajes a tu desarrollo Rails, tratando de utilizar patrones aprendidos en otros sitios, es posible que tengas una experiencia menos feliz.

La filosofía Rails incluye dos principios rectores:

* **No te repitas a ti mismo (Don't Repeat Yourself):** DRY es un principio del desarrollo de software el cual establece "Cada pieza del conocimiento debe ser única, sin ambigüedades, con su propia autoridad y representación dentro de un sistema". Al no escribir la misma información una y otra vez, nuestro código es más fácil de mantener, más reutilizable y con menos errores.
* **Convención sobre Configuración:** Rails tiene sus opiniones acerca de el mejor de hacer muchas cosas en las aplicaciones web, y por defecto las configura como convenciones, en lugar de requerir que se especifiquen minuciosamente a través de ficheros de configuración sin fin.

Creando un nuevo Projecto Rails
-------------------------------

El mejor camino para utilizar este manual es seguir cada paso como ocurre, ningún código o paso se dejará afuera para que puedas seguir literalmente paso por paso hasta el final.

Para seguir hasta el final este capítulo, crearás un proyecto Rails llamado `blog`, un (muy) sencillo webblog. Antes de que puedeas empezar a construir una aplicación, necesitas estar seguro de que tienes instalado rails.


TIP: El ejemplo de abajo utiliza `$` para representar tu terminal de linea do comandos o consola en un sistema operativo tipo UNIX, aunque puede ser modificado para tener una apariencia diferente. Si tu estás utilizando Windows, tu terminal de línea de comandos puede parecerce a esto `c:\source_code>`

### Instalando Rails

Abre una terminal de línea de comandos. En Mac OS X abre Terminal.app, en Windows elige
"Ejecutar" desde tu menú de Inicio y escribe 'cmd.exe'. Cualquier comando precedido de un signo `$` podría ser ejecutado en la línea de comandos. Verifica que tienes una versión actualizada de Ruby instalada:

TIP: Un numero de herramientas existen para ayudar a instalar rápido Ruby y Ruby on Rails, en tu sistema. Los usuarios de Windows users pueden utilizar [Instalador de Rails](http://railsinstaller.org),
mientras que los de Mac OS X pueden usar [Tokaido](https://github.com/tokaido/tokaidoapp).
Para más métodos de instalación para la mayoría de los sistemas operativos pueden mirar
[ruby-lang.org](https://www.ruby-lang.org/en/documentation/installation/).

```bash
$ ruby -v
ruby 2.0.0p353
```

En muchos sistemas operativos tipo UNIX encuentras una aceptable version de SQLite3.
En Windows, si tu estás instalando Rails a través del Instalador de Rails, ya tendrás SQLite instalado. Otros pueden encontrar instrucciones de instalación en la [Página de SQLite3](https://www.sqlite.org).
Verifica que está correctamente instalada en tu PATH:

```bash
$ sqlite3 --version
```

El programa mostrará su versión.

Para instalar Rails, utiliza el comando `gem install` proporcionado por RubyGems:

```bash
$ gem install rails
```

Para verificar que tienes todo instalado correctamente, deberías tener la posibilidad de ejecutar el siguiente comando:

```bash
$ rails --version
```
Si dice algo como "Rails 5.0.0", estás listo para continuar.

### Creando la Aplicación Blog

Rails viene con un numero de scripts llamados generadores (generators) que están diseñados para hacer tu vida de desarrollador más facil creando todo lo que necesitas para empezar a trabajar en una tarea particular. Una de esos es el generador de nuevas aplicaciones Rails, el cual provee lo necesario para crear una aplicación rails fresca, y que no tengas que hacerlo manualmente tu mismo.

Para utilizar este generardor, abre la terminal de línea de comandos, navega hasta el directorio donde tengas permisos para crear ficheros, y escribe:

```bash
$ rails new blog
```

Esto creará una aplicación Rails llamada Blog en un directorio `blog` e instalará las gemas de las que dependa enunmeradas en el `Gemfile` utilizando `bundle install`.

TIP: Puedes ver todas las opciones de los comandos que el constructor de aplicaciones Rails acepta ejecutando `rails new -h`.

Después de crear la aplicación blog, entra en el directorio:

```bash
$ cd blog
```

El directorio `blog`  tiene un numero de ficheros y carpetas auto-generados que constituyen la estructura de una aplicación Rails. La mayor parte del trabajo en este turorial se hará en la carpeta `app`, pero aqui esta la relación básica entre cada fichero/carpeta y la función que cumple:

| Fichero/Carpeta | Propósito |
| ----------- | ------- |
|app/|Contiene los controladores, modelos, vistas, funciones de ayuda, enviadores de correo, y ficheros assets fuente (sin compilar) para tu aplicación. Te enfocarás en esta carpeta en lo que queda de este capítulo.|
|bin/|Contiene los scripts rails para comenzar tu aplicación, y pude contener otros scripts utilizados para configurar, desplegar y ejecutar tu aplicación.|
|config/|Configurar tu apliación, rutas, base de datos, y más. Esto se cubre con mayor detalle en [Configurando Aplicaciones Rails](configuring.html).|
|config.ru|Configuración Rack para que los servidores basados en Rack puedan arrancar la aplicación.|
|db/|Contiene tu actual esquema de datos, como también las migraciones de la base de datos.|
|Gemfile<br>Gemfile.lock|Estos ficheros te permiten especificar las gemas que necesitas y la dependencias de tu aplicación Rails. Estos ficheros son utilizados por la gema Bundler. Para más información acerca de Bundler, ver [Página de Bundler](http://bundler.io).|
|lib/|Modulos extendidos para tu aplicación.|
|log/|Ficheros de log de la aplicación.|
|public/|La única carpeta que es vista por el mundo exterior tal cual es. Contiene los ficheros estaticos y los ficheros assets compilados.|
|Rakefile|Este fichero contiene y graba las tareas que pueden ser ejecutadas a través de una linea de comando. Las definiciones de tareas son utilizadas a lo largo de todos de los componentes de Rails. Mejor que cambiar el Rakefile, tu deberías crear tu propias tareas añadiendo ficheros en la carpeta lib/tasks de tu aplicación.|
|README.rdoc|Este es un breve manual de instrucciones de tu apliación. Tu puedes editar este fichero para decirle a otros que hace tu aplicación, como configurala, etc.|
|test/|Test unitarios, ficheros con datos de prueba y otros tipos de test. Esto está cubierto en [Probando Aplicaciones Rails](testing.html).|
|tmp/|Ficheros temporales (como los de cache e idenficadores de procesos pid).|
|vendor/|Un lugar por todo el código de terceros. En una aplicación rails típica esto incluye gemas vendidas.|

¡Hola, Rails!
-------------

Vamos a empezar, escribiendo un texto para verlo en la pantalla rápidamente. Necesitarás ejecutar tu servidor de aplicaciones rails.

### Levantando el Servidor Web

Realmente, ya tienes una aplicación Rails funcional. Para ver como funciona, solo necesitas levantar el servidor web en tu ordenador de desarrollo. Puedes hacer esto ejecutando lo siguiente en el directorio `blog`:

```bash
$ bin/rails server
```

TIP: Compilar CoffeeScript y JavaScript asset compriprimidos requiere que tengas sofware para ejectutar JavaScript disponible en tu sistema, en la ausencia de una verás un error `execjs` durante la compilation de los assets. Usualmente Mac OS X y Windows viene con su interprete para ejecución de JavaScript instalado. Rails añade la gema `therubyracer` al archivo `Gemfile` generado para las nuevas aplicaciones en una linea comentada, puedes descomentarla si la necesitas. `therubyrhino` es la gema recomendada para usuarios de JRuby y es añadida por defecto en el fichero `Gemfile` en aplicaciones generadas bajo JRuby. Puedes investigar todos los interpretes soportados en [ExecJS](https://github.com/sstephenson/execjs#readme).

Esto levantará WEBrick, un servidor web distribuido con ruby por defecto. Para ver tu aplicación en acción, abre una ventana de navegador en la siguiente dirección <http://localhost:3000>. Deberías ver la página de iformación por defecto de Rails:

![Bienvenido a bordo pantallazo](images/getting_started/rails_welcome.png)

TIP: Para parar el servidor web, teclea Ctrl+C en la terminal donde se está ejectuando el servidor. Para verificar que el servidor ha parado deberías ver el cursor activo en tu terminal otra vez. Para la mayoriade los sistemas tipo UNIX, incluyendo Mac OS X esto debería ser un signo `$`. En el modo de desarrollo, Rails generalmente no require que reinicies el servidor; los cambios que tu hagas en los ficheros ser trasmitirán automaticamente al servidor.

La página de  "Bienvenido a bordo" es una _prueba de humo_ para una nueva aplicación Rails: asegura que tienes configurado el software lo suficientemente bien para servir una página. También puedes hacer click sobre el enlace _About your application's environment_ (acerca el entorno de tu aplicación) para ver un resumen del entorno de tu aplicación.

### Decir "Hola", Rails

Para obtener que Rails diga "Hola", necesitas crear como mínimo un _controlador_ y una _vista_.

El propósito de un controlador es recibir una petición específica para la aplicación. _El sistema de enrutamiento_ , decide que controlador recibe cada petición. A menudo, hay mas de una ruta haca cada controlador, y las diferentes rutas pueden ser servidas por diferentes _acciones_. El propósito de cada acción es recoger information para proveersela a una vista.

El propósito de una vista es mostrar esa información en un formato humano legible. Una importante distinción para hacer es que es en el _controlador_, no en la vista, donde la información se recoge. La vista debe solo mostrar esa información. Por defecto, las vistas están escritas en un lenguaje llamado eRuby (Ruby Embebido), el cual es procesado en el ciclo de la petición en Rails antes de ser enviada al usuario.

Para crear un nuevo controlador, necesitaras ejectuar el generador "controller" y decirle que quieres un controlador llamado "welcome" con una acción llamada "index", de la siguiente manera:

```bash
$ bin/rails generate controller welcome index
```

Rails creará varios vicheros y rutas por ti.

```bash
create  app/controllers/welcome_controller.rb
 route  get 'welcome/index'
invoke  erb
create    app/views/welcome
create    app/views/welcome/index.html.erb
invoke  test_unit
create    test/controllers/welcome_controller_test.rb
invoke  helper
create    app/helpers/welcome_helper.rb
invoke  assets
invoke    coffee
create      app/assets/javascripts/welcome.coffee
invoke    scss
create      app/assets/stylesheets/welcome.scss
```

Lo más importante de esto es por supuesto el controlador, localizado en
`app/controllers/welcome_controller.rb` y la vista, localizada en
`app/views/welcome/index.html.erb`.

Abre el fichero `app/views/welcome/index.html.erb` con tu editor de texto. Borra todo el código existente en el fichero, y reemplazalo con la siguiente linea de código:

```html
<h1>¡Hola, Rails!</h1>
```

### Configurando la página de Inicio de la aplicación

Ahora que tenemos hecho el controlador y la vista, necesitamos decirle a Rails cuando queremos que muestre "¡Hola, Rails!". En nuestro caso queremos que lo haga cuando el navegante pida la raíz de nuestro sitio, es decir esta URL, <http://localhost:3000>. Hasta el momento, "Bienvenido a bordo" está ocupando ese lugar.

Para continuar, tienes que decirle a Rails donde tu página de inicio actual se localiza.

Abre el fichero `config/routes.rb` en tu editor.

```ruby
Rails.application.routes.draw do
  get 'welcome/index'

  # The priority is based upon order of creation:
  # first created -> highest priority.
  #
  # You can have the root of your site routed with "root"
  # root 'welcome#index'
  #
  # ...
```

Esta es el fichero de _enrutamiento_ de tu aplicación el cual mantiene las entradas en un lenguaje especial
[DSL (domain-specific language)](http://en.wikipedia.org/wiki/Domain-specific_language)
que dice a Rails como conectar las peticiones entrantes con los controladores y las acciones. Este fichero contiene muchas líneas comentadas que son rutas rutas de ejemplo, y una de ellas actualmente muestra como conectar a la raíz de tu sitio con una acción y controlador específicos. Encuentra la línea que comienza con `root` y descoméntala. Debería verse algo como esto:

```ruby
root 'welcome#index'
```

`root 'welcome#index'` le dice a Rails que derive las peticiones que llegan a la raíz de la aplicación hacia la acción index del controlador welcome y `get 'welcome/index'` le indica a Rails que las peticiones que llegan a <http://localhost:3000/welcome/index> se deriven a la acción index del controlador welcome también. Esto fue creado automáticamente cuando ejectutasteis por comando el controller generator (`rails generate controller welcome index`).

Levanta nuevamente el servidor web si lo habías parado para generar el controlador (`rails server`) y abre el navegador en  <http://localhost:3000>. Verás el mensaje "¡Hola, Rails!" que has escrito en el fichero `app/views/welcome/index.html.erb`, indicando que la nueva ruta en efecto va a la acción `index` del `WelcomeController` y muestra la vista correctamente.

TIP: Para más información acerca del enrutamiento, busca en [Enruamiento Rails Desde Fuera Hacia Dentro](routing.html).

Levantarse y Correr
-------------------

Ahora que has vistos como se crea un controlador, una acción y una vista, vamos a crear un poco más ambicioso.

En la aplicación Blog, crearás un nuevo _recurso_. Un recurso es el termino utilizado para una colección de objetos similares, como artículos, gente o animales.
Puedes crear, leer, actualizar y destruir elementos de un recurso, estas operaciones son conocidas como operaciones _CRUD_.

Rails provee un método para `recursos` que puedes utilizarlo para declarar un recurso REST. Necesitas añadir el _recurso article_ al fichero de configuración del enrutamiento `config/routes.rb` de la siguiente manera:

```ruby
Rails.application.routes.draw do

  resources :articles

  root 'welcome#index'
end
```

Si ejectutas `rake routes`, verás que como se han definido las rutas para todas las acciones RESTful estandard. El significado de la columna prefijada (y las otras columnas) lo veremos más adelante, pero ahora nota como Rails ha deducido la forma singular `article` y hace un uso significativo de esta.

```bash
$ bin/rake routes
      Prefix Verb   URI Pattern                  Controller#Action
    articles GET    /articles(.:format)          articles#index
             POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show
             PATCH  /articles/:id(.:format)      articles#update
             PUT    /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
        root GET    /                            welcome#index
```

En la próxima sección, añadiremos la capacidad de crear nuevos articulos en tu aplicación y será poblible verlos. Estas son la "C" y la "R" de "CRUD": crear y leer (creation and reading en inglés). La forma de hacer esto se ve así:


![El formulario de un nuevo artículo](images/getting_started/new_article.png)

Luce un poco básico por ahora, pero está bien. Nos ocuparemos de mejorar el estilo en cuanto podamos.

### Estableciendo la base del trabajo

Primero, necesitas un espacio dentro de la aplicación para crear un nuevo artículo. Un gran lugar podría ser `/articles/new`. Con la ruta ya definida, la petición ahora podría hacerse  `/articles/new` a la aplicación.
Si vas a <http://localhost:3000/articles/new> verás un error de enrutamiento:

![Otro error de enrutamiento, la constante ArticlesController no esta inicializada](images/getting_started/routing_error_no_controller.png)

Este error ocurre porque la ruta necesita tener un controlador definido para poder atender la petición. La solución a este particular problema es simple: crear un controlador llamado `ArticlesController`. Tu puedes hacerlo ejecutando el siguiente comando:

```bash
$ bin/rails g controller articles
```

Si abres el recientemente generado `app/controllers/articles_controller.rb` verás un controlador bastante vacio:

```ruby
class ArticlesController < ApplicationController
end
```

Un controlador es una simple case que es definida como hija de `ApplicationController`.
Es dentro de la clase donde definirás los metodos que se convertirán en acciones para este controlador. Estas acciones desempeñarán las operaciones CRUD sobre los artículos dentro de nuestro sistema.

NOTE: Hay `public`, `private` y `protected` metodos en Ruby, pero unicamente los métodos `public` pueden ser acciones para los controladores.
Para más detalles puedes investigar en [Programando en Ruby](http://www.ruby-doc.org/docs/ProgrammingRuby/).

Si recargas ahora <http://localhost:3000/articles/new>, obtendrás un nuevo error:

![¡La acción new es desconocida para ArticlesController!](images/getting_started/unknown_action_new_for_articles.png)

Este error indica que Rails no puede encontrar la acción `new` dentro del controlador `ArticlesController` que tu acabas de generar. Esto es porque cuando los controladores son generados por Rails están vacios por defecto, a menos que le manifiestes tu deseo en el proceso de generación.

Paradefinir manualemente una acción dento del controlador, todo lo que necesitas hacer es definir un método new dentro del controlador. Abre `app/controllers/articles_controller.rb` y dento de la clase `ArticlesController`, define el método `new` asi que tu controlador lucira ahora de esta manera:

```ruby
class ArticlesController < ApplicationController
  def new
  end
end
```

Con el metodo `new` definido en `ArticlesController`, si tu recargas
<http://localhost:3000/articles/new> verás otro error:

![Desaparecida la plantilla articles/new]
(images/getting_started/template_is_missing_articles_new.png)

Tu estás teniendo este error porque Rails espera que acciones llanas como estas tengan la correspondiente vista asociada para mostrar su información. Si no hay una vista disponible, Rails rails mostrara una excepción.

En la imagen de arriba, la linea de abajo está cortada. Vamos a ver el mensaje completo muestra lo siguiente:

>Desaparecida la plantilla articles/new, application/new con {locale:[:en], formats:[:html], handlers:[:erb, :builder, :coffee]}. Buscado en: * "/path/to/blog/app/views"

¡Esa es una gran cantidad de texto! Vamos rapidamente a examinarlo para entender que significa cada parte.

La primera parte identifica que plantilla esta perdida. En este caso, es la plantilla `articles/new`. Rails primero buscará está plantilla. Si no la encuentra, intentará cargar una plantilla llamada `application/new`. Busca también aqui porque `ArticlesController` es hijo de `ApplicationController`.

La siguiente parte del mensaje contiene un hash. La llave `:locale` en este hash simplemente indica en que idioma la plantilla será recuperada. Por defecto, es la plantilla inglesa - o "en" -. La siguiente clave, `:formats` especifica cual será el formato en que la plantilla será servida en la respuesta. El formato por defecto es `:html`, y así Rails buscara una plantilla html. La última llave, `:handlers`, está diciendonos que _manipulador de plantillas_ podrá ser uitlizado para mostrar nuestro template. `:erb` es el más comunmente utilizado para plantillas HTML, `:builder` es utilizado para plantillas XML, y
`:coffee` se utiliza CoffeeScript para construir plantillas JavaScript.

El mensaje final nos dice donde Rails ha buscado las plantillas. Plantillas dentro de una aplicación Rails básica como esta son mantenidas en un solo lugar, pero en aplicaciones más complejas pueden estar en lugares muy diferentes.

La más plantilla más simple que podría funcionar en este caso podría ser localizada en `app/views/articles/new.html.erb`. La extensipmes en el nombre del fichero en este caso son importantes: la primera extensión es el _formato_ de la plantilla, y la segunda extensión es la _sub-rutina_ que será utilizada. Rails esta intentando encontrar una plantilla llamada `articles/new` dentro de la vista `app/views` de la aplicación. El formato de esta plantilla solo puede ser `html` y la subrutina solo puede ser una `erb`, `builder` o `coffee`. Porque tu quieres crear un nuevo formulario HTML, deberás utilizar el lenguaje `ERB` que es diseñado para embeber Ruby en HTML.

Por lo tanto el fichero deberá ser llamado `articles/new.html.erb` y es necesario guardarlo en el directorio `app/views` de la aplicación.

Ve adelante ahora y crea el fichero `app/views/articles/new.html.erb` y escribe dentro el siguiente contenido:

```html
<h1>Nuevo Artículo</h1>
```

Cuando recargues <http://localhost:3000/articles/new> verás ahora como la página tiene un título. ¡La ruta, el controlador, la acción y la vista ahora están trabajando armoniosamente! Es hora de crear un formulario para crear artículos.

### El primer formulario

Para crear el formulario dentro de esta plantilla, vamoa a utilizar un *constructor de formularios*. El principal constructor de formularios para Rails es proporcionado por un método ayudante (helper)
llamado `form_for`. Para utilizar este método, añade este código dentro de
`app/views/articles/new.html.erb`:

```html+erb
<%= form_for :article do |f| %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>

  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>

  <p>
    <%= f.submit %>
  </p>
<% end %>
```

Si tu refecargas la página ahora, verás exactamente el mismo formulario que en el ejemplo.
¡Construir formularios en Rails es realmente muy fácil!

Cuando tu llamas al `form_for`, le pasas como parametro un identificador de objeto para este formulario. En este caso, es el símbolo `:article`. Este le dice al método de ayuda `form_for` para que es el formulario. Dentro del bloque de este método, el objeto `FormBuilder` - representado por la `f` - es utilizado para construir dos etiquetas y dos campos de texto, uno para el título y otro para el texto del artículo. Finalmente, para `enviar` el formulario con el objeto `f`, se crea un botón `submit` en él.

Hay un problema con este formulario sin embargo. Si inspecciones al HTML que se generó, viendo el código fuente de la página, verás que el atribute `action` de este formulario esta apuntando a `/articles/new`. Este es un problema porque esta ruta va a la misma página en la que tu estás en este momento, y esta ruta debería ser usada para mostrar el formulario de un artículo nuevo.

El formulario necesita utilizar una URL diferente para ir a un lugar diferente. Est puede hacerse simplemente con la opción `:url` del método `form_for`.
Es típico en Rails, que la acción que es utilizada para el envío de un nuevo formulario se llame "create", y por tanto el formulario debería apuntar a esta acción.

Edita línea `form_for` dentro de `app/views/articles/new.html.erb` para que queda así:

```html+erb
<%= form_for :article, url: articles_path do |f| %>
```

En este ejemplo, al método de ayuda `articles_path` se le pasa la opción `:url`.
Si quieres saber lo que Rails hará con esto, echemos un vistazo en la salida del comando `rake routes`:

```bash
$ bin/rake routes
      Prefix Verb   URI Pattern                  Controller#Action
    articles GET    /articles(.:format)          articles#index
             POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show
             PATCH  /articles/:id(.:format)      articles#update
             PUT    /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
        root GET    /                            welcome#index
```

El metodo de ayuda `articles_path` le dice a Rails que apunte el formulario hacia el patrón URI asociado con el prefijo `articles`; y el formulario será (por defecto) enviado por una petición
`POST` hacia esa ruta. Esta fue asociada con la acción `create` del actual controlador, el `ArticlesController`.

Con el formulario an sus definidas rutas asociadas, estarás habilitado a rellenar el fomulario y luego de hacer click en el botón submit a comenzar el proceso de crear el nuevo artículo, asi que a seguir adelante y hacerlo. Cuando envíes el formulario, deberías ver un error conocido:


![La acción create es desconocida para ArticlesController]
(images/getting_started/unknown_action_create_for_articles.png)

Necesitas ahora crear la acción `create` action dentro de el `ArticlesController` para que esto funcione.

### Creando artículos

Para hacer que se vaya el "Unknown action", pudes definir una acción `create` dentro de la clase `ArticlesController` en el fichero `app/controllers/articles_controller.rb`, por debajo de la acción `new`, como se muestra:

```ruby
class ArticlesController < ApplicationController
  def new
  end

  def create
  end
end
```

Si estás reenviando el formulario ahora, verás otro error conocido: la plantilla esta perdida. Está bien, podemos ignorar esto por ahora. Que la acción `create` deberia action guardar el nuevo artículo en la base de datos.

Cuando el formulario es enviado, los campos del formulario son enviados por Rails como _parámetros_. Estos parámetros puden ser invocados dentro de la acción en el controlador, tipicamente para realizar una tarea específica. Para ver como esos parametros se ven, cambia la acción `create` por esto:

```ruby
def create
  render plain: params[:article].inspect
end
```

El metodo `render` aqui está tomando un simple hash con una clave `plain` y el valor de  `params[:article].inspect`. El metodo `params` es un objeto, que representa los parametros (o campos) que vienen desde el formulario. El método `params` retorna un objeto `ActiveSupport::HashWithIndifferentAccess`, el cual te habilita a acceder a las claves del hash utilizando o cadenas de texto o símbolos. En esta situación, los únicos parámetros que inportan son aquellos que vienen desde el formulario.

TIP: Asegurate de tener una sólida comprensión del método `params`, porque lo utilizarás con bastante regulardidad. Vamos a considerar este ejemplo de URL: **http://www.example.com/?username=dhh&email=dhh@email.com**. En esta URL, `params[:username]` sería igual a "dhh" y `params[:email]` sería igual a "dhh@email.com".

Si tu reenvías el formulario, una vez más no tendrás ahora el error de plantilla perdida. En su lugar, tendrás algo que se verá así:

```ruby
{"title"=>"Primer artículo", "text"=>"Este es mi primer artículo."}
```

Esta acción esta ahora mostrando los parametros del artículo que ha llegado desde el formulario. Sin embargo, esto no es realmente útil. Si puedes ver los parametros pero nada en particular se está haciendo con ellos.

### Creando el Modelo Article

Los modelos en Rails utilizan un nombre en singular, y se corresponden con las las tablas de la base de datos que utilizan el nombre pero en plural. Rails provee un generador para crear modelos, que la mayoría de los desarrolladores Rails tienden a utilizar cuando crean nuevos modelos. Para crear un nuevo modelo, ejecuta este comando en tu terminal:

```bash
$ bin/rails generate model Article title:string text:text
```

Con este comando estamos diciéndole a Rails que queremos un modelo `Article`, junto con un atributo _title_ del tipo string, y un atributo _text_ del tipo text. Estos atributos son automaticamente añadidos a la tabla `articles` en la base de datos y mapeados en el modelo `Article`.

Rails responde creando un grupo de ficheros. Por ahora, nosotros solo estamos interesados en `app/models/article.rb` y en `db/migrate/20140120191729_create_articles.rb`
(el nombre de tu fichero será un poco diferente). El último es el responsable de crear la estructura de base de datos, la cual veremos más adelante.

TIP: Active Record is lo suficientemente inteligente para crear automaticamente una relación entre el nombre de las columnas de la base de datos y los atributos del modelo, esto significa que no deberás declarar atributos dentro de los modelos Rails, esto será hecho automaticamente por Active Record.

### Ejecutando una Migración

Como acabamos de ver, `rails generate model` creo un fichero _de migración de la base de datos_ dentro del directorio `db/migrate`. Las migraciones son clases Ruby que están diseñadas para hacer simple de crear y modificar las tablas de la base de datos. Rails utiliza comandos rake para ejecutar las migraciones, y es posible deshacer una migración después de que fue aplicada a la base de datos. Los nombres de los ficheros de migracioens incluyen una marca de tiempo para asegurar que serán procesados en el orden en que fueron creados.

Si miras dentro del fichero `db/migrate/YYYYMMDDHHMMSS_create_articles.rb`
(recuerda, el nombre de tu fichero será levemente diferente), encontrarás esto:

```ruby
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.string :title
      t.text :text

      t.timestamps null: false
    end
  end
end
```

La migración de arriba crea un metodo llamado `change` que será llamado cuando ejectues esta migración. La acción definida en este método es también reversible, esto significa que Rails conoce como revertir el cambio hecho por esta migración, en caso de que quieras retroceder luego más adelante. Cuando ejecutes la migración ella creará una tabla `articles` con una columna string y otra de tipo text. También creará dos campos timestamp para permitir a Rails guardar las horas en que se crea y se modifica el artículo.

TIP: Para más información acerca de las migraciones, sigue a [Migraciones de la Base de Datos en Rails]
(migrations.html).

En este punto, puedes usar el comando rake para ejectuar la migración:

```bash
$ bin/rake db:migrate
```

Rails ejecutará el comando de esta migración y te dirá que ha creado la tabla Articles.

```bash
==  CreateArticles: migrating ==================================================
-- create_table(:articles)
   -> 0.0019s
==  CreateArticles: migrated (0.0020s) =========================================
```

NOTE. Porque estás trabajando en un ambiente de desarrollo por defecto, este comando será aplicado a la base de datos definida en la sección `development` del fichero `config/database.yml`. Si tu quieres ejecutar la migración en otro ambiente, por ejemplo producción, debes añadir explicitamente el entorno en el comando: `rake db:migrate RAILS_ENV=production`.

### Grabando los datos en el controlador

Regresando al `ArticlesController`, necesitamos cambiar la acción `create` para que la utilice el nuevo modelo `Article` para grabar los datos en la base de datos.
Abre `app/controllers/articles_controller.rb` y modifica la acción `create` de la siguiente manera:

```ruby
def create
  @article = Article.new(params[:article])

  @article.save
  redirect_to @article
end
```

Esto es lo que está pasando: todos los modelos Rails pueden ser inicializados con sus respectivos atributos, los cuales serán automaticamente relacionados con sus respectivas columnas en la base de datos. En la primera línea hemos hecho justo esto (recuerda que `params[:article]` contiene los atributos que nos interesa dentro). Luego,
`@article.save` es el responsable de guardar el modelo en la base de datos. Finalmente redirigimos al usuario a la acción `show`, que definiremos luego.

TIP: Tu podrías preguntar porqué la `A` en `Article.new` es mayúsculas en el código de encima, mientras la mayoría de referencias a articles son en minusculas en este capítulo. En este contexto, nos estamos refiriendo a la clase llamada `Article` que está definida en `app/models/article.rb`. Los nombres de las clases en Ruby deben comenzar con una letra mayúsculas.

TIP: Como veremos más adelante, `@article.save` retorna un valor boleano que indica si el artículo fue guardado en la base de datos o no.

Si ahora vas a <http://localhost:3000/articles/new> estarás *casi siempre* posibilitado para crear un artículo. ¡Pruebalo! Deberías ver el siguiente error:

![Atributos prohibidos para el nuevo artículo]
(images/getting_started/forbidden_attributes_for_new_article.png)

Railst tiene varias caraterísticas que te ayudarán a escribir aplicaciones seguras, y estás ejecutando una de ellas en este momento. Es llamada [parametros fuertes](action_controller_overview.html#strong-parameters), que requiere que le digamos a Rails exactamente que parametros están permitidos dentro de las acciones de nuestro controlador.

¿Porque tienes que preocuparte? La posibilidad de grabar asignando automáticamente a todos los controladores parámetros de tu modelo de un tirón hace el trabajo de los programadores más fácil, pero esta conveniendia también permite el uso malicioso. ¿Que pasaría si la petición para crear un nuevo artículo se utilizara para incluir campos extra con valores que violaran la integridad de la aplicación? Podrían ser asignados masivamente en el modelo y luego en la base de datos junto con los datos buenos, y potencialmente podrían romper la aplicación o algo peor.

Nosotros tenemos una lista blanca en nuestro controlador con los parametros para prevenir que otros datos se asignen masivamente. En este caso, queremos ambos, permitir y requerir los parámetros `title` y `text` para validar el uso de `create`. En la sintaxis conoceremos a
`require` y `permit`. El cambio solo abarcará una línea en la acción `create`:

```ruby
  @article = Article.new(params.require(:article).permit(:title, :text))
```

This is often factored out into its own method so it can be reused by multiple
actions in the same controller, for example `create` and `update`. Above and
beyond mass assignment issues, the method is often made `private` to make sure
it can't be called outside its intended context. Here is the result:

```ruby
def create
  @article = Article.new(article_params)

  @article.save
  redirect_to @article
end

private
  def article_params
    params.require(:article).permit(:title, :text)
  end
```

TIP: For more information, refer to the reference above and
[this blog article about Strong Parameters]
(http://weblog.rubyonrails.org/2012/3/21/strong-parameters/).

### Showing Articles

If you submit the form again now, Rails will complain about not finding the
`show` action. That's not very useful though, so let's add the `show` action
before proceeding.

As we have seen in the output of `rake routes`, the route for `show` action is
as follows:

```
article GET    /articles/:id(.:format)      articles#show
```

The special syntax `:id` tells rails that this route expects an `:id`
parameter, which in our case will be the id of the article.

As we did before, we need to add the `show` action in
`app/controllers/articles_controller.rb` and its respective view.

NOTE: A frequent practice is to place the standard CRUD actions in each
controller in the following order: `index`, `show`, `new`, `edit`, `create`, `update`
and `destroy`. You may use any order you choose, but keep in mind that these
are public methods; as mentioned earlier in this guide, they must be placed
before any private or protected method in the controller in order to work.

Given that, let's add the `show` action, as follows:

```ruby
class ArticlesController < ApplicationController
  def show
    @article = Article.find(params[:id])
  end

  def new
  end

  # snipped for brevity
```

A couple of things to note. We use `Article.find` to find the article we're
interested in, passing in `params[:id]` to get the `:id` parameter from the
request. We also use an instance variable (prefixed with `@`) to hold a
reference to the article object. We do this because Rails will pass all instance
variables to the view.

Now, create a new file `app/views/articles/show.html.erb` with the following
content:

```html+erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>

<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
```

With this change, you should finally be able to create new articles.
Visit <http://localhost:3000/articles/new> and give it a try!

![Show action for articles](images/getting_started/show_action_for_articles.png)

### Listing all articles

We still need a way to list all our articles, so let's do that.
The route for this as per output of `rake routes` is:

```
articles GET    /articles(.:format)          articles#index
```

Add the corresponding `index` action for that route inside the
`ArticlesController` in the `app/controllers/articles_controller.rb` file.
When we write an `index` action, the usual practice is to place it as the
first method in the controller. Let's do it:

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
  end

  # snipped for brevity
```

And then finally, add the view for this action, located at
`app/views/articles/index.html.erb`:

```html+erb
<h1>Listing articles</h1>

<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
  </tr>

  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
    </tr>
  <% end %>
</table>
```

Now if you go to <http://localhost:3000/articles> you will see a list of all the
articles that you have created.

### Adding links

You can now create, show, and list articles. Now let's add some links to
navigate through pages.

Open `app/views/welcome/index.html.erb` and modify it as follows:

```html+erb
<h1>Hello, Rails!</h1>
<%= link_to 'My Blog', controller: 'articles' %>
```

The `link_to` method is one of Rails' built-in view helpers. It creates a
hyperlink based on text to display and where to go - in this case, to the path
for articles.

Let's add links to the other views as well, starting with adding this
"New Article" link to `app/views/articles/index.html.erb`, placing it above the
`<table>` tag:

```erb
<%= link_to 'New article', new_article_path %>
```

This link will allow you to bring up the form that lets you create a new article.

Now, add another link in `app/views/articles/new.html.erb`, underneath the
form, to go back to the `index` action:

```erb
<%= form_for :article, url: articles_path do |f| %>
  ...
<% end %>

<%= link_to 'Back', articles_path %>
```

Finally, add a link to the `app/views/articles/show.html.erb` template to
go back to the `index` action as well, so that people who are viewing a single
article can go back and view the whole list again:

```html+erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>

<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<%= link_to 'Back', articles_path %>
```

TIP: If you want to link to an action in the same controller, you don't need to
specify the `:controller` option, as Rails will use the current controller by
default.

TIP: In development mode (which is what you're working in by default), Rails
reloads your application with every browser request, so there's no need to stop
and restart the web server when a change is made.

### Adding Some Validation

The model file, `app/models/article.rb` is about as simple as it can get:

```ruby
class Article < ActiveRecord::Base
end
```

There isn't much to this file - but note that the `Article` class inherits from
`ActiveRecord::Base`. Active Record supplies a great deal of functionality to
your Rails models for free, including basic database CRUD (Create, Read, Update,
Destroy) operations, data validation, as well as sophisticated search support
and the ability to relate multiple models to one another.

Rails includes methods to help you validate the data that you send to models.
Open the `app/models/article.rb` file and edit it:

```ruby
class Article < ActiveRecord::Base
  validates :title, presence: true,
                    length: { minimum: 5 }
end
```

These changes will ensure that all articles have a title that is at least five
characters long. Rails can validate a variety of conditions in a model,
including the presence or uniqueness of columns, their format, and the
existence of associated objects. Validations are covered in detail in [Active
Record Validations](active_record_validations.html).

With the validation now in place, when you call `@article.save` on an invalid
article, it will return `false`. If you open
`app/controllers/articles_controller.rb` again, you'll notice that we don't
check the result of calling `@article.save` inside the `create` action.
If `@article.save` fails in this situation, we need to show the form back to the
user. To do this, change the `new` and `create` actions inside
`app/controllers/articles_controller.rb` to these:

```ruby
def new
  @article = Article.new
end

def create
  @article = Article.new(article_params)

  if @article.save
    redirect_to @article
  else
    render 'new'
  end
end

private
  def article_params
    params.require(:article).permit(:title, :text)
  end
```

The `new` action is now creating a new instance variable called `@article`, and
you'll see why that is in just a few moments.

Notice that inside the `create` action we use `render` instead of `redirect_to`
when `save` returns `false`. The `render` method is used so that the `@article`
object is passed back to the `new` template when it is rendered. This rendering
is done within the same request as the form submission, whereas the
`redirect_to` will tell the browser to issue another request.

If you reload
<http://localhost:3000/articles/new> and
try to save an article without a title, Rails will send you back to the
form, but that's not very useful. You need to tell the user that
something went wrong. To do that, you'll modify
`app/views/articles/new.html.erb` to check for error messages:

```html+erb
<%= form_for :article, url: articles_path do |f| %>

  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>

  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>

  <p>
    <%= f.submit %>
  </p>

<% end %>

<%= link_to 'Back', articles_path %>
```

A few things are going on. We check if there are any errors with
`@article.errors.any?`, and in that case we show a list of all
errors with `@article.errors.full_messages`.

`pluralize` is a rails helper that takes a number and a string as its
arguments. If the number is greater than one, the string will be automatically
pluralized.

The reason why we added `@article = Article.new` in the `ArticlesController` is
that otherwise `@article` would be `nil` in our view, and calling
`@article.errors.any?` would throw an error.

TIP: Rails automatically wraps fields that contain an error with a div
with class `field_with_errors`. You can define a css rule to make them
standout.

Now you'll get a nice error message when saving an article without title when
you attempt to do just that on the new article form
<http://localhost:3000/articles/new>:

![Form With Errors](images/getting_started/form_with_errors.png)

### Updating Articles

We've covered the "CR" part of CRUD. Now let's focus on the "U" part, updating
articles.

The first step we'll take is adding an `edit` action to the `ArticlesController`,
generally between the `new` and `create` actions, as shown:

```ruby
def new
  @article = Article.new
end

def edit
  @article = Article.find(params[:id])
end

def create
  @article = Article.new(article_params)

  if @article.save
    redirect_to @article
  else
    render 'new'
  end
end
```

The view will contain a form similar to the one we used when creating
new articles. Create a file called `app/views/articles/edit.html.erb` and make
it look as follows:

```html+erb
<h1>Editing article</h1>

<%= form_for :article, url: article_path(@article), method: :patch do |f| %>

  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>

  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>

  <p>
    <%= f.submit %>
  </p>

<% end %>

<%= link_to 'Back', articles_path %>
```

This time we point the form to the `update` action, which is not defined yet
but will be very soon.

The `method: :patch` option tells Rails that we want this form to be submitted
via the `PATCH` HTTP method which is the HTTP method you're expected to use to
**update** resources according to the REST protocol.

The first parameter of `form_for` can be an object, say, `@article` which would
cause the helper to fill in the form with the fields of the object. Passing in a
symbol (`:article`) with the same name as the instance variable (`@article`)
also automagically leads to the same behavior. This is what is happening here.
More details can be found in [form_for documentation]
(http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_for).

Next, we need to create the `update` action in
`app/controllers/articles_controller.rb`.
Add it between the `create` action and the `private` method:

```ruby
def create
  @article = Article.new(article_params)

  if @article.save
    redirect_to @article
  else
    render 'new'
  end
end

def update
  @article = Article.find(params[:id])

  if @article.update(article_params)
    redirect_to @article
  else
    render 'edit'
  end
end

private
  def article_params
    params.require(:article).permit(:title, :text)
  end
```

The new method, `update`, is used when you want to update a record
that already exists, and it accepts a hash containing the attributes
that you want to update. As before, if there was an error updating the
article we want to show the form back to the user.

We reuse the `article_params` method that we defined earlier for the create
action.

TIP: You don't need to pass all attributes to `update`. For
example, if you'd call `@article.update(title: 'A new title')`
Rails would only update the `title` attribute, leaving all other
attributes untouched.

Finally, we want to show a link to the `edit` action in the list of all the
articles, so let's add that now to `app/views/articles/index.html.erb` to make
it appear next to the "Show" link:

```html+erb
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th colspan="2"></th>
  </tr>

  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
    </tr>
  <% end %>
</table>
```

And we'll also add one to the `app/views/articles/show.html.erb` template as
well, so that there's also an "Edit" link on an article's page. Add this at the
bottom of the template:

```html+erb
...

<%= link_to 'Back', articles_path %> |
<%= link_to 'Edit', edit_article_path(@article) %>
```

And here's how our app looks so far:

![Index action with edit link](images/getting_started/index_action_with_edit_link.png)

### Using partials to clean up duplication in views

Our `edit` page looks very similar to the `new` page; in fact, they
both share the same code for displaying the form. Let's remove this
duplication by using a view partial. By convention, partial files are
prefixed with an underscore.

TIP: You can read more about partials in the
[Layouts and Rendering in Rails](layouts_and_rendering.html) guide.

Create a new file `app/views/articles/_form.html.erb` with the following
content:

```html+erb
<%= form_for @article do |f| %>

  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>

  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>

  <p>
    <%= f.submit %>
  </p>

<% end %>
```

Everything except for the `form_for` declaration remained the same.
The reason we can use this shorter, simpler `form_for` declaration
to stand in for either of the other forms is that `@article` is a *resource*
corresponding to a full set of RESTful routes, and Rails is able to infer
which URI and method to use.
For more information about this use of `form_for`, see [Resource-oriented style]
(http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_for-label-Resource-oriented+style).

Now, let's update the `app/views/articles/new.html.erb` view to use this new
partial, rewriting it completely:

```html+erb
<h1>New article</h1>

<%= render 'form' %>

<%= link_to 'Back', articles_path %>
```

Then do the same for the `app/views/articles/edit.html.erb` view:

```html+erb
<h1>Edit article</h1>

<%= render 'form' %>

<%= link_to 'Back', articles_path %>
```

### Deleting Articles

We're now ready to cover the "D" part of CRUD, deleting articles from the
database. Following the REST convention, the route for
deleting articles as per output of `rake routes` is:

```ruby
DELETE /articles/:id(.:format)      articles#destroy
```

The `delete` routing method should be used for routes that destroy
resources. If this was left as a typical `get` route, it could be possible for
people to craft malicious URLs like this:

```html
<a href='http://example.com/articles/1/destroy'>look at this cat!</a>
```

We use the `delete` method for destroying resources, and this route is mapped
to the `destroy` action inside `app/controllers/articles_controller.rb`, which
doesn't exist yet. The `destroy` method is generally the last CRUD action in
the controller, and like the other public CRUD actions, it must be placed
before any `private` or `protected` methods. Let's add it:

```ruby
def destroy
  @article = Article.find(params[:id])
  @article.destroy

  redirect_to articles_path
end
```

The complete `ArticlesController` in the
`app/controllers/articles_controller.rb` file should now look like this:

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def edit
    @article = Article.find(params[:id])
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render 'new'
    end
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy

    redirect_to articles_path
  end

  private
    def article_params
      params.require(:article).permit(:title, :text)
    end
end
```

You can call `destroy` on Active Record objects when you want to delete
them from the database. Note that we don't need to add a view for this
action since we're redirecting to the `index` action.

Finally, add a 'Destroy' link to your `index` action template
(`app/views/articles/index.html.erb`) to wrap everything together.

```html+erb
<h1>Listing Articles</h1>
<%= link_to 'New article', new_article_path %>
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th colspan="3"></th>
  </tr>

  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
      <td><%= link_to 'Destroy', article_path(article),
              method: :delete,
              data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</table>
```

Here we're using `link_to` in a different way. We pass the named route as the
second argument, and then the options as another argument. The `:method` and
`:'data-confirm'` options are used as HTML5 attributes so that when the link is
clicked, Rails will first show a confirm dialog to the user, and then submit the
link with method `delete`.  This is done via the JavaScript file `jquery_ujs`
which is automatically included into your application's layout
(`app/views/layouts/application.html.erb`) when you generated the application.
Without this file, the confirmation dialog box wouldn't appear.

![Confirm Dialog](images/getting_started/confirm_dialog.png)

Congratulations, you can now create, show, list, update and destroy
articles.

TIP: In general, Rails encourages using resources objects instead of
declaring routes manually. For more information about routing, see
[Rails Routing from the Outside In](routing.html).

Adding a Second Model
---------------------

It's time to add a second model to the application. The second model will handle
comments on articles.

### Generating a Model

We're going to see the same generator that we used before when creating
the `Article` model. This time we'll create a `Comment` model to hold
reference of article comments. Run this command in your terminal:

```bash
$ bin/rails generate model Comment commenter:string body:text article:references
```

This command will generate four files:

| File                                         | Purpose                                                                                                |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| db/migrate/20140120201010_create_comments.rb | Migration to create the comments table in your database (your name will include a different timestamp) |
| app/models/comment.rb                        | The Comment model                                                                                      |
| test/models/comment_test.rb                  | Testing harness for the comments model                                                                 |
| test/fixtures/comments.yml                   | Sample comments for use in testing                                                                     |

First, take a look at `app/models/comment.rb`:

```ruby
class Comment < ActiveRecord::Base
  belongs_to :article
end
```

This is very similar to the `Article` model that you saw earlier. The difference
is the line `belongs_to :article`, which sets up an Active Record _association_.
You'll learn a little about associations in the next section of this guide.

In addition to the model, Rails has also made a migration to create the
corresponding database table:

```ruby
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.string :commenter
      t.text :body

      # this line adds an integer column called `article_id`.
      t.references :article, index: true

      t.timestamps null: false
    end
    add_foreign_key :comments, :articles
  end
end
```

The `t.references` line sets up a foreign key column for the association between
the two models. An index for this association is also created on this column.
Go ahead and run the migration:

```bash
$ bin/rake db:migrate
```

Rails is smart enough to only execute the migrations that have not already been
run against the current database, so in this case you will just see:

```bash
==  CreateComments: migrating =================================================
-- create_table(:comments)
   -> 0.0115s
-- add_foreign_key(:comments, :articles)
   -> 0.0000s
==  CreateComments: migrated (0.0119s) ========================================
```

### Associating Models

Active Record associations let you easily declare the relationship between two
models. In the case of comments and articles, you could write out the
relationships this way:

* Each comment belongs to one article.
* One article can have many comments.

In fact, this is very close to the syntax that Rails uses to declare this
association. You've already seen the line of code inside the `Comment` model
(app/models/comment.rb) that makes each comment belong to an Article:

```ruby
class Comment < ActiveRecord::Base
  belongs_to :article
end
```

You'll need to edit `app/models/article.rb` to add the other side of the
association:

```ruby
class Article < ActiveRecord::Base
  has_many :comments
  validates :title, presence: true,
                    length: { minimum: 5 }
end
```

These two declarations enable a good bit of automatic behavior. For example, if
you have an instance variable `@article` containing an article, you can retrieve
all the comments belonging to that article as an array using
`@article.comments`.

TIP: For more information on Active Record associations, see the [Active Record
Associations](association_basics.html) guide.

### Adding a Route for Comments

As with the `welcome` controller, we will need to add a route so that Rails
knows where we would like to navigate to see `comments`. Open up the
`config/routes.rb` file again, and edit it as follows:

```ruby
resources :articles do
  resources :comments
end
```

This creates `comments` as a _nested resource_ within `articles`. This is
another part of capturing the hierarchical relationship that exists between
articles and comments.

TIP: For more information on routing, see the [Rails Routing](routing.html)
guide.

### Generating a Controller

With the model in hand, you can turn your attention to creating a matching
controller. Again, we'll use the same generator we used before:

```bash
$ bin/rails generate controller Comments
```

This creates five files and one empty directory:

| File/Directory                               | Purpose                                  |
| -------------------------------------------- | ---------------------------------------- |
| app/controllers/comments_controller.rb       | The Comments controller                  |
| app/views/comments/                          | Views of the controller are stored here  |
| test/controllers/comments_controller_test.rb | The test for the controller              |
| app/helpers/comments_helper.rb               | A view helper file                       |
| app/assets/javascripts/comment.coffee        | CoffeeScript for the controller          |
| app/assets/stylesheets/comment.scss          | Cascading style sheet for the controller |

Like with any blog, our readers will create their comments directly after
reading the article, and once they have added their comment, will be sent back
to the article show page to see their comment now listed. Due to this, our
`CommentsController` is there to provide a method to create comments and delete
spam comments when they arrive.

So first, we'll wire up the Article show template
(`app/views/articles/show.html.erb`) to let us make a new comment:

```html+erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>

<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<h2>Add a comment:</h2>
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>

<%= link_to 'Back', articles_path %> |
<%= link_to 'Edit', edit_article_path(@article) %>
```

This adds a form on the `Article` show page that creates a new comment by
calling the `CommentsController` `create` action. The `form_for` call here uses
an array, which will build a nested route, such as `/articles/1/comments`.

Let's wire up the `create` in `app/controllers/comments_controller.rb`:

```ruby
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
```

You'll see a bit more complexity here than you did in the controller for
articles. That's a side-effect of the nesting that you've set up. Each request
for a comment has to keep track of the article to which the comment is attached,
thus the initial call to the `find` method of the `Article` model to get the
article in question.

In addition, the code takes advantage of some of the methods available for an
association. We use the `create` method on `@article.comments` to create and
save the comment. This will automatically link the comment so that it belongs to
that particular article.

Once we have made the new comment, we send the user back to the original article
using the `article_path(@article)` helper. As we have already seen, this calls
the `show` action of the `ArticlesController` which in turn renders the
`show.html.erb` template. This is where we want the comment to show, so let's
add that to the `app/views/articles/show.html.erb`.

```html+erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>

<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<h2>Comments</h2>
<% @article.comments.each do |comment| %>
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>

  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>
<% end %>

<h2>Add a comment:</h2>
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>

<%= link_to 'Edit Article', edit_article_path(@article) %> |
<%= link_to 'Back to Articles', articles_path %>
```

Now you can add articles and comments to your blog and have them show up in the
right places.

![Article with Comments](images/getting_started/article_with_comments.png)

Refactoring
-----------

Now that we have articles and comments working, take a look at the
`app/views/articles/show.html.erb` template. It is getting long and awkward. We
can use partials to clean it up.

### Rendering Partial Collections

First, we will make a comment partial to extract showing all the comments for
the article. Create the file `app/views/comments/_comment.html.erb` and put the
following into it:

```html+erb
<p>
  <strong>Commenter:</strong>
  <%= comment.commenter %>
</p>

<p>
  <strong>Comment:</strong>
  <%= comment.body %>
</p>
```

Then you can change `app/views/articles/show.html.erb` to look like the
following:

```html+erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>

<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<h2>Comments</h2>
<%= render @article.comments %>

<h2>Add a comment:</h2>
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>

<%= link_to 'Edit Article', edit_article_path(@article) %> |
<%= link_to 'Back to Articles', articles_path %>
```

This will now render the partial in `app/views/comments/_comment.html.erb` once
for each comment that is in the `@article.comments` collection. As the `render`
method iterates over the `@article.comments` collection, it assigns each
comment to a local variable named the same as the partial, in this case
`comment` which is then available in the partial for us to show.

### Rendering a Partial Form

Let us also move that new comment section out to its own partial. Again, you
create a file `app/views/comments/_form.html.erb` containing:

```html+erb
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```

Then you make the `app/views/articles/show.html.erb` look like the following:

```html+erb
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>

<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>

<h2>Comments</h2>
<%= render @article.comments %>

<h2>Add a comment:</h2>
<%= render 'comments/form' %>

<%= link_to 'Edit Article', edit_article_path(@article) %> |
<%= link_to 'Back to Articles', articles_path %>
```

The second render just defines the partial template we want to render,
`comments/form`. Rails is smart enough to spot the forward slash in that
string and realize that you want to render the `_form.html.erb` file in
the `app/views/comments` directory.

The `@article` object is available to any partials rendered in the view because
we defined it as an instance variable.

Deleting Comments
-----------------

Another important feature of a blog is being able to delete spam comments. To do
this, we need to implement a link of some sort in the view and a `destroy`
action in the `CommentsController`.

So first, let's add the delete link in the
`app/views/comments/_comment.html.erb` partial:

```html+erb
<p>
  <strong>Commenter:</strong>
  <%= comment.commenter %>
</p>

<p>
  <strong>Comment:</strong>
  <%= comment.body %>
</p>

<p>
  <%= link_to 'Destroy Comment', [comment.article, comment],
               method: :delete,
               data: { confirm: 'Are you sure?' } %>
</p>
```

Clicking this new "Destroy Comment" link will fire off a `DELETE
/articles/:article_id/comments/:id` to our `CommentsController`, which can then
use this to find the comment we want to delete, so let's add a `destroy` action
to our controller (`app/controllers/comments_controller.rb`):

```ruby
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end

  def destroy
    @article = Article.find(params[:article_id])
    @comment = @article.comments.find(params[:id])
    @comment.destroy
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
```

The `destroy` action will find the article we are looking at, locate the comment
within the `@article.comments` collection, and then remove it from the
database and send us back to the show action for the article.


### Deleting Associated Objects

If you delete an article, its associated comments will also need to be
deleted, otherwise they would simply occupy space in the database. Rails allows
you to use the `dependent` option of an association to achieve this. Modify the
Article model, `app/models/article.rb`, as follows:

```ruby
class Article < ActiveRecord::Base
  has_many :comments, dependent: :destroy
  validates :title, presence: true,
                    length: { minimum: 5 }
end
```

Security
--------

### Basic Authentication

If you were to publish your blog online, anyone would be able to add, edit and
delete articles or delete comments.

Rails provides a very simple HTTP authentication system that will work nicely in
this situation.

In the `ArticlesController` we need to have a way to block access to the
various actions if the person is not authenticated. Here we can use the Rails
`http_basic_authenticate_with` method, which allows access to the requested
action if that method allows it.

To use the authentication system, we specify it at the top of our
`ArticlesController` in `app/controllers/articles_controller.rb`. In our case,
we want the user to be authenticated on every action except `index` and `show`,
so we write that:

```ruby
class ArticlesController < ApplicationController

  http_basic_authenticate_with name: "dhh", password: "secret", except: [:index, :show]

  def index
    @articles = Article.all
  end

  # snipped for brevity
```

We also want to allow only authenticated users to delete comments, so in the
`CommentsController` (`app/controllers/comments_controller.rb`) we write:

```ruby
class CommentsController < ApplicationController

  http_basic_authenticate_with name: "dhh", password: "secret", only: :destroy

  def create
    @article = Article.find(params[:article_id])
    # ...
  end

  # snipped for brevity
```

Now if you try to create a new article, you will be greeted with a basic HTTP
Authentication challenge:

![Basic HTTP Authentication Challenge](images/getting_started/challenge.png)

Other authentication methods are available for Rails applications. Two popular
authentication add-ons for Rails are the
[Devise](https://github.com/plataformatec/devise) rails engine and
the [Authlogic](https://github.com/binarylogic/authlogic) gem,
along with a number of others.


### Other Security Considerations

Security, especially in web applications, is a broad and detailed area. Security
in your Rails application is covered in more depth in
the [Ruby on Rails Security Guide](security.html).


What's Next?
------------

Now that you've seen your first Rails application, you should feel free to
update it and experiment on your own.

Remember you don't have to do everything without help. As you need assistance
getting up and running with Rails, feel free to consult these support
resources:

* The [Ruby on Rails Guides](index.html)
* The [Ruby on Rails Tutorial](http://railstutorial.org/book)
* The [Ruby on Rails mailing list](http://groups.google.com/group/rubyonrails-talk)
* The [#rubyonrails](irc://irc.freenode.net/#rubyonrails) channel on irc.freenode.net

Rails also comes with built-in help that you can generate using the rake
command-line utility:

* Running `rake doc:guides` will put a full copy of the Rails Guides in the
  `doc/guides` folder of your application. Open `doc/guides/index.html` in your
  web browser to explore the Guides.
* Running `rake doc:rails` will put a full copy of the API documentation for
  Rails in the `doc/api` folder of your application. Open `doc/api/index.html`
  in your web browser to explore the API documentation.

TIP: To be able to generate the Rails Guides locally with the `doc:guides` rake
task you need to install the Redcarpet and Nokogiri gems. Add it to your `Gemfile` and run
`bundle install` and you're ready to go.

Configuration Gotchas
---------------------

The easiest way to work with Rails is to store all external data as UTF-8. If
you don't, Ruby libraries and Rails will often be able to convert your native
data into UTF-8, but this doesn't always work reliably, so you're better off
ensuring that all external data is UTF-8.

If you have made a mistake in this area, the most common symptom is a black
diamond with a question mark inside appearing in the browser. Another common
symptom is characters like "Ã¼" appearing instead of "ü". Rails takes a number
of internal steps to mitigate common causes of these problems that can be
automatically detected and corrected. However, if you have external data that is
not stored as UTF-8, it can occasionally result in these kinds of issues that
cannot be automatically detected by Rails and corrected.

Two very common sources of data that are not UTF-8:

* Your text editor: Most text editors (such as TextMate), default to saving
  files as UTF-8. If your text editor does not, this can result in special
  characters that you enter in your templates (such as é) to appear as a diamond
  with a question mark inside in the browser. This also applies to your i18n
  translation files. Most editors that do not already default to UTF-8 (such as
  some versions of Dreamweaver) offer a way to change the default to UTF-8. Do
  so.
* Your database: Rails defaults to converting data from your database into UTF-8
  at the boundary. However, if your database is not using UTF-8 internally, it
  may not be able to store all characters that your users enter. For instance,
  if your database is using Latin-1 internally, and your user enters a Russian,
  Hebrew, or Japanese character, the data will be lost forever once it enters
  the database. If possible, use UTF-8 as the internal storage of your database.
