**NO LEAS ESTE FICHERO EN GITHUB, LAS GUIAS ESTÁN PUBLICADAS EN http://guiasrails.es .**

Una Guía para Probar las Aplicaciones Rails
===========================================

Esta guía cubre la construcción de mecanismos en Rails para testear tu aplicación.

Después de leer esta guía, tu conocerás:

* La terminología de pruebas Rails.
* Como escribir un test unitario, funcional y de integración para tu aplicación.
* Otros enfoques de pruebas populares y plugins.

--------------------------------------------------------------------------------

¿Porqué Escribir Pruebas para tus Aplicaciones Rails?
-----------------------------------------------------

Rails hace super fácil escribir tus pruebas. Comienza produciendo un esqueleto de código de prueba mientras tu estás creando tus modelos y controladores.

Simplemente ejecutando tus pruebas Rails puedes asegurar tu código corresponde a la funcionalidad deseada, incluso después de algunas de las principales refactorizaciones de código.

Las pruebas Rails también pueden simular peticiones del navegador y por lo tanto puedes probar la respuesta de tu aplicación sin tener que probarla en tu navegador.

Introducción a las Pruebas
--------------------------

El soporte para pruebas estuvo dentro de la estructura de Rails desde el principio. No fué porque hemos tenido una aparición divina y pensamos "vamos a apoyar la realización de pruebas porque es nueva y es guay". Casi toda aplicación Rails interactúa fuertemente con una base de datos, como consecuencia, tus pruebas necesitarán una base de datos con la cual interactuar también. Para escribir pruebas eficientes, necesitarás entender como configurar esta base de datos y rellenarla con datos de ejemplo.

### El Entorno de Pruebas

Por defecto, toda aplicación Rails tiene tres entornos:
desarrollo, pruebas y producción. La base de datos para cada uno de ellos se configura en `config/database.yml`.

Una base de datos dedicada a pruebas te permite configurar e interactuar con los los datos de prueba de forma ailsada.
De esta manera tus pruebas pueden destrozar los datos de prueba con confianza, sin la preocupación de los datos en las bases de datos de desarrollo y de producción.

También, la configuración de cada entorno puede ser modificada de manera similar. En este caso, podemos modificar nuetro entorno de pruebas cambiando las opciones que se encuentran en
 `config/environments/test.rb`.

### Rails Configurado para Pruebas desde el Primer Momento

Rails crea un directorio `test` para ti desde que creas el proyecto Rails utilizando `rails new` _nombre_del_proyecto_. Si listas el contenido de este directorio deberías ver:

```bash
$ ls -F test
controllers/    helpers/        mailers/        test_helper.rb
fixtures/       integration/    models/
```

El directorio `models` está pensado para contener las pruebas de tu modelo, el directorio `controllers` está pensado para contener las pruebas de tus controladores y el directorio `integration` está pensado para contener las pruebas que involucran a un conjunto de controladores interactuando. También hay un directorio para probar el envío de correos y otro para probar los ayudantes (helpers) de las vistas.

Los Fixtures son una manera de organizar los datos de test; residen en el directorio `fixtures`.

El fichero `test_helper.rb` contiene la configuración por defecto para las pruebas.

### Los Verdaderos Datos en los Fixtures

Para buenas pruebas, necesitarás dar cierta inteligencia a la configuración de los datos de prueba.
En Rails, puedes manejar esto definiendo y adaptando fixtures.

Puedes encontrar amplia documentación en la [Documentación API de los Fixtures](http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html).

#### ¿Qué son los Fixtures?

_Fixtures_ (datos fijos) es una palabra de fantasía para nombrar a los datos de ejemplo. Los fixtures te permiten poblar la base de datos de prueba con datos predefinidos antes de ejecutar las pruebas. Los fixtures son independientes de la base de datos y se escriben en ficheros YAML. Hay un fichero por cada entidad del modelo.

Encontrarás fixtures bajo el directorio `test/fixtures`. Cuando ejecutas `rails generate model` para crear un nuevo modelo una nueva plantilla fixture del modelo será creada automáticamente y guardada en este directorio.

#### YAML

Fixtures con formato YAML son una manera humanamente amigable para describir los datos de ejemplo. Este tipo de fixtures tienen la extensión en el nombre del fichero **.yml** (como por ejemplo `users.yml`).

Aquí un ejemplo de un fichero fixture en YAML:

```yaml
# lo & behold! I am a YAML comment!
david:
  name: David Heinemeier Hansson
  birthday: 1979-10-15
  profession: Systems development

steve:
  name: Steve Ross Kellock
  birthday: 1974-09-27
  profession: guy with keyboard
```

A cada fixture se le da un nombre seguido de una lista identada de clave/valor separados por dos puntos. Los registros son típicamente separados por un espacio en blanco. Puedes escribir comentarios en un fichero fixture utilizando el caracter # en la primera columna.

Si trabajas con [asociaciones](/association_basics.html), puedes simplemente definir un nodo de referencia entre dos fixtures diferentes. Aquí hay un ejemplo de una asociación `belongs_to`/`has_many`:

```yaml
# In fixtures/categories.yml
about:
  name: About

# In fixtures/articles.yml
one:
  title: Welcome to Rails!
  body: Hello world!
  category: about
```

Nota la clave `category` del artículo `one` creada en `fixtures/articles.yml` tiene el valor `about`. Esto le dice a Rails que traiga la categoría `about` creada en `fixtures/categories.yml`.

NOTE: Para que las asociaciones puedan referenciarse una a otra por el nombre, no puedes especificar el atributo `id:` en el fixture asociado. Rails asignará automáticamente una clave primaria para ser consistente entre ejecuciones. Para más información sobre el comportamiento de esta asociación lee por favor la [Documentación API de los Fixtures](http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html).

#### ERB para Crear Datos

ERB te permite embeber código Ruby entre templates. El formato YAML del fixture es pre-procesado con ERB cuando Rails carga los fixtures. Esto te permite utilizar Ruby para ayudarte a generar datos de ejemplo. Por ejemplo, el siguiente código genera mil usuarios:

```erb
<% 1000.times do |n| %>
user_<%= n %>:
  username: <%= "user#{n}" %>
  email: <%= "user#{n}@example.com" %>
<% end %>
```

#### Fixtures en Acción

Rails por defecto carga automáticamente todos los fixtures desde el directorio `test/fixtures` para las pruebas de tus modelos y controladores. La carga envuelve tres pasos:

1. Borrar cualquier dato existente de la tabla correspondiente al fixture
2. Cargar los datos del fixture en la tabla
3. Volcar los datos del fixture data en un método en caso de que quieras acceder a ellos directamente

TIP: Para borrar los datos existentes de la base de datos, Rails trata de deshabilitar los disparadores de integridad referencial (como las claves foráneas y la comprobación de restricciones). Si se producen errores molestos sobre permisos cuando ejecutas las pruebas, hay que asegurarse que el usuario de base de datos tiene los privilegios para deshabilitar esos disparadores en el entorno de pruebas. (En PostgreSQL, solo los susperusuarios pueden deshabilitar todos los disparadores. Lee más acerca los permisos en PostgreSQL [aquí](http://blog.endpoint.com/2012/10/postgres-system-triggers-error.html))

#### Los fixtures son objetos Active Record

Los fixtures son instancias de Active Record. Como se menciona en el punto #3 anterior, puedes acceder al objeto directamente porque está automáticamente disponible como un método cuyo alcance es local para el caso de prueba. Por ejemplo:

```ruby
# esto retornara el objeto User para el fixture llamado david
users(:david)

# esto retornará la propiedad de david llamada id
users(:david).id

# one también puede acceder a los métodos disponibles en la clase User
email(david.girlfriend.email, david.location_tonight)
```

### Tareas Rake para la Ejecución de las Pruebas

Rails viene con un numero de tareas rake construidas para ayudar a probar. La tabla de abajo lista los comandos incluídos en el Rakefile por defecto cuando un proyecto Rails es creado.

| Tareas                  | Descripción |
| ----------------------- | ----------- |
| `rake test`             | Ejecuta todas las pruebas en el directorio `test`. También puedes ejecutar `rake` y Rails ejecutará todos las pruebas por defecto |
| `rake test:controllers` | Ejecuta todas las pruebas de controlador desde `test/controllers` |
| `rake test:functionals` | Ejecuta todas las pruebas funcionales desde `test/controllers`, `test/mailers`, y `test/functional` |
| `rake test:helpers`     | Ejecuta todas las pruebas de los helpers desde `test/helpers` |
| `rake test:integration` | Ejecuta tadas las pruebas de integración desde `test/integration` |
| `rake test:jobs`        | Ejecuta todas las pruebas de trabajos desde `test/jobs` |
| `rake test:mailers`     | Ejecuta todas las pruebas de envíos de correos desde `test/mailers` |
| `rake test:models`      | Ejecuta todas las pruebas del modelo desde `test/models` |
| `rake test:units`       | Ejecuta todas las pruebas unitarias desde `test/models`, `test/helpers`, y `test/unit` |
| `rake test:db`          | Ejecuta todas las pruebas en el directorio `test` y resetea la base de datos |

Vamos a cubrir cada tipo de pruebas Rails listadas arriba en esta guía.

Pruebas Unitarias del Modelo
----------------------------

En Rails, las pruebas unitarias son lo que escribes para probar el modelo.

Para esta guía estaremos usando la aplicación que construímos en la guía [Comenzando con Rails](getting_started.html) guide.

Si recuerdas cuando utilizasteis el comando `rails generate scaffold` al principio. Creamos nuestro primer recurso entre otras cosas se creó una plantilla de código en el directorio `test/models`:

```bash
$ bin/rails generate scaffold article title:string body:text
...
create  app/models/article.rb
create  test/models/article_test.rb
create  test/fixtures/articles.yml
...
```

La plantilla de código por defecto `test/models/article_test.rb` es la siguiente:

```ruby
require 'test_helper'

class ArticleTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
```

Examinar línea por línea de este fichero nos ayudará a orientarte en el código de pruebas Rails y la terminología.

```ruby
require 'test_helper'
```

Por requerimiento este fichero, `test_helper.rb` la configuración por defecto para ejecutar nuestro test es cargada. Incluiremos este con todas las pruebas que escribimos, entonces cualquiera de los métodos añadidos a este fichero están disponibles para todas tus pruebas.

```ruby
class ArticleTest < ActiveSupport::TestCase
```

La clase `ArticleTest` define un _caso de pueba_ porque hereda de `ActiveSupport::TestCase`. `ArticleTest` por consiguiente tiene todos los métodos disponibles de `ActiveSupport::TestCase`. Más tarde en esta guía, verás algo de lo que estos métodos te darán.

Cualquier método definido dentro de una clase hereda de `Minitest::Test` (que es la super clase de  `ActiveSupport::TestCase`) y comienza con `test_` (case sensitive) es simplemente llamado un test. Entonces, los métodos definidos como `test_password` y `test_valid_password` son nombres correctos y son ejecutados automáticamente cuando el caso de prueba se ejecuta.

Rails también añade un método `test` que toma un nombre de test y un bloque. Esto genera un test unitario normal `Minitest::Unit` con los nombres de los métodos prefijados con `test_`. Entonces no debes preocuparte acerca de nombrar los métodos, y puedes escribir algo como:

```ruby
test "the truth" do
  assert true
end
```

Que es aproximadamente lo mismo que escribir esto:

```ruby
def test_the_truth
  assert true
end
```

Sin embargo, solo el macro `test` permite un nombre de test más leíble. Puedes seguir utilizando un método regular sin embargo.

NOTE: El nombre del método es generado reemplazando espacios por guiónes bajos. El resultado no necesita ser un identificador válido sin embargo, el nombre puede contener caracteres de puntuación, etc. Eso es porque tecnicamente en Ruby cualquier cadena puede ser un nombre de método. Esto puede requerir el uso de `define_method` y llamadas `send` para funcionar correctamente, pero formalmente hay pocas restricciones en el nombre.

A continuación, vamos a ver nuestra primera afirmación:

```ruby
assert true
```

Una afirmación es una línea de código que evalúa un objeto (o expresión) con resultados esperados. Por ejemplo, una afimación puede comprobar:

* ¿Es este valor = a aquél valor?
* ¿Es nulo este objeto?
* ¿Esta línea de código lanza una excepción?
* ¿Tiene la contraseña del usuario más de 5 carateres?

Toda prueba debe contener al menos una afirmación, sin restricciones en cuanto a que cantidad de afirmaciones están permitidas. Solo cuando todas las afirmaciones tienen éxito se pasa la prueba.

### Mantenimiento del esquema de la base de datos de prueba

Con el fin de ejecutar las pruebas, necesitaremos que la base de datos de prueba tenga actualizada su estructura. El helper de pruebas comprueba si la base de datos de pruebas tiene alguna migración pendiente. Si es así, intentará cargar el `db/schema.rb` o el `db/structure.sql` dentro de la base de datos de prueba. Si las migraciones están aún pendientes, veremos un error. Usualmente esto indica que el esquema no se ha migrado totalmente. Ejecutando las migraciones nuevamente (`bin/rake db:migrate`) la base de datos de desarrollo actualizará el esquema.

NOTE: Si existen migraciones que requieren modificaciones, la base de datos de pruebas necesita ser reconstruida. Esto se puede hacer ejectuando `bin/rake db:test:prepare`.

### Ejecutando Pruebas

Ejecutar una prueba es tan simple como invocar el fichero que contiene los casos de prueba a través del comando `rake test`.

```bash
$ bin/rake test test/models/article_test.rb
.

Finished tests in 0.009262s, 107.9680 tests/s, 107.9680 assertions/s.

1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
```

Esto ejecutará todos los métodos de prueba del caso de prueba.

También puedes ejecutar un método particular del caso de prueba ejecutando la prueba junto con el nombre del método `test method name`.

```bash
$ bin/rake test test/models/article_test.rb test_the_truth
.

Finished tests in 0.009064s, 110.3266 tests/s, 110.3266 assertions/s.

1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
```

El `.` (punto) arriba indica una prueba que se ha pasado. Cuando una prueba falla se muestra una `F`; cuando la prueba arroja un error se muestra una `E` in su lugar. La última línea de salida es el resumen general.

#### Tu primera prueba fallida

Para ver como una prueba fallida es reportada, puedes añadir una prueba fallida al caso de prueba `article_test.rb`.

```ruby
test "should not save article without title" do
  article = Article.new
  assert_not article.save
end
```

Vamos a ejectuar este prueba que se acaba de agregar.

```bash
$ bin/rake test test/models/article_test.rb test_should_not_save_article_without_title
F

Finished tests in 0.044632s, 22.4054 tests/s, 22.4054 assertions/s.

  1) Failure:
test_should_not_save_article_without_title(ArticleTest) [test/models/article_test.rb:6]:
Failed assertion, no message given.

1 tests, 1 assertions, 1 failures, 0 errors, 0 skips
```

En la salida, `F` significa un fallo. Puedes ver la traza correspondiente debajo del `1)` mostrando el nombre de la prueba fallida. Las pocas líneas a continuación contienen la traza de la pila seguida por un mensaje que muestra el valor actual y el valor esperado por la afirmación. Los mensajes de afirmación por defecto proveen justo la suficiente información para ayudar a localizar el error. Para hacer el mensaje de afirmación más legible, todas las afirmaciones poseen un  parámetro opcional para mensajes, como se muestra aquí:

```ruby
test "should not save article without title" do
  article = Article.new
  assert_not article.save, "Saved the article without a title"
end
```

Al ejectuar nuevamente esta prueba muestra el mensaje de fallo más amigable:

```bash
  1) Failure:
test_should_not_save_article_without_title(ArticleTest) [test/models/article_test.rb:6]:
Saved the article without a title
```

Ahora para lograr pasar esta prueba podemos añadir un nivel de validación al modelo para el campo _title_.

```ruby
class Article < ActiveRecord::Base
  validates :title, presence: true
end
```
Ahora deberías pasar la prueba. Vamos a verificarlo ejecutándola nuevamente:

```bash
$ bin/rake test test/models/article_test.rb test_should_not_save_article_without_title
.

Finished tests in 0.047721s, 20.9551 tests/s, 20.9551 assertions/s.

1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
```

Ahora, si lo has notado, primero escribimos la prueba que falló por una funcionalidad deseada, luego escribimos algún código que añade la funcionalidad  y finalmente nos aseguramos que pase las pruebas. Este enfoque de desarrollo de software es llamado Desarrollo Conducido por Pruebas y su nombre original en inglés es [_Test-Driven Development_ (TDD)](http://c2.com/cgi/wiki?TestDrivenDevelopment).

#### Como se ve un error

Para ver como un error es reportado, aquí mostramos una prueba que contiene un error:

```ruby
test "should report error" do
  # some_undefined_variable is not defined elsewhere in the test case
  some_undefined_variable
  assert true
end
```

Ahora puedes ver incluso más en la salida de la consola cuando ejecutas las pruebas:

```bash
$ bin/rake test test/models/article_test.rb test_should_report_error
E

Finished tests in 0.030974s, 32.2851 tests/s, 0.0000 assertions/s.

  1) Error:
test_should_report_error(ArticleTest):
NameError: undefined local variable or method `some_undefined_variable' for #<ArticleTest:0x007fe32e24afe0>
    test/models/article_test.rb:10:in `block in <class:ArticleTest>'

1 tests, 0 assertions, 0 failures, 1 errors, 0 skips
```

Mira la 'E' en la salida. Esta donota una prueba con error.

NOTE: La ejecución de cada método de prueba se detiene tan pronto como se encuentra un error o una afirmación fallida, y el conjunto de pruebas continúa con el siguiente método. Todos los métodos son ejecututados en un order aleatorio. La opción
[`config.active_support.test_order` option](http://edgeguides.rubyonrails.org/configuring.html#configuring-active-support)
se puede utilizar para configurar el orden de las pruebas.

Cuando una prueba falla se te presenta la traza que lo indica. Por defecto los filtros de Rails que escriben la traza solo imprimirán las líneas relevantes de tu aplicación. Esto elimina el ruido del framework y te ayuda a enfocarte en el código. Sin embargo hay sitiuaciones en las que tu quieres ver la traza completa. Simplemente configura la variable de entorno `BACKTRACE` para habilitar este comportamiento:

```bash
$ BACKTRACE=1 bin/rake test test/models/article_test.rb
```

Si queremos pasar esta prueba podemos modificarla para utilizar los `assert_raises` como sigue:

```ruby
test "should report error" do
  # some_undefined_variable is not defined elsewhere in the test case
  assert_raises(NameError) do
    some_undefined_variable
  end
end
```

Esta prueba ahora tendría que pasar.

### Afirmaciones Disponibles

De momento, ya tienes una idea de algunas afirmaciones.
Las afirmaciones son las abejas trabajadoras de las pruebas. Ellas son quienes realmente llevan a cabo los controles para asegurarse que las cosas vayan como las hemos planeado.

Hay un montón de diferentes tipos de afirmaciones puedes utilizar que vienen con [`Minitest`] (https://github.com/seattlerb/minitest), la biblioteca de pruebas por defecto utilizada por Rails.

Para obtener una lista de todas las afirmaciones disponibles,  por favor comprueba la [Documentación API de las Mini Pruebas](http://docs.seattlerb.org/minitest/), especificamente [`Minitest::Assertions`](http://docs.seattlerb.org/minitest/Minitest/Assertions.html)

Por la naturaleza modular del framework de pruebas, es posible crear nuestras propias afirmaciones. En realidad, esto es exactamente lo que Rails hace. Incluye algunas afirmaciones especializadas para hacer tu vida más fácil.

NOTE: Crear tus propias afirmaciones es un tópico avanzado que no cubriremos en este tutorial.

### Afirmaciones Específicas de Rails

Rails añade algunas afirmaciones tradicionales por si mismo al `minitest` framework:

| Afirmación                                                                         | Propósito |
| --------------------------------------------------------------------------------- | ------- |
| `assert_difference(expressions, difference = 1, message = nil) {...}`             | Prueba la diferencia numérica entre el valor retornado como resultado de una expresión  y de lo que se evalúa en el bloque cedido.|
| `assert_no_difference(expressions, message = nil, &amp;block)`                    | Afirma que el resultado numérico de la expresión evaluada no ha cambiado antes y después de invocar al bloque que se pasa como parámetro.|
| `assert_recognizes(expected_options, path, extras={}, message=nil)`               | Afirma que el enrutamiento de una ruta dada es manejado correctamente y que las opciones analizadas (recibidas en el hash expected_options hash) coinciden con la ruta. Basicamente, afirma que Rails reconoce la ruta dada por expected_options.|
| `assert_generates(expected_path, options, defaults={}, extras = {}, message=nil)` | Afirmaciones que proveen opciones que pueden ser utilizadas para generar la ruta enviada. Es la inversa de assert_recognizes. Los parámetros extra son utilizados para indicarle a la petición los nombres y valores de los parámetros adicionales de la petición que podrían ser en una query string. El parámetro message te permite especificar un mensaje de error personalizado para los fallos de las afirmaciones.|
| `assert_response(type, message = nil)`                                            | Afirma que la respuesta viene con un código de estado específico. Puedes especificar `:success` para indicar 200-299, `:redirect` para indicar 300-399, `:missing` para indicar 404, o `:error` para asignar el rango 500-599. También puedes enviar un número de estado explícito o sus símbolos equivalentes. Para más información, ver [lista de los códigos de estado](http://rubydoc.info/github/rack/rack/master/Rack/Utils#HTTP_STATUS_CODES-constant) y como funciona el  [mapeo](http://rubydoc.info/github/rack/rack/master/Rack/Utils#SYMBOL_TO_STATUS_CODE-constant).|
| `assert_redirected_to(options = {}, message=nil)`                                 | Afirmación que hace coincidir las opciones de redireccionamiento enviadas con la redirección llamada en la última acción. Esta coincidencia puede ser parcial, tal como `assert_redirected_to(controller: "weblog")` también concuerda con la redirección de `redirect_to(controller: "weblog", action: "show")`, etc. También puedes pasar una ruta nombrada tal como `assert_redirected_to root_path` y objetos Active Record tal como `assert_redirected_to @article`.|
| `assert_template(expected = nil, message=nil)`                                    | Afirma que la petición fue representada con el fichero de plantilla apropiado.|

Verás el uso the algunas de estas afirmaciones en el próximo capítulo.

### Una Breve Nota sobre el Minitest

Todos las afirmaciones básicas tal como `assert_equal` definidas en `Minitest::Assertions` están solo disponibles en las clases que estamos utilizando en nuestro propio caso de prueba. En realidad, Rails provee las siguientes clases para que puedeas extenderlas:

* `ActiveSupport::TestCase`
* `ActionController::TestCase`
* `ActionMailer::TestCase`
* `ActionView::TestCase`
* `ActionDispatch::IntegrationTest`
* `ActiveJob::TestCase`

Cada una de estas clases incluye `Minitest::Assertions`, permitiéndonos utilizar todas las afirmaciones básicas en nuestras pruebas.

NOTE: Para más información sobre `Minitest`, consultar a  [Minitest](http://ruby-doc.org/stdlib-2.1.0/libdoc/minitest/rdoc/MiniTest.html)

Pruebas Funcionales para los Controladores
------------------------------------------

En Rails, testear la variedad de acciones de un controlador es una forma de escribir pruebas funcionales.
Recuerda que tus controladores manejan las peticiones web que llegan a tu aplicación y finalmente responden con una vista representada. Cuando escribes pruebas funcionales, estás probando como tus actions manejan las peticiones y los resultados esperados, o responden en algunos casos una vista HTML.

### Que debes Incluir en tus Pruebas Funcionales

Deberías incluir en tu prueba cosas tales como:

* ¿Fue la petición web exitosa?
* ¿Fue el usuario redirigido a la página correcta?
* ¿Fue el usuario existosamente autenticado?
* ¿Fue el objeto correctamente guardado en la plantilla de respuesta?
* ¿Fue mostrado el mensaje apropiado al usuario en la vista?

Ahora que hemos utilizado el andamiaje Rails para generar nuestro recurso `Article`, este ya ha creado el código del controlador y de las pruebas. Puedes echar un vistazo al fichero `articles_controller_test.rb` en el directorio `test/controllers`.

Déjame llevarte a través de una prueba, `test_should_get_index` del fichero `articles_controller_test.rb`.

```ruby
class ArticlesControllerTest < ActionController::TestCase
  test "should get index" do
    get :index
    assert_response :success
    assert_not_nil assigns(:articles)
  end
end
```

En la prueba `test_should_get_index`, Rails simula una llamada en la acción `index`, asegurándose que la petición fue existosa y también asegurando que esta asigna una variable de instancia `articles` válida.

El método `get` dispara la petición web y rellena los resultados en la respuesta. Acepta 4 argumentos:

* La acción del controlador que tu estás pidiendo.
  Esta puede ser en la forma de una cadena o un símbolo.

* `params`: opción con un hash de parámetros de petición para pasar dentro de la acción (por ejemplo una query string de parámetros o variables del article).

* `session`: opción con un hash de variables de sesión para pasar a lo largo de la petición.

* `flash`: opción con un hash de valores flash.

Todas los argumentos palabras clave son opcionales.

Ejemplo: Llamando la acción `:show`, pasando como `params` un `id` 12 y configurando un `user_id` 5 en la sesión:

```ruby
get(:show, params: { 'id' => "12" }, session: { 'user_id' => 5 })
```

Otro ejemplo: Llamando la acción `:view`, pasando como `params` un `id` 12, esta vez sin sesión, pero con un mensaje flash.

```ruby
get(:view, params: { 'id' => '12' }, flash: { 'message' => 'booya!' })
```

NOTE: Si tratas de ejecutar la prueba `test_should_create_article` desde `articles_controller_test.rb` este fallará a causa del nivel de validación del modelo recientemente añadido y con razón.

Vamos a modificar la prueba `test_should_create_article` en `articles_controller_test.rb` en consecuencia todas nuestras pruebas pasaran:

```ruby
test "should create article" do
  assert_difference('Article.count') do
    post :create, params: { article: { title: 'Some title' } }
  end

  assert_redirected_to article_path(assigns(:article))
end
```

Ahora tu podrás ejecutar todaslas pruebas y ellas deberían pasar.

### Tipos de Peticiones Disponibles para las Pruebas Funcionales

Si estás familiarizado con el protocolo HTTP, podrías conocer que `get` es un tipo de petición. Hay 6 tipos de peticiones soportadas en las pruebas funcionales de Rails:

* `get`
* `post`
* `patch`
* `put`
* `head`
* `delete`

Todoslos tipos de peticiones tienen métodos equivalentes que puedes utilizar. En una típica aplicación C.R.U.D. estamos usuando `get`, `post`, `put` y `delete` más a menudo.

NOTE: Las pruebas funcionales no verifican si un tipo específico de petición es aceptada por la acción, estamos más preocupados en la respuesta. Existen las pruebas de peticiones para que en este caso de uso se hagan las pruebas con más sentido.

### Pruebas de peticiones XHR (AJAX)

Para probar peticiones AJAX, puedes especificar la opción `xhr: true` para los métodos `get`, `post`,
`patch`, `put`, y `delete`:

```ruby
test "ajax request responds with no layout" do
  get :show, params: { id: articles(:first).id }, xhr: true

  assert_template :index
  assert_template layout: nil
end
```

### Los Cuatro Elementos hash del Apocalipsis

Después de haber hecho y procesado una petición, tendrás 4 objetos Hash listos para utilizar:

* `assigns` - Algunos objetos que son guardados como variables de instancia en acciones para usarlos en las vistas.
* `cookies` - Algunas cookies son creadas.
* `flash` - Algunos objetos residentes en las variables flash.
* `session` - Algunos objetos residentes en las variables de sesión.

Como los objetos se almacenan en un Hash, puedes acceder a los valores referenciando las claves con cadenas de texto. Puedes también referenciarlos con un nombre símbolo, exepto para `assigns`. Por ejemplo:

```ruby
flash["gordon"]               flash[:gordon]
session["shmession"]          session[:shmession]
cookies["are_good_for_u"]     cookies[:are_good_for_u]

# Por razones históricas no puedes usar assigns[:something]:
assigns["something"]          assigns(:something)
```

### Variables de Instancia Disponibles

También tienes acceso a tres variables de instancia en tus pruebas funcionales:

* `@controller` - El controlador que procesa la petición
* `@request` - El objeto petición
* `@response` - El objeto respuesta

### Configurando Cabeceras y variables CGI

Las [Cabeceras HTTP](http://tools.ietf.org/search/rfc2616#section-5.3) y las
[Variables CGI](http://tools.ietf.org/search/rfc3875#section-4.1)
pueden ser directamente configuradas en la variable de instancia `@request`:

```ruby
# configurando una cabecera HTTP
@request.headers["Accept"] = "text/plain, text/html"
get :index # simula la petición con la cabecera personalizada

# configurando una variable CGI
@request.headers["HTTP_REFERER"] = "http://example.com/home"
post :create # simula la petición con una variable de entorno personalizada
```

### Probando Plantillas y Layouts

Eventualmente, es posible que quieras probar si en la vista de una respuesta se pinta un layout específico.

#### Afirmaciones de Plantillas

Si tu quieres asegurarte que la respuesta pinta la plantilla y el layout correcto, puedes utilizar el método `assert_template`:

```ruby
test "index should render correct template and layout" do
  get :index
  assert_template :index
  assert_template layout: "layouts/application"

  # También puedes pasar una expresión regular.
  assert_template layout: /layouts\/application/
end
```

NOTE: No puedes probar una plantilla y un layout al mismo tiempo, con una sola llamada a `assert_template`.

WARNING: Debes incluir el nombre del directorio "layouts" incluso si grabas el fichero layout en este directorio de layouts estandard. Por lo tanto, `assert_template layout: "application"` no funcionará.

#### Afirmaciones para Partials

Si estás pintando algún partial, cuando afirmas el  layout, puedes afirmar el partial al mismo tiempo.

En caso contrario, la afirmación fallará.

¿Recuerdas que añadimos el partial "_form" para la vistas de creación de Articles? Vamos a escribir ahora una afirmación para probar esto en la acción `:new`:

```ruby
test "new should render correct layout" do
  get :new
  assert_template layout: "layouts/application", partial: "_form"
end
```

Esta es la manera correcta de afirmar cuando se pinta un partial con un nombre dado. Como identificado por la clave `:partial` pasado a la llamada `assert_template`.

### Pruebas para mensajes `flash`

Si recuerdas lo que contamos antes de los Cuatro Hashes del Apocalipsis, uno era `flash`.

Queremos añadir un mensaje `flash` a nuestra aplicación blog siempre que alguien cree un artículo existosamente.

Empecemos por añadir esta afirmación a nuestra prueba `test_should_create_article`:

```ruby
test "should create article" do
  assert_difference('Article.count') do
    post :create, params: { article: { title: 'Some title' } }
  end

  assert_redirected_to article_path(assigns(:article))
  assert_equal 'El artículo fue creado existosamente.', flash[:notice]
end
```

Si ejecutas el test ahora, verás un fallo:

```bash
$ bin/rake test test/controllers/articles_controller_test.rb test_should_create_article
Run options: -n test_should_create_article --seed 32266

# Running:

F

Finished in 0.114870s, 8.7055 runs/s, 34.8220 assertions/s.

  1) Failure:
ArticlesControllerTest#test_should_create_article [/Users/zzak/code/bench/sharedapp/test/controllers/articles_controller_test.rb:16]:
--- expected
+++ actual
@@ -1 +1 @@
-"El artículo fue creado existosamente."
+nil

1 runs, 4 assertions, 1 failures, 0 errors, 0 skips
```

Vamos a implementar ahora el mensaje flash en nuestro controlador. Nuestra acción `:create` debería ahora lucir así:

```ruby
def create
  @article = Article.new(article_params)

  if @article.save
    flash[:notice] = 'El artículo fue creado existosamente.'
    redirect_to @article
  else
    render 'new'
  end
end
```

Ahora si ejecturamos nuestra prueba, deberíamos verla pasar correctamente:

```bash
$ bin/rake test test/controllers/articles_controller_test.rb test_should_create_article
Run options: -n test_should_create_article --seed 18981

# Running:

.

Finished in 0.081972s, 12.1993 runs/s, 48.7972 assertions/s.

1 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

### Uniendo las Partes

Hasta este punto probamos el controlador Articles con las acciones `:index` también `:new` y `:create`. ¿Qué pasa con el tratamiento de los datos existentes?

Vamos a escribir una prueba para la acción `:show`:

```ruby
test "should show article" do
  article = articles(:one)
  get :show, params: { id: article.id }
  assert_response :success
end
```

Recuerda nuestra charla anterior acerca de que el método `articles()` de los fixtures nos dará acceso a nuestros fixtures de artículos.

¿Qué hay acerca de la eliminación de los artículos existentes?

```ruby
test "should destroy article" do
  article = articles(:one)
  assert_difference('Article.count', -1) do
    delete :destroy, params: { id: article.id }
  end

  assert_redirected_to articles_path
end
```

También podemos añadir una prueba para actualizar un artículo existente.

```ruby
test "should update article" do
  article = articles(:one)
  patch :update, params: { id: article.id, article: { title: "updated" } }
  assert_redirected_to article_path(assigns(:article))
end
```

Nota que estamos comenzando a ver alguna duplicación en estas tres pruebas, ambas acceden los mismos datos de fixtures de artículo. Podemos no repetirnos a nosotros mismos (D.R.Y.) usando los métodos `setup` y `teardown` que provee `ActiveSupport::Callbacks`.

Nuestra prueba debería ahora lucir como algo parecido a esto, haciendo caso omiso a las otras pruebas que estamos dejando fuera por abreviar:

```ruby
require 'test_helper'

class ArticlesControllerTest < ActionController::TestCase
  # called before every single test
  def setup
    @article = articles(:one)
  end

  # called after every single test
  def teardown
    # when controller is using cache it may be a good idea to reset it afterwards
    Rails.cache.clear
  end

  test "should show article" do
    # Reuse the @article instance variable from setup
    get :show, params: { id: @article.id }
    assert_response :success
  end

  test "should destroy article" do
    assert_difference('Article.count', -1) do
      delete :destroy, params: { id: @article.id }
    end

    assert_redirected_to articles_path
  end

  test "should update article" do
    patch :update, params: { id: @article.id, article: { title: "updated" } }
    assert_redirected_to article_path(assigns(:article))
  end
end
```

Parecida a otras devoluciones de llamada en Rails, los métodos `setup` y `teardown` pueden también ser usados pasándo un bloque, lambda, o nombre de método como un símbolo a llamar.

### Ayudantes (helpers) para Pruebas

Para evitar la duplicación de código, puedes añadir tu propios ayudantes (helpers) de pruebas.
El sign in helper  puede ser un buen ejemplo:

```ruby
test/test_helper.rb

module SignInHelper
  def sign_in(user)
    session[:user_id] = user.id
  end
end

class ActionController::TestCase
  include SignInHelper
end
```

```ruby
require 'test_helper'

class ProfileControllerTest < ActionController::TestCase

  test "should show profile" do
    # helper is now reusable from any controller test case
    sign_in users(:david)

    get :show
    assert_response :success
    assert_equal users(:david), assigns(:user)
  end
end
```

Pruebas de Rutas
----------------

Como todas las cosas en tu aplicación Rails, está recomendado que pruebes las rutas. Debajo hay ejemplos de pruebas para la rutas por defecto de las acciones `show` y `create` del controlador `Articles` de arriba y debería lucir como:

```ruby
class ArticleRoutesTest < ActionController::TestCase
  test "should route to article" do
    assert_routing '/articles/1', { controller: "articles", action: "show", id: "1" }
  end

  test "should route to create article" do
    assert_routing({ method: 'post', path: '/articles' }, { controller: "articles", action: "create" })
  end
end
```

He añadido este fichero aquí  `test/controllers/articles_routes_test.rb` y si ejecutamos la prueba deberíamos ver:

```bash
$ bin/rake test test/controllers/articles_routes_test.rb

# Running:

..

Finished in 0.069381s, 28.8263 runs/s, 86.4790 assertions/s.

2 runs, 6 assertions, 0 failures, 0 errors, 0 skips
```

Para más información sobre las afirmaciones de enrutamiento disponibles en Rails, ver la documentación API para [`ActionDispatch::Assertions::RoutingAssertions`](http://api.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html).

Pruebas de Vistas
-----------------

Probar la respuesta a tu petición afirmando la presencia de claves de elementos HTML y sus contenidos es la manera común para probar las vistas de tu aplicación. El método `assert_select` te permite consultar los elementos HTML de la respuesta utilizando una sintaxis simple pero potente.

Hay 2 formularios de `assert_select`:

`assert_select(selector, [equality], [message])` asegura que se cumple la condición de igualdad en los elementos seleccionados a través del selector. El selector puede ser una expresión de selección CSS (una cadena de texto) o una expresión con valores de sustitución.

`assert_select(element, selector, [equality], [message])` asegura que se cumple la condición de igualdad en todos los elementos seleccionados a través del selector que empieza por el _elemento_ (instancia de `Nokogiri::XML::Node` o `Nokogiri::XML::NodeSet`) y sus descendientes.

Por ejemplo, puedes verificar los contenidos en el elemento title en tu respuesta con:

```ruby
assert_select 'title', "Bienvenido a la las Guías de Ruby on Rails"
```

También puedes utilizar bloques `assert_select` para comprobaciones en profundidad.

En el siguiente ejemplo, el `assert_select` interior de `li.menu_item` se ejecuta dentro de la colección de elementos seleccionados por el bloque exterior:

```ruby
assert_select 'ul.navigation' do
  assert_select 'li.menu_item'
end
```

Una colección de elementos seleccionados podrá ser iterada de manera que a través de `assert_select` puede ser llamada en forma separada.

Por ejemplo si la respuesta contiene 2 listados ordenados, cada uno con cuatro elementos anidados, entonces las siguientes pruebas pasarán.

```ruby
assert_select "ol" do |elements|
  elements.each do |element|
    assert_select element, "li", 4
  end
end

assert_select "ol" do
  assert_select "li", 8
end
```

Esta afirmación es bastante potente. Para más usos avanzados, consultar esta [documentación](http://www.rubydoc.info/github/rails/rails-dom-testing).

#### Afirmaciones Adicionales Basadas en Vistas

Hay más afirmaciones que pueden ser utilizadas para probar vistas:

| Afirmación                                                 | Propósito |
| --------------------------------------------------------- | ------- |
| `assert_select_email`                                     | Te permite hacer afirmaciones sobre el cuerpo de un e-mail. |
| `assert_select_encoded`                                   | Te permite hacer afirmaciones sobre el HTML codificado. Hace esto descodificando el contenido de cada elemento y luego llamando al bloque con todos los elementos codificados.|
| `css_select(selector)` o `css_select(element, selector)` | Retorna un array de todos los elementos seleccionados por el _selector_. En la segunda variante primero busca la correspondencia con el _element_ base y trata de buscarla con la expresión _selector_ en cualquiera de sus hijos. Si no hay coincidencias ambas retornan un array vacío.|

Aquí un ejemplo del uso de `assert_select_email`:

```ruby
assert_select_email do
  assert_select 'small', 'Please click the "Unsubscribe" link if you want to opt-out.'
end
```

Pruebas de Helpers
------------------

Con el fin de probar helpers, todo lo que necesitas hacer es comprobar que la salida del método helper coincida con lo que esperas. Las pruebas relacionadas con los helpers están localizadas en el directorio `test/helpers`.

Una prueba de helper se parece a:

```ruby
require 'test_helper'

class UserHelperTest < ActionView::TestCase
end
```

Un helper es solo un simple módulo donde tu puedes definir métodos que están disponibles dentro de las vistas. Para probar la salida de un método helper, solo debes utilizar un mixin como este:

```ruby
class UserHelperTest < ActionView::TestCase
  include UserHelper

  test "should return the user name" do
    # ...
  end
end
```

Además, dado que la clase de prueba extiende de `ActionView::TestCase`, tienes acceso a los helpers de Rails tal como `link_to` o `pluralize`.

Pruebas de Integración
----------------------

Las pruebas de integración son utilizadas para probar como varias partes de tu aplicación interacturan. Son generalmente utilizadas para probar el flujo de trabajo (work flow) dentro de la aplicación.

Para crear una prueba de integración de Rails, utilizamos el directorio 'test/integration' de la aplicación. Rails provee un generador para crear un test de integración para tí.

```bash
$ bin/rails generate integration_test user_flows
      exists  test/integration/
      create  test/integration/user_flows_test.rb
```

Aquí como se ve una prueba de integración recién generada:

```ruby
require 'test_helper'

class UserFlowsTest < ActionDispatch::IntegrationTest
  # test "the truth" do
  #   assert true
  # end
end
```

Al heredar de `ActionDispatch::IntegrationTest` viene con algunas ventajas, como hacer disponibles algunos helpers adicionales que puedes utilizar en tus pruebas de integración.

### Helpers Disponibles para Pruebas de Integración

Además de los helpers pruebas estandard, al heredar de `ActionDispatch::IntegrationTest` nuestras pruebas de integración vienen con algunos helpers adicionales disponibles que podemos utilizarlos al escribir las pruebas. Vamos a presentarte brevemente a las tres categorías de helpers de donde puedes elegir.

Para tratar con el runner de pruebas de integración, ver [`ActionDispatch::Integration::Runner`](http://api.rubyonrails.org/classes/ActionDispatch/Integration/Runner.html).

Al realizar peticiones, tendrás [`ActionDispatch::Integration::RequestHelpers`](http://api.rubyonrails.org/classes/ActionDispatch/Integration/RequestHelpers.html) disponible para utilizarlo.

Si quisieras modificar la sesión, o el estado de tu prueba de integración deberías buscar en [`ActionDispatch::Integration::Session`](http://api.rubyonrails.org/classes/ActionDispatch/Integration/Session.html) para conseguir ayuda.

### Implementando una Prueba de Integración

Vamos a añadir una prueba de integración a nuestra aplicación blog. Comenzaremos con un flujo básico de creación de un nuevo artículo del blog, para verificar que todo está funcionando correctamente.

Comenzaremos generando el esqueleto de nuestra prueba de integración.


```bash
$ bin/rails generate integration_test blog_flow
```

Este comando debería haber creado el fichero de pruebas en la posición indicada por nosotros, en la salida del comando anterior deberías ver:

```bash
      invoke  test_unit
      create    test/integration/blog_flow_test.rb
```

Ahora vamos a abrir este fichero y escribir nuestra primera afirmación:

```ruby
require 'test_helper'

class BlogFlowTest < ActionDispatch::IntegrationTest
  test "can see the welcome page" do
    get "/"
    assert_select "h1", "Welcome#index"
  end
end
```

Si recuerdas en la sección anterior "Probando Vistas" hemos explicado los `assert_select` para consultar el resultado HTML de una petición.

Entonces cuando visitamos nuestra ruta raíz, la vista debería pintar `welcome/index.html.erb`. Si es así esta afirmación debería pasar.

#### Creando la Integración de artículos

Como probar nuestra habilidad para crear un artículo nuevo en nuestro blog y ver el artículo resultante.

```ruby
test "can create an article" do
  get "/articles/new"
  assert_response :success
  assert_template "articles/new", partial: "articles/_form"

  post "/articles",
    params: { article: { title: "can create", body: "article successfully." } }
  assert_response :redirect
  follow_redirect!
  assert_response :success
  assert_template "articles/show"
  assert_select "p", "Title:\n  can create"
end
```

Vamos a analizar paso a paso esta prueba para poder entenderla.

Comenzamos llamando la acción `:new` en nuestro controlador Articles. Esta respuesta debería ser existosa, y podemos verificar si se pinta la plantilla correcta incluyendo el partial del formulario.
Después de esto hacemos una petición post a la acción `:create` del controlador Articles.

```ruby
post "/articles",
  params: { article: { title: "can create", body: "article successfully." } }
assert_response :redirect
follow_redirect!
```

Las dos líneas que siguen a la petición son para conducir la redirección que configuramos cuando se crea un nuevo artículo.

NOTE: No te olvides de llamar a `follow_redirect!` si planeas hacer peticiones subsiguientes después de que la redirección se haga.

Finalmente podemos afirmar que nuestra respuesta fue existosa, la plantilla fue pintada, y nuestro nuevo artículo se puede leer en la página.

#### Avanzar un poco más

Hemos sido capaces con éxito de probar un pequeño flujo de trabajo para visitar nuestro blog y crear un nuevo artículo.
Si queremos llevar esto más lejos podríamos añadir pruebas para la creación de comentarios, borrar artículos y editar comentarios. Las pruebas de integración son un gran lugar para experimentar con todos los tipos de casos de uso de nuestra aplicación.

Pruebas de Envío de Correo
--------------------------

Las clases para probar envíos de correo requieren algo de inteligencia específica para hacer el trabajo.

### Mantener la Verificación del Cartero

Las clases mailer - como cualquier otra parte de tu aplicación Rails - deberían ser probada para asegurarse que están funcionado como esperas.

El objetivo de probar tus clases mailer es asegurarte que:

* los correos electrónicos están siendo procesados (creados y enviados)
* el contenido del correo electrónico es correcto (asunto, remitente, cuerpo, etc)
* los correos correctos son envíados en el momento correcto

#### Desde Todas Partes

Hay dos aspectos de probar el mailer, las pruebas unitarias y las pruebas funcionales. En las pruebas unitarias, ejecutas la prueba de un mailer con entradas estrictamente controladas y comparas la salida con un valor conocido (un fixture). La prueba funcional no la hacemos para obtener un amplio detalle de lo que produce el mailer; en su lugar, probamos que nuestros modelos y controladores utilizan el mailer de manera correcta. Son pruebas para demostrar que el mensaje correcto fue enviado en el momento correcto.

### Pruebas Unitarias

Con el fin de probar que tu mailer está funcionando como lo esperas, puedes utilizar una prueba unitaria para comparar el resultado actual del mailer con ejemplos pre-escritos de lo que debería producir.

#### La Venganza de los Fixtures

Con el propósito de realizar pruebas unitarias, los fixtures son utilizados para proveer ejemplos de como las salidas _deberían_ verse. Por ser estos ejemplos de mensajes, y no datos Active Record como los otros fixtures, se mantienen en sus propios subdirectorios aparte de los otros fixtures. El nombre del directorio dentro de `test/fixtures` directamente corresponde al nombre del mailer. Entonces, para un mailer llamado `UserMailer`, los fixtures deberían residir en el directorio `test/fixtures/user_mailer`.

Cuando generas el mailer, el generador crea fixtures de resguardo para cada una de las acciones mailers. Si no has utilizado el generador tendrás que crear estos ficheros tu mismo a mano.

#### El Caso de Uso Básico

Aquí tenemos una prueba unitaria para probar un mailer llamado `UserMailer` cuya acción `invite` es utilizada para enviar una invitación a un amigo. Es una versión adaptada de la prueba base creada por el generador para una acción `invite`.

```ruby
require 'test_helper'

class UserMailerTest < ActionMailer::TestCase
  test "invite" do
    # Enivar el mensaje, luego probar que se envió a la cola
    assert_emails 1 do
      email = UserMailer.create_invite('me@example.com',
                                       'friend@example.com', Time.now).deliver_now
    end

    # Prueba que el cuerpo del mensaje enviado contiene lo que esperamos
    assert_equal ['me@example.com'], email.from
    assert_equal ['friend@example.com'], email.to
    assert_equal 'You have been invited by me@example.com', email.subject
    assert_equal read_fixture('invite').join, email.body.to_s
  end
end
```

En la prueba enviamos el mensaje y almacenamos el objeto retornando la variable `email`. Entonces nos aseguramos que fue enviado (la primera afirmación), luego, en el segundo lote de afirmaciones, nos aseguramos que el email contiene en realidad lo que esperamos. El helper `read_fixture` es utilizado para leer dentro del contenido de este fichero.

Aquí el contenido del fichero `invite`:

```
Hi friend@example.com,

You have been invited.

Cheers!
```

Este es el momento correcto para entender un poco más acerca de la escritura de pruebas para tus mailers. La línea `ActionMailer::Base.delivery_method = :test` de
`config/environments/test.rb` configura el método de entrega para el modo de prueba, entonces el mensaje no será realmente enviado (útil para no enviar correo basura a los usuarios mientras pruebas) pero en su lugar será añadido a un array (`ActionMailer::Base.deliveries`).

NOTE: El array `ActionMailer::Base.deliveries` es solo reinicializado automaticamente en pruebas
`ActionMailer::TestCase`. Si quieres tener una pizarra en blanco para la salida de las pruebas Action
Mailer, la puedes borrar manualmente con:
`ActionMailer::Base.deliveries.clear`

### Pruebas Funcionales

Las pruebas funcionales para mailers envuelven son algo más que comprobar que el cuerpo del mensaje, los destinatarios, etc., son correctos.
En una prueba funcional de correo llamas a los métodos de entrega de correo y pruebas que los mensajes apropiados han sido añadidos a la lista de entrega.
Es bastante seguro asumir que los métodos de entrega hacen su trabajo. Por eso en las pruebas funcionales, estarás probablemente más interesado en verificar la lógica de nogocio de la aplicación, es decir, si la aplicación está enviado mensajes cuando esperas que salgan. Por ejemplo, puedes comprobar que la operación invite friend está enviando un mensaje apropiadamente:

```ruby
require 'test_helper'

class UserControllerTest < ActionController::TestCase
  test "invite friend" do
    assert_difference 'ActionMailer::Base.deliveries.size', +1 do
      post :invite_friend, params: { email: 'friend@example.com' }
    end
    invite_email = ActionMailer::Base.deliveries.last

    assert_equal "You have been invited by me@example.com", invite_email.subject
    assert_equal 'friend@example.com', invite_email.to[0]
    assert_match(/Hi friend@example.com/, invite_email.body.to_s)
  end
end
```

Pruebas de Trabajos
-------------------

Desde que tus trabajos (jobs) pueden ser encolados a diferentes niveles dentro de tu aplicación, necesitarás probar por sí mismas estas dos cosas: el comportamiento cuando son encolados y que otras entidades los encolen corramente.

### Un Caso Básico de Prueba

Por defecto, cuando generas un trabajo, una prueba asociada se generará también bajo el directorio `test/jobs`. Aquí una prueba de ejemplo de un trabajo de facturación:

```ruby
require 'test_helper'

class BillingJobTest < ActiveJob::TestCase
  test 'that account is charged' do
    BillingJob.perform_now(account, product)
    assert account.reload.charged_for?(product)
  end
end
```

Esta es una simple y bonita prueba y solo afirma que el trabajo tiene el funcionamiento que se espera.

Por defecto, `ActiveJob::TestCase` configurará el adpatador del encolado a `:test` de modo que los trabajos se realicen en línea. Esto también asegura que todos los  trabajos previamente realizados y encolados son limpiados antes de que cualquier prueba se ejecute, entonces puedes asumir con seguridad que no hay trabajos que ya han sido  ejecutados en el ámbito de cada prueba.



### Afirmaciones Personalizadas y Pruebas de Trabajos Dentro de Otros Componentes

Active Job viene con un lote de afirmaciones personalizadas que pueden ser utilizadas para disminuir el nivel de los mensajes de salida de las pruebas. Para una lista completa de las afirmaciones disponibles, ver la documentación API para [`ActiveJob::TestHelper`](http://api.rubyonrails.org/classes/ActiveJob/TestHelper.html).

Esta es una buena práctica para asegurarse que tus trabajos son correctamente encolados y realizados cuando se invocan (por ejemplo dentro de los controladores).
Aquí es precisamente donde las afirmaciones personalizadas proveídas por Active Job son muy útiles. Por ejemplo, dentro de un modelo:

```ruby
require 'test_helper'

class ProductTest < ActiveJob::TestCase
  test 'billing job scheduling' do
    assert_enqueued_with(job: BillingJob) do
      product.charge(account)
    end
  end
end
```

Otros Enfoques de Pruebas
-------------------------

La construcción de una prueba basada en un `minitest` no es el único camino para probar aplicaciones Rails. Los desarrolladores Rails han llegado a una gran variedad de enfoques y ayudas para realizar pruebas, incluyendo:

* [NullDB](http://avdi.org/projects/nulldb/), una manera de acelerar las pruebas evitando el uso de las bases de datos.
* [Factory Girl](https://github.com/thoughtbot/factory_girl/tree/master), un reemplazo para los fixtures.
* [Fixture Builder](https://github.com/rdy/fixture_builder), una herramienta que compila factories Ruby dentro de fixtures antes de ejecutar una prueba.
* [MiniTest::Spec Rails](https://github.com/metaskills/minitest-spec-rails), utilizar el  MiniTest::Spec DSL dentro de tus pruebas rails.
* [Shoulda](http://www.thoughtbot.com/projects/shoulda), una extensión de `test/unit` con helpers adicionales, macros, y afirmaciones.
* [RSpec](http://relishapp.com/rspec), un framework para BDD, desarrollo conducido por el comportamiento (behavior-driven development).
* [Capybara](http://jnicklas.github.com/capybara/), framework para pruebas de aceptación de aplicaciones web.
