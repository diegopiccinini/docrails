**NO LEAS ESTE FICHERO EN GITHUB, LAS GUIAS ESTÁN PUBLICADAS EN http://www.guiasrails.es**

Active Record Migraciones
=========================

Las migraciones son la característica de Active Record que permite mantener tu esquema de base de datos a través del tiempo. En lugar de escribir las modificaciones del esquema en SQL puro, las migraciones te permiten utilizar un lenguaje de definción fácil en Ruby DSL, para describir cambios para tus tablas.

Después de leeer esta guía sabrás:

* Los comandos generadores que puedes utilizar para crearlas.
* Los métodos que Active Record provee para manipular tu base de datos.
* Las tareas Rake para manipular las migraciones y el esquema.
* Cómo las migraciones afectan al fichero `schema.rb`.

--------------------------------------------------------------------------------

Resumen de las Migraciones
--------------------------

Las migraciones son el modo  conveniente de
[cambiar el esquema de tu base de datos a través del tiempo](http://en.wikipedia.org/wiki/Schema_migration) de una manera consistente y fácil. Ellas utilizan un lenguaje de Definición de Esquemas (DSL) en Ruby por lo que no tienes que escribir SQL a mano, permitiéndole al esquema y a los cambios en la base de datos ser independientes.
Puedes pensar cada migración como una nueva 'versión' de la base de datos. Un esquema comienza sin nada dentro, y cada migración lo modifica para añadir o remover tablas, columnas, o registros. Active Record conoce como actualizar tu esquema a lo largo de su vida, trayendo desde cualquier punto de su historia hasta la última versión. Active Record actualizará también el fichero
`db/schema.rb` para emparejar la estructura modificada de tu base de datos.

Aquí un ejemplo de migración:

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.text :description

      t.timestamps null: false
    end
  end
end
```

Esta migración añade una tabla llamada `products` con una columna cadena de caractéres llamada `name` y una columna de texto llamada  `description`. Una  columna de clave primaria llamada `id` será también añadida implicitamente, como la clave primaria por defecto para todos los modelos Active
Record. Los macro `timestamps` añaden dos columnas, `created_at` y
`updated_at`. Estas columnas especiales son automaticamemente administradas por Active Record si existen.

Nota que nosotros definimos el cambio que queremos que ocurra a través del tiempo. Antes de que esta migración se ejecute, no existía la tabla, después de la migración la tabla existirá. Active Record sabe como revertir esta migración tan bien. que si ejecutamos la migración hacia atrás, borrará la tabla.
En las bases de datos que soportan transacciones con declaraciones que cambian el esquema, las migraciones son envueltas en una trasacción. Si la base de datos no soporta esto, cuando una migración falla la parte de esta que ha tenido éxito no se vuelve atrás. Tendrás que hacer el retroceso de los cambios que fueron hechos a mano.

NOTE: Hay ciertas consultas que no pueden ejecutarse en una transacción. Si tu adaptador soporta transacciones DSL puedes utilizar `disable_ddl_transaction!` para deshabilitarlas para una simple migración.

Si tu deseas en una migración hacer algo que Active Record no sabe como revertir, puedes utilizar `reversible`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration
  def change
    reversible do |dir|
      change_table :products do |t|
        dir.up   { t.change :price, :string }
        dir.down { t.change :price, :integer }
      end
    end
  end
end
```
Alternativamente, puedes utilizar `up` y `down` en lugar de `change`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration
  def up
    change_table :products do |t|
      t.change :price, :string
    end
  end

  def down
    change_table :products do |t|
      t.change :price, :integer
    end
  end
end
```

Creando una Migración
---------------------

### Creando una Migración Independiente

Las migraciones son grabadas como ficheros  en el directorio `db/migrate`, una por una para cada clase migration. El nombre del fichero es de la forma `YYYYMMDDHHMMSS_create_products.rb`, que es para darle como decir un instante del tiempo UTC para identificar la migración seguida por un guión bajo seguido de un nombre de migración. El nombre de la clase migración (versión CamelCased) debería emparejar con la parte posterior del nonbre del fichero. Por ejemplo en el fichero `20080906120000_create_products.rb` se deberia definir la clase `CreateProducts` y en el fichero
`20080906120001_add_details_to_products.rb` se debería definir
`AddDetailsToProducts`. Rails utiliza esta marca de tiempo para determinar cual migración debería ejecutarse en su correspondiente orden, entonces si tu estás copiando desde otra aplicación o generas el fichero por ti mismo, se consciente de sus posiciones en el orden.

Por supuesto, calcular los instantes no es divertido, entonces Active Record provee un generador que hace esto por ti:

```bash
$ bin/rails generate migration AddPartNumberToProducts
```

Esto creará una migración vacía pero con el nombre apropiado:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration
  def change
  end
end
```

Si el nombre de la migración es de la forma "AddXXXToYYY" o "RemoveXXXFromYYY" y es seguido por una lista de nombres de columnas y tipos, luego una migración conteniendo las declaraciones apropiadas `add_column` y `remove_column` será creada.

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string
```

generará

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration
  def change
    add_column :products, :part_number, :string
  end
end
```

Si tu quieres indexar una nueva columna, puedes hacerlo así:

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string:index
```

generará

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration
  def change
    add_column :products, :part_number, :string
    add_index :products, :part_number
  end
end
```


De modo similar, puedes generar una migración para remover una columna desde la línea de comandos:

```bash
$ bin/rails generate migration RemovePartNumberFromProducts part_number:string
```

genera

```ruby
class RemovePartNumberFromProducts < ActiveRecord::Migration
  def change
    remove_column :products, :part_number, :string
  end
end
```

No estás limitado para añadir una columna generada magicamente. Por ejemplo:

```bash
$ bin/rails generate migration AddDetailsToProducts part_number:string price:decimal
```

genera

```ruby
class AddDetailsToProducts < ActiveRecord::Migration
  def change
    add_column :products, :part_number, :string
    add_column :products, :price, :decimal
  end
end
```

Si el nombre de la migración es de la forma "CreateXXX" y es seguida por una lista de nombres de columnas y tipos entonces  será generada una migración para crear la tabla XXX con las columnas listadas. Por ejemplo:

```bash
$ bin/rails generate migration CreateProducts name:string part_number:string
```

genera

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.string :part_number
    end
  end
end
```

Como siempre, lo que ha sido recientemente generado es un punto de partida. Puedes añadir o remover lo que sea adecuado editando el fichero
`db/migrate/YYYYMMDDHHMMSS_add_details_to_products.rb`.

También, el generador acepta tipos de columna como `references`(también disponible como `belongs_to`). Como ejemplo:

```bash
$ bin/rails generate migration AddUserRefToProducts user:references
```

genera

```ruby
class AddUserRefToProducts < ActiveRecord::Migration
  def change
    add_reference :products, :user, index: true
  end
end
```

Esta migración creará una columna `user_id` y un índice apropiado.

Hay también un generador que produce tablas de intersección, si `JoinTable` es parte del nombre:

```bash
$ bin/rails g migration CreateJoinTableCustomerProduct customer product
```

producirá la siguiente migración:

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration
  def change
    create_join_table :customers, :products do |t|
      # t.index [:customer_id, :product_id]
      # t.index [:product_id, :customer_id]
    end
  end
end
```

### Generadores del Modelo

Los generadores the modelos y de andamiaje crearán las migraciones apropiadas para añadir un nuevo modelo. Estas migraciones ya contendrán las instrucciones para la creación de la correspondiente tabla. Si le dices a Rils que columnas quieres, luego las declaraciones para añadir esas columnas serán también creadas. Por ejemplo, ejecutando:

```bash
$ bin/rails generate model Product name:string description:text
```

creará una migración que se ve de la siguiente manera:

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.text :description

      t.timestamps null: false
    end
  end
end
```

Puedes añadir columnas con cuantos pares nombre/tipo como quieras.


### Modificadores de Paso

Algo comunmente utilizado [tipo de modificador](#columna-modificada) pueden ser pasados directamente en la lína de comandos. Están encerrados por llaves y después del tipo de campo:

Por ejemplo, ejecutando:

```bash
$ bin/rails generate migration AddDetailsToProducts 'price:decimal{5,2}' supplier:references{polymorphic}
```

producirá una migración que luce como esta

```ruby
class AddDetailsToProducts < ActiveRecord::Migration
  def change
    add_column :products, :price, :decimal, precision: 5, scale: 2
    add_reference :products, :supplier, polymorphic: true, index: true
  end
end
```

TIP: Echa un vistazo a las ayudas de los generadores para mayor detalle.

Escribiendo una Migración
-------------------------

¡Una vez que has creado tu migración utilizando uno de los generadores es tiempo de comenzar el trabajo!

### Creando una Tabla

El método `create_table` es uno de los fundamentales, pero la mayoría del tiempo, los generarás utilizando un generador del modelo o de andamiaje. Un típico uso podría ser

```ruby
create_table :products do |t|
  t.string :name
end
```

El cual crea una tabla `products` con una columna llamada `name` (y como se ha desicutido debajo, una columna implicita `id`).

Por defecto, `create_table` creará una clave primaria llamada `id`. Puedes cambiar el nombre de la clave primaria con la opción `:primary_key` (no te olvides de actualizar el modelo correspondiente modelo) o, si no quieres ninguna clave primaria, puedes escribir la opción `id: false`. Si necesitas pasarle a la base de datos opciones específicas puedes escribir un fragmento de SQĹ en la opción `:options`. Por ejemplo:

```ruby
create_table :products, options: "ENGINE=BLACKHOLE" do |t|
  t.string :name, null: false
end
```

esto añadirá `ENGINE=BLACKHOLE` con la declaración SQL utilizada para crear una tabla (cuando utilizamos MySQL, por defecto es `ENGINE=InnoDB`).

### Crear una Tabla de Intersección

El método de migración `create_join_table` crea una tabla de intersección HABTM. Una típica podría ser:

```ruby
create_join_table :products, :categories
```

la cual creará una tabla `categories_products` con dos columnas llamadas
`category_id` y `product_id`. Esas columnas tienen la opción `:null` configurada a `false` por defecto. Esto será sobrescrito especificando la opción `:column_options`.

```ruby
create_join_table :products, :categories, column_options: {null: true}
```

creará `product_id` y `category_id` con la opción `:null` a `true`.

Puedes pasar la opción `:table_name` cuando quieras para adaptar el nombre de la tabla a tus necesidades. Por ejemplo:

```ruby
create_join_table :products, :categories, table_name: :categorization
```

creará una tabla `categorization`.

`create_join_table` también accepta un bloque, el cual pudes utilizar para añadir índices (que no son creados por defecto) o columnas adicionales:

```ruby
create_join_table :products, :categories do |t|
  t.index :product_id
  t.index :category_id
end
```

### Cambiando Tablas

La prima hermana de `create_table` es `change_table`, utilizada para modificar tablas existentes. Esto es utilizado de forma similar a `create_table` pero el objeto que se cede al bloque conoce más trucos. Por ejemplo:

```ruby
change_table :products do |t|
  t.remove :description, :name
  t.string :part_number
  t.index :part_number
  t.rename :upccode, :upc_code
end
```

borra las columnas `description` y `name`, crea una columna string `part_number`
y añade un ínice sobre ella. Finalmente renombra la columna `upccode`.

### Cambiando Columnas

Like the `remove_column` and `add_column` Rails provides the `change_column`
migration method.

```ruby
change_column :products, :part_number, :text
```

Esto cambia la columna `part_number` sobre la tabla products para convertirla en un campo de texto.

Además `change_column`, los métodos `change_column_null` y `change_column_default`
son utilizados especificamente para cambiar el nulo y el valor por defecto de una columna.

```ruby
change_column_null :products, :name, false
change_column_default :products, :approved, false
```

Esto establece el campo `:name` de products a una columna `NOT NULL` y el valor por defecto del campo `:approved` a false.

TIP: A diferencia de `change_column` (y `change_column_default`), `change_column_null` es reversible.

### Modificadores de Columnas

Los modificadores de columnas pueden ser aplicados cuando se crea o se cambia una columna:

* `limit`        Establece el máximo tamaño de un campo `string/text/binary/integer` fields.
* `precision`    Define la precisión de los campos `decimal`, representando el número de dígitos total del número.
* `scale`        Define la escala para los campos `decimal`, representando el numero de dígitos después del punto decimal.
* `polymorphic`  Añade una columna `type` para las asociaciones `belongs_to`.
* `null`         Añade o rechaza valores `NULL` en la columna.
* `default`      Permite establecer un valor por defecto de la columna. Nota que si utilizas un valor dinámico (como una fecha), el valor por defecto solo será calculado la primera vez (ej: en la fecha y hora que la migración es aplicada).
* `index`        Añade un índice para la columna.
* `required`     Añade `required: true` para las asociaciones `belongs_to` y
`null: false` para la columna de la migración.

Algunos adaptadores pueden soportar opciones adicionales; ver el adaptador específico en los documentos API para más información.

### Claves foraneas

Mientras que no es requerida podrías querer añadir una restricción de clave foranea para [garantizar la integridad referencial](#active-record-and-referential-integrity).

```ruby
add_foreign_key :articles, :authors
```

Esto añade una nueva clave foranea a la columna `author_id` en la tabla `articles`. Las clave referencia a la columna `id` de la tabla `authors`. Si los nombres de las columnas no pueden ser derivados de los nombres de las tablas, puedes utilizar ls opciones `:column` y `:primary_key`.

Rails generará un nombre para cada clave foranea empezando por
`fk_rails_` seguido por 10 caracteres aleatóreos.
Hau una opción `:name` para especificar un nombre diferente si fuera necesario.

NOTE: Active Record solo soporta una columna simple como clave foranea. `execute` y
`structure.sql` son requeridas para utilizar claves foraneas compuestas.

Removiendo una clave foranea es tan fácil como:

```ruby
# decirle a Active Record que la averigue por los modelos involucrados
remove_foreign_key :accounts, :branches

# borrar la clave foranea de una columna específica
remove_foreign_key :accounts, column: :owner_id

# borrar la clave foranea por su nombre
remove_foreign_key :accounts, name: :special_fk_name
```

### Cuando los Helpers no son Suficientes

Si los helpers provistos por Active Record no son suficientes puedes utilizar el método `execute` para ejecutar cualquier SQL:

```ruby
Product.connection.execute('UPDATE `products` SET `price`=`free` WHERE 1')
```

Para más detalles y ejemplos de métodos individuales, leer la documentación API.
En particular la documentación para
[`ActiveRecord::ConnectionAdapters::SchemaStatements`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html)
(la cual provee los métodos disponibles en los métodos `change`, `up` y `down`),
[`ActiveRecord::ConnectionAdapters::TableDefinition`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/TableDefinition.html)
(el cual provee los métodos disponibles en el objeto cedido por `create_table`)
y
[`ActiveRecord::ConnectionAdapters::Table`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/Table.html)
(el cual provee los métodos disponibles en el objeto cedido por `change_table`).

### Utilizando el Método `change`

El método `change` es la principal manera de escribir migraciones. Este funciona para la mayoría de los casos, donde Active Record conoce como revertir la migración automaticamente. Actualmente, el método `change` soporta solo estas definiciones de migración:

* `add_column`
* `add_index`
* `add_reference`
* `add_timestamps`
* `add_foreign_key`
* `create_table`
* `create_join_table`
* `drop_table` (must supply a block)
* `drop_join_table` (must supply a block)
* `remove_timestamps`
* `rename_column`
* `rename_index`
* `remove_reference`
* `rename_table`

`change_table` es también reversible, siempre y cuando el bloque no llame a `change`,`change_default` o `remove`.

Si necesitarás utilizar algún otro método, deberías usar `reversible` o escribir los métodos `up` y `down` en lugar de utilizar el método `change`.

### Usando `reversible`

Migraciones complejas pueden requerir procesamiento que Active Record no sabe como revertir. Puedes utilizar `reversible` para especificar que hacer cuando estás ejecutando una migración y será lo que se haga cuando se revierta. Por ejemplo:

```ruby
class ExampleMigration < ActiveRecord::Migration
  def change
    create_table :distributors do |t|
      t.string :zipcode
    end

    reversible do |dir|
      dir.up do
        # añadir una restricción CHECK
        execute <<-SQL
          ALTER TABLE distributors
            ADD CONSTRAINT zipchk
              CHECK (char_length(zipcode) = 5) NO INHERIT;
        SQL
      end
      dir.down do
        execute <<-SQL
          ALTER TABLE distributors
            DROP CONSTRAINT zipchk
        SQL
      end
    end

    add_column :users, :home_page_url, :string
    rename_column :users, :email, :email_address
  end
end
```

Utilizando `reversible` nos aseguraremos que las instrucciones son ejecutadas en el orden correcto también. Si ejemplo de la anterior migración es revertido, el bloque `down` será ejecutado después de que la columna `home_page_url` sea borrada y justo antes de que la tabla `distributors` se borre.

Algunas veces tu migración hará algo que es simplemente irreversible, por ejemplo, pude borrar algunos datos, En tales casos, puedes lanzar una excepción
`ActiveRecord::IrreversibleMigration` en tu bloque `down`. Si alguien intenta revertir tu migración, se le mostrará un mensaje de error diciendo que no se pude hacer.

### Utilizando los Métodos `up`/`down`

También puedes utilizar los viejos métodos `up` y `down` en lugar del método `change`.
El método `up` debería describir la transformación que deseas hacer a tu esquema, y el método `down` de tu migración debería revertir las transformaciones hechas por el método `up`. En otras palabras, el esquema de la base de datos permanecerá sin cambios si ejecutas un `up` seguido por un `down`. Por ejemplo, si creas una tabla en el método `up`, deberías borrarla en el método `down`. Esto es para revertir las transformaciones precisamente en el orden reverso en el que fueron hechas en el método `up`. El ejemplo en la sección `reversible` es equivalente a:

```ruby
class ExampleMigration < ActiveRecord::Migration
  def up
    create_table :distributors do |t|
      t.string :zipcode
    end

    # añadir una restricción CHECK
    execute <<-SQL
      ALTER TABLE distributors
        ADD CONSTRAINT zipchk
        CHECK (char_length(zipcode) = 5);
    SQL

    add_column :users, :home_page_url, :string
    rename_column :users, :email, :email_address
  end

  def down
    rename_column :users, :email_address, :email
    remove_column :users, :home_page_url

    execute <<-SQL
      ALTER TABLE distributors
        DROP CONSTRAINT zipchk
    SQL

    drop_table :distributors
  end
end
```

Si tu migración es irreversible, deberías lanzar una
`ActiveRecord::IrreversibleMigration` desde tu método `down`. Si alguién trata de revertir la mitración, un mensaje de error será mostrado diciendo que esto no se puede hacer.

### Revertir Migraciones Anteriores

Puedes utilizar las posibilidades de Active Record para revertir migracciones utilizando el método `revert`:

```ruby
require_relative '2012121212_example_migration'

class FixupExampleMigration < ActiveRecord::Migration
  def change
    revert ExampleMigration

    create_table(:apples) do |t|
      t.string :variety
    end
  end
end
```

El método `revert` también acepta un bloque de instrucciones para revertir.
Este puede ser utilizado para revertir partes seleccionadas de las seleccionadas migraciones. Por ejemplo, vamos a imaginar que la `ExampleMigration` es cometida y más tarde se decide que lo mejor sería utilizar validaciones Active Record, en lugar de la restricción `CHECK`, para verificar el código postal (zipcode).

```ruby
class DontUseConstraintForZipcodeValidationMigration < ActiveRecord::Migration
  def change
    revert do
      # código cortado y pedado de ExampleMigration
      reversible do |dir|
        dir.up do
          # add a CHECK constraint
          execute <<-SQL
            ALTER TABLE distributors
              ADD CONSTRAINT zipchk
                CHECK (char_length(zipcode) = 5);
          SQL
        end
        dir.down do
          execute <<-SQL
            ALTER TABLE distributors
              DROP CONSTRAINT zipchk
          SQL
        end
      end

      # El resto de la migración fue bien
    end
  end
end
```

La misma migración podría también haberse escrito sin utilizar `revert` pero esto podría haber llevado unos pocos pasos más: revertiendo el orden de `create_table` y `reversible`, reemplazando `create_table` por `drop_table`, y finalmente reemplazando `up` por `down` y vice-versa.
Todo esto está a cargo de `revert`.

Ejecutando Migraciones
----------------------

Rails provee un conjunto de tareas Rake para ejecutar cierto conjunto de migraciones.

La primera tarea de migración que utilizarás será probablemente
`rake db:migrate`. En su forma más básica ejecuta un método `change` o `up`
para todas las migraciones que aún no se han ejecutado. Si no hay tales migraciones, finaliza. Ejecutará esas migraciones en orden basado en la fecha de creación de cada migración.

Nota que ejecutar la tarea `db:migrate` también se invoca la tarea `db:schema:dump`, el cual actualizará tu fichero `db/schema.rb` para emparejarlo a la estructura de tu base de datos.

Si especificas una versión de destino, Active Record ejecturará las migraciones requeridas (change, up, down) hasta alcanzar la versión específica. La versión es el prefijo numérico en el nombre de fichero de la migración. Por ejemplo, para migrar a la versión 20080906120000 ejectuta:

```bash
$ bin/rake db:migrate VERSION=20080906120000
```

Si la versión 20080906120000 es mayor que la versión actual, y se está migrando hacia arriba, se ejectutarán el método `change` (o `up`) en todas las migraciones hacia arriba e incluirá la 20080906120000, y no ejecutará ninguna migración posterior. Si se está migrando hacia abajo, esto ejecutará los metodos `down` de todas las migraciones hacia abajo, paro no incluirá la 20080906120000.

### Deshaciendo Migraciones

Una tarea común es deshacer la última migración. Por ejemplo, si cometes un error en esta y quieres corregirlo. En lugar de rastrear la versión anteior asociada puedes ejecutar:

```bash
$ bin/rake db:rollback
```

Esto ejutará hacia atrás la última migración, ya sea por revertir el método `change` o por ejecución del método `down`. Si necesitas deshacer varias migraciones puedes proveer un parámetro `STEP`:

```bash
$ bin/rake db:rollback STEP=3
```

revertirá las últimas 3 migraciones.

La tarea `db:migrate:redo` es un atajo para deshacer y luego migrar hacia arriba otra vez. Tanto con la tarea `db:rollback`, puedes utilizar el parámetro `STEP`

Si necesitas más retroceder más de una versión, por ejemplo:

```bash
$ bin/rake db:migrate:redo STEP=3
```

Ningna de estas tareas Rake hace algo que no se pueda hacer con `db:migrate`. Son simplemente más convenientes, por que no necesitas escribir la versión específica desde la cual hay deshacer y volver a ejecutar las migraciones.

### Configurar la Base de Datos

La tarea `rake db:setup` creará la base de datos, carga el esquema los datos iniciales.

### Recomponer la Base de Datos

La tarea `rake db:reset` borrará la base de datos y la configurá nuevamente. Esto es funcionalmente equivalente a `rake db:drop db:setup`.

NOTE: Esto no es lo mismo que ejecutar todas las migraciones. Esto utilizará unicamente el fichero `schema.rb` actual. Si una migración no puede ser revertida, `rake db:reset` no te podrá ayudar. Para descubrir más acerca del volcado del esquema ver la sección [El Volcado del Esquema y Tú](#schema-dumping-and-you).

### Ejecutar Migraciones Específicas

Si necesitas ejecutar una migración específica hacia arriba o abajo, las tareas `db:migrate:up` y `db:migrate:down` harán esto. Sólo especifica la versión adecuada y la migración correspondiente tendrá su método `change`, `up` o `down` invocado, por ejemplo:

```bash
$ bin/rake db:migrate:up VERSION=20080906120000
```

ejecutara la migración 20080906120000 por ejecución del método `change` (o del método `up`). Esta tarea primero comprobará si la migración está ya realizada y no hará nada si Active Record cree que esta ya se ha ejecutado antes.

### Ejecutando Migraciones en Diferentes Entornos

Por defecto cuando ejecutamos `rake db:migrate` se ejecutará en el entorno de desarrollo  o `development`.
Si quieres ejecutar las migraciones otra vez en otro entorno, puedes especificarlo utilizando la variable de entorno `RAILS_ENV` cuando ejecutas el comando. Por ejemplo para ejecutar las migraciones nuevamente en el entorno de pruebas, `test`, podrías ejecutar:

```bash
$ bin/rake db:migrate RAILS_ENV=test
```

### Cambiando la Salida de la Ejecución de Migraciones

Por defecto las migraciones te indican exactamente que han hecho y cuanto tiempo le ha llevado.
Una migración que crea una tabla y añade un índice puede producir una salida como esta:

```bash
==  CreateProducts: migrating =================================================
-- create_table(:products)
   -> 0.0028s
==  CreateProducts: migrated (0.0028s) ========================================
```

Varios métodos son provistos en las migraciones que te permiten controlar todo esto:

| Método               | Propósito
| -------------------- | -------
| suppress_messages    | Toma un bloque como argumento y suprime cualquier salida generada por el bloque.
| say                  | Toma como argumento un mensaje y emite tal cual es. Se le puede pasar un segundo argumento booleano para especificar si se va a tabular  o no.
| say_with_time        | Salida de texto junto con el tiempo que tomó para ejecutar el bloque correspondiente. Si el bloque devuelve un entero que asume que es el número de filas afectadas.

Por ejemplo, esta migración:

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    suppress_messages do
      create_table :products do |t|
        t.string :name
        t.text :description
        t.timestamps null: false
      end
    end

    say "Created a table"

    suppress_messages {add_index :products, :name}
    say "and an index!", true

    say_with_time 'Waiting for a while' do
      sleep 10
      250
    end
  end
end
```

genera la siguiente salida

```bash
==  CreateProducts: migrating =================================================
-- Created a table
   -> and an index!
-- Waiting for a while
   -> 10.0013s
   -> 250 rows
==  CreateProducts: migrated (10.0054s) =======================================
```

Si quieres que Active Record no muestre ninguna salida, ejecutando `rake db:migrate VERBOSE=false` se suprimirán toda las salidas.

Cambiando las Migraciones Existentes
------------------------------------

Ocasionalmente podemos cometer un error cuando escribimos una migración. Si has ejecutado ya la migración entonces no puedes editarla y ejecutarla otra vez: Rails pensará que ya se ha ejecutado la migración y no hará nada cuando ejecutes otra vez `rake db:migrate`. Por eso debes revertir la migración (por ejemplo con `rake db:rollback`), edita la migración y luego escribe `rake db:migrate` para ejecutar la versión corregida.

En general, editar una migración existente no es una buena idea. Crearás trabajo extra para ti mismo y tus colaboradores y causará mayores dolores de cabeza si la versión existente de una migración ha sido ya ejecutada en los servidores de producción. En su lugar, deberías escribir una nueva migración que realice los cambios que requieres. Editar una fresca migración recién generada que no ha sido cometida en la base de datos, controlano el código fuente (que además en general no se ha propagado más allá de tu máquina de desarrollo) es reletivamente inofensivo.

El método `revert` puede ser de mucha ayuda cuando escribes una nueva migración para deshacer migraciones anteriores en conjunto o en parte
(ver [Revertir Migraciones Anteriores](#reverting-previous-migrations) arriba).

Volcando el Esquema y Tú
------------------------

### ¿Para qué son los Ficheros de Esquema?

Las migraciones, con lo poderosas que pueden ser, no son la fuente fiel de tu esquema de base de datos. Este rol cae sobre ya sea sobre el fichero `db/schema.rb` o un fichero SQL generado por Active Record al examinar la base de datos. Estos no están diseñados para ser editados, representan solo el estado actual del esquema de la base de datos.

No es necesario (y eso es un error repetido) despelegar una nueva instancia de una aplicación a través de volver a ejecutar todas el historial de migraciones. Es mucho más simple y rápido gargar en la base de datos la descripción del esquema actual.

Por ejemplo, así es como la base de datos de test es creada: la actual base de datos de desarrollo es volcada (ya sea desde `db/schema.rb` o desde `db/structure.sql`) y luego grabada en la base de datos de test.

Los fichero de esquema es también útil si quieres echar un vistazo a los atributos que un objeto Active Record tiene. Esta información no está en el código de los modelos y es frecuentemente propagada a través de varias migraciones, pero esta información está muy bien resumida en el fichero de esquema.

La gema annotate_models](https://github.com/ctran/annotate_models) automaticamente añade y actualiza comentarios en la parte superior del cada modelo resumiendo el esquema si deseas esa funcionalidad.

### Tipos de Volcado del Esquema

Hay 2 maneras de volcar el esquema. Esto se configura en `config/application.rb`
a través de la directiva `config.active_record.schema_format`, la cual puede ser o bien `:sql` o `:ruby`.

Si está seleccionado `:ruby` entonces el esquema es guardado en `db/schema.rb`. Si miras este fichero encontrarás que luce casi como una gran migración:

```ruby
ActiveRecord::Schema.define(version: 20080906171750) do
  create_table "authors", force: true do |t|
    t.string   "name"
    t.datetime "created_at"
    t.datetime "updated_at"
  end

  create_table "products", force: true do |t|
    t.string   "name"
    t.text "description"
    t.datetime "created_at"
    t.datetime "updated_at"
    t.string "part_number"
  end
end
```

En muchos sentidos esto es exactamente lo que es. Este fichero fue creado inspeccionando la base de datos y expresando su estructura utilizando `create_table`, `add_index`, etc. Porque es independiente de la base de datos, podría ser cargdo en cualquier base de datos que Active Record soporte. Esto podría ser muy útil is tienes que distribuir una aplicación que debe estar disponible para ejecutarse contra multiples bases de datos.

Sin embargo hay algunas funcionalidades que quedan fuera: `db/schema.rb` no puede expresar items específicos de una base de datos como disparadores, o procedimientos almacenados. Mientras que en una migración tu puedes ejecutar definiciones SQL personlizadas, al volcado del esquema no puede reconstituir aquellas definiciones desde la base de datos. Si estás utilizando características como estas, entonces deberías configuar el formato del esquema a `:sql`.

En lugar de utilizar el volcado del esquema de Active Record, la estructura de la base de datos será volcada utilizando una herramienta específica de la base de datos dentro del `db/structure.sql` (via la tarea Rake `db:structure:dump`). Por ejemplo, para PostgreSQL, se utiliza  `pg_dump`. Para MySQL, este fichero contendrá la salida  `SHOW CREATE TABLE` te varias tablas.

Cargar estos esquemas es sencillamente cuestión de ejecutar las definiciones SQL que contienen. Por definción, esto creará una copia perfecta de la estructura de la base de datos. Utilizando el formato del esquema `:sql` sin embargo, vamos a asegurarnos la carga del esquema en otro RDBMS distinto al que fue utilizado para crearlo.

### Volcados del Esquema y Control de la Fuente

Porque los volcados del esquema son la fuente que manda para tu esquema de base de datos, esto más que recomendable que las revises dentro del control de la fuente.

`db/schema.rb` contiene la versión actual de la base de datos. Esto asegura que los conflictos ocurrirán en el caso de que se mezclen dos ramas que han tocado el esquema. Cuando esto ocurre, resuelve los conflictos manualmente, manteniendo el número de versión más alto entre los dos.

Active Record y la Integridad Referencial
-----------------------------------------

El metodología de Active Record dice que la inteligencia pertenece al modelo, no a la base de datos. Así bien, características tales como disparadores o restricciones, las cuales otorgan algo de inteligencia a la base de datos, no son fuertemente utilizadas.

Las validaciones tal como `validates :foreign_key, uniqueness: true` son un modo por el cual los modelos pueden forzar la integridad de los datos. La opción `:dependent` en las asociaciones permite a los modelos destruir los objetos que son hijos cuando el padre es destruido. Como cualquier cosa que opere a nivel de aplicación, eso no garantiza integridad referencial, entonces algunas personas se aseguran esta con [restricciones sobre las claves foraneas](#foreign-keys) en la base de datos.

Sin embargo Active Record no provee todas las herramientas para trabajar directamente con algunas características, el método `execute` puede ser utilizado para ejecutar un SQL arbitrario.

Migarciones y Semillas de Datos
-------------------------------

Algunas personas utilizan las migraciones para añadir datos a la base de datos:

```ruby
class AddInitialProducts < ActiveRecord::Migration
  def up
    5.times do |i|
      Product.create(name: "Product ##{i}", description: "A product.")
    end
  end

  def down
    Product.delete_all
  end
end
```

Sin embargo, Rails tiene una caraterística 'seeds' que puede ser utilizada para crear las semillas en una base de datos con datos iniciales. Esta es una característica realmente simple: solo rellena `db/seeds.rb` con algún codigo Ruby, y ejecuta `rake db:seed`:

```ruby
5.times do |i|
  Product.create(name: "Product ##{i}", description: "A product.")
end
```
Esto es generalmente el camino más limpio para configurar la base de datos de una aplicación en blanco.
