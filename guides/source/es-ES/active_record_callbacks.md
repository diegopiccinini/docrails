**NO LEAS ESTE FICHERO EN GITHUB, LAS GUIAS ESTÁN PUBLICADAS EN http://www.guiasrails.es**

Active Record Callbacks
=======================
Esta guía te enseñará como mantener el ciclo de vida de tus objetos Active Record.

Después de leer esta guía conoceras:

* El ciclo de vida de los objetos Active Record.
* Como crear métodos de retrollamada que respondan a eventos en el ciclo de vida del objeto.
* Como crear clases especiales para encapsular el comportamiento común para tus callbacks.

--------------------------------------------------------------------------------

El Ciclo de Vida del Objeto
---------------------------

Durante la operativa normal de una aplicación Rails, los objetos pueden ser creados, actualizados, y destruídos. Active Records provee disparadores dentro del *ciclo de vida del objeto* para que con ellos puedas controlar tu aplicación y sus datos.

Los Callbacks o retrollamdas, te posibilitan dispararar lógica antes o después de la alteración del estado de un objeto.

Un vistazo a los Callbacks
--------------------------

Los Callbacks son métodos que son llamados en ciertos momentos del ciclo de vida de un objeto. Con los callbacks es posible escribir código que se ejecutará siempre que un objeto Active Record es creado, guardado, actualizado, borrado, validado o cargado desde la base de datos.

### Registro de Callbacks

Con el objeto de utilizar algunos callbacks disponibles, necesitas registrarlos. Puedes implementar los callbacks como cualquier método normal y utilizar un método de clase macro-style para registrarlos como  callbacks:

```ruby
class User < ActiveRecord::Base
  validates :login, :email, presence: true

  before_validation :ensure_login_has_a_value

  protected
    def ensure_login_has_a_value
      if login.nil?
        self.login = email unless email.blank?
      end
    end
end
```

Los métodos de clase macro-style pueden también recibir un bloque. Considera utilizar este estilo si el código dentro de tu bloque está utilizando es tan corto que llevará una sola línea:

```ruby
class User < ActiveRecord::Base
  validates :login, :email, presence: true

  before_create do
    self.name = login.capitalize if name.blank?
  end
end
```

Los callbacks también pueden ser registrados con solo disparar ciertos eventos del ciclo de vida:

```ruby
class User < ActiveRecord::Base
  before_validation :normalize_name, on: :create

  # :on takes an array as well
  after_validation :set_location, on: [ :create, :update ]

  protected
    def normalize_name
      self.name = self.name.downcase.titleize
    end

    def set_location
      self.location = LocationService.query(self)
    end
end
```

Está considerada una buena práctica declarar los métodos callback como privados o protegidos. Si los dejas públicos, pueden ser llamados desde fuera del modelo y violar el principio de encapsular del objecto

Callbacks Disponibles
---------------------

Aquí hay una lista con todos los callbacks Active Record, listadas en el mismo orden en el cual serán llamados durante la ejecución de las respectivas operaciones:

### Creando un Objeto

* `before_validation`
* `after_validation`
* `before_save`
* `around_save`
* `before_create`
* `around_create`
* `after_create`
* `after_save`
* `after_commit/after_rollback`

### Actualizando un Objecto

* `before_validation`
* `after_validation`
* `before_save`
* `around_save`
* `before_update`
* `around_update`
* `after_update`
* `after_save`
* `after_commit/after_rollback`

### Destruyendo un Objecto

* `before_destroy`
* `around_destroy`
* `after_destroy`
* `after_commit/after_rollback`

WARNING. `after_save` se ejectua tanto en la creación como en la actualización, pero siempre _after_ los callbacks más específicos `after_create` y `after_update`, no cambia el orden en el cual las llamadas macro fueron ejecutadas.

### `after_initialize` y `after_find`

El callbak `after_initialize` será llamado siempre que un objeto Active Record es instanciado, tanto por el uso directo de `new` o cuando un objeto es cargado desde la base de datos. Esto puede ser de utilidad cuando se tiene la necesidad de directamente sobrescribir un método Active Record `initialize`.

El callback `after_find` será llamado siempre que Active Record cargue un registro desde la base de datos. `after_find` es llamado antes de `after_initialize` si ambos están definidos.

Los callbacks `after_initialize` y `after_find` no tienen `before_*` contrapartidas, pero ellos pueden ser registrados como cualquier otro callback Active Record.

```ruby
class User < ActiveRecord::Base
  after_initialize do |user|
    puts "You have initialized an object!"
  end

  after_find do |user|
    puts "You have found an object!"
  end
end

>> User.new
You have initialized an object!
=> #<User id: nil>

>> User.first
You have found an object!
You have initialized an object!
=> #<User id: 1>
```

### `after_touch`

El callback `after_touch` será llamado si un objeto Active Record es tocado.

```ruby
class User < ActiveRecord::Base
  after_touch do |user|
    puts "Has tocado un objeto"
  end
end

>> u = User.create(name: 'Kuldeep')
=> #<User id: 1, name: "Kuldeep", created_at: "2013-11-25 12:17:49", updated_at: "2013-11-25 12:17:49">

>> u.touch
Has tocado un objeto
=> true
```

Este puede ser utilizado junto con `belongs_to`:

```ruby
class Employee < ActiveRecord::Base
  belongs_to :company, touch: true
  after_touch do
    puts 'Un empleado fue tocado'
  end
end

class Company < ActiveRecord::Base
  has_many :employees
  after_touch :log_when_employees_or_company_touched

  private
  def log_when_employees_or_company_touched
    puts 'Employee/Company fueron tocados'
  end
end

>> @employee = Employee.last
=> #<Employee id: 1, company_id: 1, created_at: "2013-11-25 17:04:22", updated_at: "2013-11-25 17:05:05">

# triggers @employee.company.touch
>> @employee.touch
Employee/Company fueron tocados
Un empleado fue tocado
=> true
```

Ejecutando Callbacks
--------------------

Los siguientes métodos disparan callbacks:

* `create`
* `create!`
* `decrement!`
* `destroy`
* `destroy!`
* `destroy_all`
* `increment!`
* `save`
* `save!`
* `save(validate: false)`
* `toggle!`
* `update_attribute`
* `update`
* `update!`
* `valid?`

Adicionalmente, el callback `after_find` es disparado por los siguientes métodos de búsqueda:

* `all`
* `first`
* `find`
* `find_by`
* `find_by_*`
* `find_by_*!`
* `find_by_sql`
* `last`

El callback `after_initialize` es lanzado cada vez que un nuevo objeto de la clase es instanciado.

NOTE: Los métodos `find_by_*` y `find_by_*!` son finders dinámicos generados automáticamente para todos los atributos. Aprende más acerca de ellos en la [Sección de finders dinámicos](active_record_querying.html#dynamic-finders)

Saltando Callbacks
------------------

Así como con las validaciones, es también posible saltarse callbacks por el uso de los siguientes métodos:

* `decrement`
* `decrement_counter`
* `delete`
* `delete_all`
* `increment`
* `increment_counter`
* `toggle`
* `touch`
* `update_column`
* `update_columns`
* `update_all`
* `update_counters`

Estos métodos deben ser uttilizados con precaución, sin embargo, porque importantes reglas de negocio y lógica de la aplicación pueden ser mantenidas en los callbacks. Pasar por encima de ellas sin entender las implicaciones potenciales podrían dejar inválidos los datos.

Parando la Ejecución
--------------------

Como comienzas el registrando nuevos callbaks para tus modelos, ellos serán encolados para su ejecución. Esta cola incluirá toda la validación de tu modelo, los callbaks registradas, y la operación en la base de datos que será ejecutada.

La cadena entera de callbaks es envuelta en una transacción. Si algún método callback _before_ retorna exactamente `false` o lanza una excepción, la cadena de ejecución se detiene y un hace ROLLBACK; los callbacks _after_ solo pueden llevar esto a cabo lanzando una exepción.


WARNING. Alguna excepción que no es `ActiveRecord::Rollback` será relanzada por Rails después de la que la cadena de calbacks es detenida. Lanzando una exepción otra como `ActiveRecord::Rollback` puede romper el código de manera que no espere que métodos como `save` y `update_attributes` (los cuales normalmente intentan retornar `true` o `false`) para lanzar una excepción.

Callbacks Relacionados
----------------------

Los Callbacks trabajan a través del modelo de relaciones, y pueden también ser definidos por ellos. Supón un ejemplo donde un usuario tiene muchos artículos. Un artículo de usuario debería ser borrado si el usuario es borrado. Vamos a añadir un callback `after_destroy` al modelo `User` de vía de esta relación con el modelo `Article`:

```ruby
class User < ActiveRecord::Base
  has_many :articles, dependent: :destroy
end

class Article < ActiveRecord::Base
  after_destroy :log_destroy_action

  def log_destroy_action
    puts 'Artículo borrado'
  end
end

>> user = User.first
=> #<User id: 1>
>> user.articles.create!
=> #<Article id: 1, user_id: 1>
>> user.destroy
Artículo borrado
=> #<User id: 1>
```

Callbacks Condicionales
-----------------------

Así como las validaciones, podemos también crear la llamada de un método condicional sobre la satisfacción de un predicado dado. Tamabién podemos hacer esto utilizando las opciones `:if` y `:unless`, las cuales pueden tomar un símbolo, un string, un `Proc` o un `Array`. Puedes utilizar la opción `:if` cuando quieras especificar bajo que condiciones el callback **debería** ser llamado. Si quieres especificar las condiciones bajo las cuales el callback **no debería** ser llamado, podrías utilizar la opción `:unless`.

### Utilizando `:if` y `:unless` con un `Símbolo`

Puedes asociar la opciones `:if` y `:unless` con un símbolo correspondiendo con el nombre del método predicado que será llamado correctamente antes del callback. Cuando usamos la opción `:if`, el  callback no se ejecutará si el método predicado retorna false; cuando utilizamos la opción `:unless`, el callback no será ejecutado si el método predicado retorna true. Esta es la opción más común. Utilizando esta forma de registrarlos es también posible registrar varios predicados diferentes que deberían ser llamados para comprobar si el callback debería ser ejecutado.

```ruby
class Order < ActiveRecord::Base
  before_save :normalize_card_number, if: :paid_with_card?
end
```

### Utilizando `:if` y `:unless` como un String

También puedes utilizar un string que será evaluado que será evaluado utilizando `eval` y por lo tanto necesita contener código Ruby válido. Deberías utilizar esta opción solo cuando el string representa una condición realmente corta:

```ruby
class Order < ActiveRecord::Base
  before_save :normalize_card_number, if: "paid_with_card?"
end
```

### Utilizando `:if` y `:unless` con un `Proc`

Finalmente, esto es posible para asociar `:if` y `:unless` con un objeto `Proc`. Esta opción es la mejor combinación cuando escribimos métodos cortos para validar, normalmente una línea:

```ruby
class Order < ActiveRecord::Base
  before_save :normalize_card_number,
    if: Proc.new { |order| order.paid_with_card? }
end
```

### Condiciones Multiples para Callbacks

Cuando escribimos callbacks condicionales, es también mezclar ambos `:if` y `:unless` en la misma declaración callback:

```ruby
class Comment < ActiveRecord::Base
  after_create :send_email_to_author, if: :author_wants_emails?,
    unless: Proc.new { |comment| comment.article.ignore_comments? }
end
```

Clases Calbacks
----------------

Algunas veces los métodos callback que escribirás serán lo suficientemente útiles como para ser reutilizados por otros modelos. Active Record hace posible crear clases que encapsulen métodos callbacks, entonces se vuelve muy sencillo reutilizarlos.

Aquí un ejemplo donde creamos una clase con un callback `after_destroy` para un modelo `PictureFile`:

```ruby
class PictureFileCallbacks
  def after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
```

Cuando declaramos dentro de una clase, como la de arriba, los métodos callback recibirán el objeto del modelo como un parámetro. Ahora podemos utilizar la clase callback en el modelo:

```ruby
class PictureFile < ActiveRecord::Base
  after_destroy PictureFileCallbacks.new
end
```

Nota que necesitamos instanciar un nuevo objeto `PictureFileCallbacks`, por haber declarado nuetro callback como un método de instancia. Esto es particularmente útil si los callbacks hacen uso del estado del objeto instanciado. Frecuentemente, sin embargo, tendrá más sentido para declarar los callbacks como métodos de clase:

```ruby
class PictureFileCallbacks
  def self.after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
```

Si el método callback es declarado de esta manera, no será necesario instanciar un objeto `PictureFileCallbacks`.

```ruby
class PictureFile < ActiveRecord::Base
  after_destroy PictureFileCallbacks
end
```

Puedes declarar tantos callbacks como quieras dentro de tu clase callback.


Callbacks de Transacciones
--------------------------

Hay dos callbacks adicionales que serán disparados para completar una transacción de base de datos: `after_commit` y `after_rollback`. Estos  callbacks son muy parecidos al callback `after_save` excepto que estos no se ejecutan hasta después de los cambios en la base de datos hayan sido cometidos o revertidos. Son muy útiles cuando tus modelos active record necesitan interactuar con sistemas externos los cuales no son parte de una transacción de la base de datos.

Considera, por ejemplo, los ejemplos previos donde el modelo `PictureFile` necesita borrar un fichero después de que el registro correspondiente es destruído. Si por algo se lanzase una excepción después del callback `after_destroy` y la transacción se revirtiera, el fichero se habrá borrado y dejará el modelo en un estado inconsistente. Por ejemplo, soponiendo que `picture_file_2` en el código de abajo no es válido y el métod `save!` arroja un error.

```ruby
PictureFile.transaction do
  picture_file_1.destroy
  picture_file_2.save!
end
```

Utilizando el callback `after_commit` podemos solucionar este caso.

```ruby
class PictureFile < ActiveRecord::Base
  after_commit :delete_picture_file_from_disk, on: [:destroy]

  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

NOTE: la opción `:on` especifica cuando un será disparado. Si tu no escribes la opción `:on` el callback será disparado en todas las acciones.

WARNING. Los callbacks `after_commit` y `after_rollback` nos garantizan que serán llamados por todos los modelos creados, actualizados, o destuídos, dentro de un bloque de transacción. Si alguna excepción es lanzada dentro de estos callbacks, serán ignorardos entonces no interfieren con otros callbacks. Así como también, si tu coódigo callback podría lanzar una excepción, necesitarás rescatarla y manejarla apropiadamente dentro del callback.
