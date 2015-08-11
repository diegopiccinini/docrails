**NO LEAS ESTE FICHERO EN GITHUB, LAS GUIAS ESTÁN PUBLICADAS EN http://www.guiasrails.es**

Fundamentos de Active Record
============================

Esta guía es una introducción a Active Record, (Registro Activo).

Después de leer esta guía, conocerás:

* Qué son el Mapeo de Objetos Relacionales y Active Record y como se utilizan en Rails.
* Cómo entra Active Record en el paradigma Modelo-Vista-Controlador.
* Cómo se utilizan los modelos Active Record para manipular datos buardados en una base de datos relacional.
* Las convenciones sobre el esquema de nombres en Active Record.
* Los conceptos de migraciones de bases de datos, validaciones y retrollamadas.

--------------------------------------------------------------------------------

¿Que es Active Record?
----------------------

Active Record es la M en [MVC](getting_started.html#the-mvc-architecture) - el modelo - el cual es la capa del sistema responsable de representar los datos y la lógica de nogocio para manipularlos. Active Record facilita la creación y manipulación de objetos de negocio quienes requieren ser almacenados persistentemente en una base de datos. Esta es una implementación del patrón de Active Record el cual en si mismo es una descripción de un sitema de Mapeo de Objetos Relacionales.

### El Patrón Active Record

[Active Record fue descripto por Martin Fowler](http://www.martinfowler.com/eaaCatalog/activeRecord.html)
en su libro  _Patterns of Enterprise Application Architecture_. En
Active Record, los objetos soportan tanto la persistencia y el comportamiento que opera con los datos. Active Record toma la opinion que al asegurar el acceso lógico a los datos como parte del objeto educará a los usuarios de ese objeto sobre como escribir y leer en la base de datos.

### Mapeo de Objetos Relacionales

El Mapeo de Objetos Relacionales (Object-Relational Mapping), comunmente nombrado por sus siglas ORM, es una técnica que conecta la riquesa de los objetos de una aplicación con las tablas de un sistema de base de datos relacional. Utilizando ORM, las propiedades y las relaciones de los objetos en una aplicación pueden ser facilmente guardadas y recuperadas desde la base de datos sin escribir sentencias SQL directamente y con el mínimo código en general de acceso a la base de datos.

### Active Record como un Framework ORM

Active Record brinda varios mecanismos, los más importantes nos la capacidad para:

* Representar modelos y sus datos.
* Representar asociaciones entre esos modelos.
* Representar jerarquías de herencia a través de modelos relacionados.
* Validar modelos antes the que sean guardados o cambiados en la base de datos.
* Mantener las operaciones de la base de datos orientadas a objetos.

Convención sobre Configuración en Active Record
-----------------------------------------------

Cuando escribimos aplicaciones usando otros lenguajes de programacion o frameworks, es necesario escribir una gran cantidad de código de configuración.
Esto es particularmente verdad para los frameworks ORM en general. Sin embargo, si sigues las convenciones adoptadas por Rails, necesitarás escribir muy poca configuración (en algunos casos ninguna) cuando creas modelos Active Record. La idea es que si configuras tus aplicaciones de la misma manera la mayoría de las veces, entonces ese debería ser la manera por defecto. Por consiguiente, podríamos necesitar configuración explícita solo en los casos donde no podemos seguir la convención estandard.

### Convenciones sobre Nombres

Por defecto, Active Record utiliza algunas convenciones para encontrar algunas convenciones para conocer con detalle como el mapeo entre los modelos y las tablas de la base de datos debería crearse. Rails convertirá al plural los nombres de tus clases para encontrar la respectiva tabal en la base de datos. Entonces, para una clase `Book`, deberías tener una tabla de base de datos llamada **books**. Los mecanismos de pluralizar the Rails son muy potentes, tiene la capacidad de pluarizar (y singularizar) ambos en palabras regulares e irregulares. Cuando usamos nombres de clases compuestos de dos o más palabras, el nombre de la clase del modelo debe debería seguir las convenciones Ruby, usando la forma CamelCase, mientras que el nombre de la tabla debe contener las palabras separadas por guiones bajos. Ejemplos:

* Tabla de Base de Datos - Plural con guiones bajos separando las palabras (ej: `book_clubs`).
* Clase del Modelo - Singular con la primera letra de cada palabra en mayúsculas (ej: `BookClub`).

| Modelo / Clase   | Tabla / Esquema |
| ---------------- | --------------- |
| `Article`        | `articles`      |
| `LineItem`       | `line_items`    |
| `Deer`           | `deers`         |
| `Mouse`          | `mice`          |
| `Person`         | `people`        |


### Convenciones del Esquema

Active Record utiliza convenciones de nombres para las columnas en las tablas de la base de datos, dependiendo de los propósitos de esas columnas.

* **Claves foráneas** - Estos camos deberían ser nombrados siguiendo el patrón
  `nombre_de_tabla_en_singular_id` (ej: `item_id`, `order_id`). Estos son los campos que Active Record buscará cuando crees asociaciones entre tus modelos.
* **Claves primarias** - Por defecto, Active Record utilizará una columna entero llamada `id` como la clave primaria de la tabla. Cuando utilizas [Active Record Migrations](migrations.html) para crear tus tablas, esta columna será creada automáticamente.

También hay nombres de columnas opcionales que podrán dar características adicionales a las instancias de Active Record:

* `created_at` - Automáticamente guarda el dia y la hora actual  el momento en que se crea el objeto.
* `updated_at` - Automáticamente guarda el día y la hora actual del momento en que se actualiza un registro.
* `lock_version` - Añade [bloqueo optimista](http://api.rubyonrails.org/classes/ActiveRecord/Locking.html) a un modelo.
* `type` - Especifica el modo de utilizar el modelo [La Herencia Simple de Tables](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#class-ActiveRecord::Base-label-Single+table+inheritance).
* `(association_name)_type` - Graba el tipo para
  [asociaciones polimórficas](association_basics.html#polymorphic-associations).
* `(table_name)_count` - Utilizado para cachear el número de objetos que pertencientes en una asociación. Por ejemplo, una columna `comments_count` en una clase `Articles` que tiene muchas instancias de `Comment` guardará en cache el número de los comentarios existentes de cada artículo.

NOTE: Mientras estos nombres de columnas son opcionales, están en realidad reservados por Active Record. Evita el uso de las palabras reservadas si no quieres funcionalidades extra. Por ejemplo, `type` es una palabra reservada utilizada para definir que una tabla está utilizando  Herencia Simple de Tabla (STI). Si no estás utilizando STI, intenta con una palabra análoga como "context", que puede aún así mantener la descripción de los datos que estás modelando.

Creando Modelos Active Record
-----------------------------

Es muy fácil crear modelos Active Record. Todo lo que necesitas hacer es una subclase de la clase `ActiveRecord::Base` y listo:

```ruby
class Product < ActiveRecord::Base
end
```

Esto creará una clase modelo `Product`, mapeada a la tabla `products` de la base de datos. Para hacer esto también tendrás que tener la posibilidad de mapear columnas de cada fila son los atributos de cada instancia del modelo. Supón que la tabla `products` fue creada utilizando una sentencia SQL como:

```sql
CREATE TABLE products (
   id int(11) NOT NULL auto_increment,
   name varchar(255),
   PRIMARY KEY  (id)
);
```

Siguiendo el esquema de arriba, tendrías la capacidad de escribir código como el que sigue:

```ruby
p = Product.new
p.name = "Some Book"
puts p.name # "Some Book"
```

Sobrescribiendo Las Convensiones de Nombres
-------------------------------------------

¿Que sucede si quieres seguir diferenes convenciones de nombres o necesitas utilizar una aplicación Rails con una base de datos heredada? No hay problema, puedes facilmente sobrescribir las convenciones por defecto.

Utilizando el método `ActiveRecord::Base.table_name=` para especificar el nombre de la tabla que debería ser utilizada:


```ruby
class Product < ActiveRecord::Base
  self.table_name = "PRODUCT"
end
```

Si tu haces esto, tendrás que definir manualmente el nombre de la clase a contiene los fixtures (class_name.yml) utilizando el método  the `set_fixture_class` en la definición de los test.

```ruby
class FunnyJoke < ActiveSupport::TestCase
  set_fixture_class funny_jokes: Joke
  fixtures :funny_jokes
  ...
end
```

También es posible cambiar la columna clave primaria de la tabla con el método `ActiveRecord::Base.primary_key=`:

```ruby
class Product < ActiveRecord::Base
  self.primary_key = "product_id"
end
```

CRUD: Leyendo y Escribiendo Datos
---------------------------------

CRUD es el acronismo para las cuatro verbos que utilizamos para operar con los datos: **C**reate,**R**ead, **U**pdate and **D**elete. Active Record automáticamente crea métodos que permiten a una aplicación leer y manipular los datos guardados dentro de las tablas.

### Create

Los objetos Active Record pueden crearse desde un hash, un bloque o configurar sus atributos manualmente antes de la creación. El método `new` retornará un nuevo objeto mientras `create` retornará retornará el objeto y lo guardará en la base de datos.

Por ejemplo, dado un modelo `User` con los atributos `name` y `occupation`, el método llamado `create` creará y guardará un nuevo registro en la base de datos:

```ruby
user = User.create(name: "David", occupation: "Code Artist")
```

Utilizando el método `new`, un objeto puede ser instanciado sin haber sido guardado:

```ruby
user = User.new
user.name = "David"
user.occupation = "Code Artist"
```

Una llamada a `user.save` guardará el registro en la base de datos.

Finalmente, si un bloque es proveído, ambos `create` y `new` producirá el nuevo objeto inicializado dentro de un bloque:

```ruby
user = User.new do |u|
  u.name = "David"
  u.occupation = "Code Artist"
end
```

### Read

Active Record provee una rica API para acceder a los datos dentro de una base de datos. Debajo hay algunos ejemplos de diferentes métodos provistos por Active Record.

```ruby
# devuelve una colección de usuarios
users = User.all
```

```ruby
# devuelve el primer usuario
user = User.first
```

```ruby
# devuelve el primer usuario llamado David
david = User.find_by(name: 'David')
```

```ruby
# encontrar todos los usuarios llamados David que tienen de ocupación Code Artists y ordenado por created_at en sentido cronologicamente inverso
users = User.where(name: 'David', occupation: 'Code Artist').order('created_at DESC')
```

Puedes aprender más acerca de consultar un modelo Active Record en la guía [Interface de Consultas Active Record](active_record_querying.html).

### Update

Una vez que un objeto Active Record ha sido recuperado, sus atributos pueden ser modificados y volver a ser guardados en la base de datos.

```ruby
user = User.find_by(name: 'David')
user.name = 'Dave'
user.save
```

Un camino corto para este uso es mapear un hash con los nombres de los atributos que se desean modificar y su valor, como por ejemplo:

```ruby
user = User.find_by(name: 'David')
user.update(name: 'Dave')
```

Esta es la forma más poderosa de actualizar varios atributos a la vez. Si, por otro lado, quieres actualizar varios registros a la vez, encontrarás muy útil el método de clase `update_all`:

```ruby
User.update_all "max_login_attempts = 3, must_change_password = 'true'"
```

### Delete

Asimismo, una vez que se recupera el objeto Active Record también puede ser destruído, lo cual lo borrará de la base de datos.

```ruby
user = User.find_by(name: 'David')
user.destroy
```

Validaciones
------------

Active Record te permite validar el estado del modelo antes que se escriba en la base de datos. Hay varios metodos que puedes utilizar para comprobar tu modelo y validar que un atributo no está vacio, es único y no está aún en la base de datos, que siga un formato específico, y muchos más.
La validación es una tarea muy importante a considerar cuando se guardan datos en una base de datos, entonces los métodos `save` y `update` toman esto en cuenta cuando se ejecutan: ellos retornan `false` cuando una validación falla y no mantuvieron ninguna operación en la base de datos. Todos estos tienen una contrapartida bang (esta es , `save!` y `update!`), la cuales son estrictas y arrojan una excepciónn `ActiveRecord::RecordInvalid` si la validación falla.
Un rápido ejemplo para ilustralo:

```ruby
class User < ActiveRecord::Base
  validates :name, presence: true
end

user = User.new
user.save  # => false
user.save! # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

Puedes aprender más acerca de validaciones en la [Guía de Validaciones Active Record](active_record_validations.html).

Callbacks
---------

Las retrollamadas Active Record (o callbacks) te permiten adjuntar ciertos eventos en el ciclo de vida de tus modelos. Esto te faculta a añadir comportamintos a tus modelos de forma transparente en la ejecución cuando estos eventos ocurren. Como cuando tu quieres crear un nuevo registro, actualizarlo, destruírlo, etc. Puedes aprender más de retrollamadas en la [Guía de Active Record Callbacks](active_record_callbacks.html).

Migraciones
-----------

Rails provee un lenguaje de dominio específico para manejar un esquema de base de datos llamado migraciones (migrations). Las migraciones son ficheros guardados que se ejectutan contra cualquier base de datos que Active Record soporte utilizando `rake`. Aquí hay una migración que crea una tabla:

```ruby
class CreatePublications < ActiveRecord::Migration
  def change
    create_table :publications do |t|
      t.string :title
      t.text :description
      t.references :publication_type
      t.integer :publisher_id
      t.string :publisher_type
      t.boolean :single_issue

      t.timestamps null: false
    end
    add_index :publications, :publication_type_id
  end
end
```

Rails mantiene el historial sobre que fichero fue actualizado en la base de datos y provee características para deshacer los cambios. Para realmente crear la tabla, deberías ejecutar `rake db:migrate` y para deshacerlo `rake db:rollback`.

Nota que el código de arriba es database-agnostic: se puede ejectuar en MySQL,
PostgreSQL, Oracle y others. Puedes aprender más acerca de las migraciones en la [Guía de Migraciones Active Record](migrations.html).
