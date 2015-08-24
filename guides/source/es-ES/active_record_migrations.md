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
[cambiar el esquema de tu base de datos a través del tiempo](http://en.wikipedia.org/wiki/Schema_migration) de una manera consistente y fácil. Ellas utilizan un lenguaje de Definición de Esquemas (DSL) en Ruby por lo que no tienes que escribir SQL a mano, permitiendole al esquema y a los cambios en la base de datos ser independientes.
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

### Foreign Keys

While it's not required you might want to add foreign key constraints to
[guarantee referential integrity](#active-record-and-referential-integrity).

```ruby
add_foreign_key :articles, :authors
```

This adds a new foreign key to the `author_id` column of the `articles`
table. The key references the `id` column of the `authors` table. If the
column names can not be derived from the table names, you can use the
`:column` and `:primary_key` options.

Rails will generate a name for every foreign key starting with
`fk_rails_` followed by 10 random characters.
There is a `:name` option to specify a different name if needed.

NOTE: Active Record only supports single column foreign keys. `execute` and
`structure.sql` are required to use composite foreign keys.

Removing a foreign key is easy as well:

```ruby
# let Active Record figure out the column name
remove_foreign_key :accounts, :branches

# remove foreign key for a specific column
remove_foreign_key :accounts, column: :owner_id

# remove foreign key by name
remove_foreign_key :accounts, name: :special_fk_name
```

### When Helpers aren't Enough

If the helpers provided by Active Record aren't enough you can use the `execute`
method to execute arbitrary SQL:

```ruby
Product.connection.execute('UPDATE `products` SET `price`=`free` WHERE 1')
```

For more details and examples of individual methods, check the API documentation.
In particular the documentation for
[`ActiveRecord::ConnectionAdapters::SchemaStatements`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html)
(which provides the methods available in the `change`, `up` and `down` methods),
[`ActiveRecord::ConnectionAdapters::TableDefinition`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/TableDefinition.html)
(which provides the methods available on the object yielded by `create_table`)
and
[`ActiveRecord::ConnectionAdapters::Table`](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/Table.html)
(which provides the methods available on the object yielded by `change_table`).

### Using the `change` Method

The `change` method is the primary way of writing migrations. It works for the
majority of cases, where Active Record knows how to reverse the migration
automatically. Currently, the `change` method supports only these migration
definitions:

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

`change_table` is also reversible, as long as the block does not call `change`,
`change_default` or `remove`.

If you're going to need to use any other methods, you should use `reversible`
or write the `up` and `down` methods instead of using the `change` method.

### Using `reversible`

Complex migrations may require processing that Active Record doesn't know how
to reverse. You can use `reversible` to specify what to do when running a
migration what else to do when reverting it. For example:

```ruby
class ExampleMigration < ActiveRecord::Migration
  def change
    create_table :distributors do |t|
      t.string :zipcode
    end

    reversible do |dir|
      dir.up do
        # add a CHECK constraint
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

Using `reversible` will ensure that the instructions are executed in the
right order too. If the previous example migration is reverted,
the `down` block will be run after the `home_page_url` column is removed and
right before the table `distributors` is dropped.

Sometimes your migration will do something which is just plain irreversible; for
example, it might destroy some data. In such cases, you can raise
`ActiveRecord::IrreversibleMigration` in your `down` block. If someone tries
to revert your migration, an error message will be displayed saying that it
can't be done.

### Using the `up`/`down` Methods

You can also use the old style of migration using `up` and `down` methods
instead of the `change` method.
The `up` method should describe the transformation you'd like to make to your
schema, and the `down` method of your migration should revert the
transformations done by the `up` method. In other words, the database schema
should be unchanged if you do an `up` followed by a `down`. For example, if you
create a table in the `up` method, you should drop it in the `down` method. It
is wise to reverse the transformations in precisely the reverse order they were
made in the `up` method. The example in the `reversible` section is equivalent to:

```ruby
class ExampleMigration < ActiveRecord::Migration
  def up
    create_table :distributors do |t|
      t.string :zipcode
    end

    # add a CHECK constraint
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

If your migration is irreversible, you should raise
`ActiveRecord::IrreversibleMigration` from your `down` method. If someone tries
to revert your migration, an error message will be displayed saying that it
can't be done.

### Reverting Previous Migrations

You can use Active Record's ability to rollback migrations using the `revert` method:

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

The `revert` method also accepts a block of instructions to reverse.
This could be useful to revert selected parts of previous migrations.
For example, let's imagine that `ExampleMigration` is committed and it
is later decided it would be best to use Active Record validations,
in place of the `CHECK` constraint, to verify the zipcode.

```ruby
class DontUseConstraintForZipcodeValidationMigration < ActiveRecord::Migration
  def change
    revert do
      # copy-pasted code from ExampleMigration
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

      # The rest of the migration was ok
    end
  end
end
```

The same migration could also have been written without using `revert`
but this would have involved a few more steps: reversing the order
of `create_table` and `reversible`, replacing `create_table`
by `drop_table`, and finally replacing `up` by `down` and vice-versa.
This is all taken care of by `revert`.

Running Migrations
------------------

Rails provides a set of Rake tasks to run certain sets of migrations.

The very first migration related Rake task you will use will probably be
`rake db:migrate`. In its most basic form it just runs the `change` or `up`
method for all the migrations that have not yet been run. If there are
no such migrations, it exits. It will run these migrations in order based
on the date of the migration.

Note that running the `db:migrate` task also invokes the `db:schema:dump` task, which
will update your `db/schema.rb` file to match the structure of your database.

If you specify a target version, Active Record will run the required migrations
(change, up, down) until it has reached the specified version. The version
is the numerical prefix on the migration's filename. For example, to migrate
to version 20080906120000 run:

```bash
$ bin/rake db:migrate VERSION=20080906120000
```

If version 20080906120000 is greater than the current version (i.e., it is
migrating upwards), this will run the `change` (or `up`) method
on all migrations up to and
including 20080906120000, and will not execute any later migrations. If
migrating downwards, this will run the `down` method on all the migrations
down to, but not including, 20080906120000.

### Rolling Back

A common task is to rollback the last migration. For example, if you made a
mistake in it and wish to correct it. Rather than tracking down the version
number associated with the previous migration you can run:

```bash
$ bin/rake db:rollback
```

This will rollback the latest migration, either by reverting the `change`
method or by running the `down` method. If you need to undo
several migrations you can provide a `STEP` parameter:

```bash
$ bin/rake db:rollback STEP=3
```

will revert the last 3 migrations.

The `db:migrate:redo` task is a shortcut for doing a rollback and then migrating
back up again. As with the `db:rollback` task, you can use the `STEP` parameter
if you need to go more than one version back, for example:

```bash
$ bin/rake db:migrate:redo STEP=3
```

Neither of these Rake tasks do anything you could not do with `db:migrate`. They
are simply more convenient, since you do not need to explicitly specify the
version to migrate to.

### Setup the Database

The `rake db:setup` task will create the database, load the schema and initialize
it with the seed data.

### Resetting the Database

The `rake db:reset` task will drop the database and set it up again. This is
functionally equivalent to `rake db:drop db:setup`.

NOTE: This is not the same as running all the migrations. It will only use the
contents of the current `schema.rb` file. If a migration can't be rolled back,
`rake db:reset` may not help you. To find out more about dumping the schema see
[Schema Dumping and You](#schema-dumping-and-you) section.

### Running Specific Migrations

If you need to run a specific migration up or down, the `db:migrate:up` and
`db:migrate:down` tasks will do that. Just specify the appropriate version and
the corresponding migration will have its `change`, `up` or `down` method
invoked, for example:

```bash
$ bin/rake db:migrate:up VERSION=20080906120000
```

will run the 20080906120000 migration by running the `change` method (or the
`up` method). This task will
first check whether the migration is already performed and will do nothing if
Active Record believes that it has already been run.

### Running Migrations in Different Environments

By default running `rake db:migrate` will run in the `development` environment.
To run migrations against another environment you can specify it using the
`RAILS_ENV` environment variable while running the command. For example to run
migrations against the `test` environment you could run:

```bash
$ bin/rake db:migrate RAILS_ENV=test
```

### Changing the Output of Running Migrations

By default migrations tell you exactly what they're doing and how long it took.
A migration creating a table and adding an index might produce output like this

```bash
==  CreateProducts: migrating =================================================
-- create_table(:products)
   -> 0.0028s
==  CreateProducts: migrated (0.0028s) ========================================
```

Several methods are provided in migrations that allow you to control all this:

| Method               | Purpose
| -------------------- | -------
| suppress_messages    | Takes a block as an argument and suppresses any output generated by the block.
| say                  | Takes a message argument and outputs it as is. A second boolean argument can be passed to specify whether to indent or not.
| say_with_time        | Outputs text along with how long it took to run its block. If the block returns an integer it assumes it is the number of rows affected.

For example, this migration:

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

generates the following output

```bash
==  CreateProducts: migrating =================================================
-- Created a table
   -> and an index!
-- Waiting for a while
   -> 10.0013s
   -> 250 rows
==  CreateProducts: migrated (10.0054s) =======================================
```

If you want Active Record to not output anything, then running `rake db:migrate
VERBOSE=false` will suppress all output.

Changing Existing Migrations
----------------------------

Occasionally you will make a mistake when writing a migration. If you have
already run the migration then you cannot just edit the migration and run the
migration again: Rails thinks it has already run the migration and so will do
nothing when you run `rake db:migrate`. You must rollback the migration (for
example with `rake db:rollback`), edit your migration and then run
`rake db:migrate` to run the corrected version.

In general, editing existing migrations is not a good idea. You will be
creating extra work for yourself and your co-workers and cause major headaches
if the existing version of the migration has already been run on production
machines. Instead, you should write a new migration that performs the changes
you require. Editing a freshly generated migration that has not yet been
committed to source control (or, more generally, which has not been propagated
beyond your development machine) is relatively harmless.

The `revert` method can be helpful when writing a new migration to undo
previous migrations in whole or in part
(see [Reverting Previous Migrations](#reverting-previous-migrations) above).

Schema Dumping and You
----------------------

### What are Schema Files for?

Migrations, mighty as they may be, are not the authoritative source for your
database schema. That role falls to either `db/schema.rb` or an SQL file which
Active Record generates by examining the database. They are not designed to be
edited, they just represent the current state of the database.

There is no need (and it is error prone) to deploy a new instance of an app by
replaying the entire migration history. It is much simpler and faster to just
load into the database a description of the current schema.

For example, this is how the test database is created: the current development
database is dumped (either to `db/schema.rb` or `db/structure.sql`) and then
loaded into the test database.

Schema files are also useful if you want a quick look at what attributes an
Active Record object has. This information is not in the model's code and is
frequently spread across several migrations, but the information is nicely
summed up in the schema file. The
[annotate_models](https://github.com/ctran/annotate_models) gem automatically
adds and updates comments at the top of each model summarizing the schema if
you desire that functionality.

### Types of Schema Dumps

There are two ways to dump the schema. This is set in `config/application.rb`
by the `config.active_record.schema_format` setting, which may be either `:sql`
or `:ruby`.

If `:ruby` is selected then the schema is stored in `db/schema.rb`. If you look
at this file you'll find that it looks an awful lot like one very big
migration:

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

In many ways this is exactly what it is. This file is created by inspecting the
database and expressing its structure using `create_table`, `add_index`, and so
on. Because this is database-independent, it could be loaded into any database
that Active Record supports. This could be very useful if you were to
distribute an application that is able to run against multiple databases.

There is however a trade-off: `db/schema.rb` cannot express database specific
items such as triggers, or stored procedures. While in a migration you can
execute custom SQL statements, the schema dumper cannot reconstitute those
statements from the database. If you are using features like this, then you
should set the schema format to `:sql`.

Instead of using Active Record's schema dumper, the database's structure will
be dumped using a tool specific to the database (via the `db:structure:dump`
Rake task) into `db/structure.sql`. For example, for PostgreSQL, the `pg_dump`
utility is used. For MySQL, this file will contain the output of
`SHOW CREATE TABLE` for the various tables.

Loading these schemas is simply a question of executing the SQL statements they
contain. By definition, this will create a perfect copy of the database's
structure. Using the `:sql` schema format will, however, prevent loading the
schema into a RDBMS other than the one used to create it.

### Schema Dumps and Source Control

Because schema dumps are the authoritative source for your database schema, it
is strongly recommended that you check them into source control.

`db/schema.rb` contains the current version number of the database. This
ensures conflicts are going to happen in the case of a merge where both
branches touched the schema. When that happens, solve conflicts manually,
keeping the highest version number of the two.

Active Record and Referential Integrity
---------------------------------------

The Active Record way claims that intelligence belongs in your models, not in
the database. As such, features such as triggers or constraints,
which push some of that intelligence back into the database, are not heavily
used.

Validations such as `validates :foreign_key, uniqueness: true` are one way in
which models can enforce data integrity. The `:dependent` option on
associations allows models to automatically destroy child objects when the
parent is destroyed. Like anything which operates at the application level,
these cannot guarantee referential integrity and so some people augment them
with [foreign key constraints](#foreign-keys) in the database.

Although Active Record does not provide all the tools for working directly with
such features, the `execute` method can be used to execute arbitrary SQL.

Migrations and Seed Data
------------------------

Some people use migrations to add data to the database:

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

However, Rails has a 'seeds' feature that should be used for seeding a database
with initial data. It's a really simple feature: just fill up `db/seeds.rb`
with some Ruby code, and run `rake db:seed`:

```ruby
5.times do |i|
  Product.create(name: "Product ##{i}", description: "A product.")
end
```

This is generally a much cleaner way to set up the database of a blank
application.
