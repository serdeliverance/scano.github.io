---
title: Slick Getting Started
date: 2021-03-29 23:00:00
excerpt: 'Getting Started with Slick and Best Practices'
categories:
  - blog
tags:
  - scala
  - slick
header:
  image: '/images/header.jpg'
---


# Intro

In this post, we are going to see an intro to `Slick`, which is a powerfull library for interacting with relational databases in `Scala`.

# Slick

First of all, it is important to mention that `Slick` is not an ORM. It is a lower level API. You can see `Slick` as a wrapper around `JDBC`.

`Slick` is a `FRM` (`Functional Relational Mapper`). It means that it tries to embrace relational database access by providing an API that feels like working with collections and that takes advantages of functional programming concepts, such as composition. Also, `Slick` is asynchronous and provides streaming capabilities.

In the `Slick` world, it is important to know three concepts, which together are known as the `Monadic Trio`: Query, DBAction, Future. Before diving into each of them (excepts `Future`, which is a cross topic that will deserve a dedicated post comming soon), it is important to set up some dependencies.

# Project setup

Let's create a `sbt` project and add the following dependencies to your `build.sbt`

``` scala
libraryDependencies ++= Seq(
  "com.typesafe.slick" %% "slick" % "3.3.2",
  "com.h2database" % "h2" % "1.4.200",
  "ch.qos.logback" % "logback-classic" % "1.2.3"
)
```

# Queries

## Table Queries

# DBAction

## Combining Actions

## Transactions

# Conclusion

In a future article, we are going to talk about `Slick` best practices for real world projects.

###### Notas

* Query, DBIO y Future son mónadas
* Query representa una query. Al ser una mónada, podemos hacer flatMap. Esto nos permite poder partir de una query base (generalmente de una TableQuery) e ir armando la query que necesitamos utilizando las operaciones que nos da la API, la cual, busca que trabajemos las queries de una forma similar a si estuvieramos usando collections.
* cuando invocamos al método `result` de una `Query`, eso nos da el `DBIO`.
* `DBIO` es la descripción de la interacción/acción que queremos performar contra la base de datos. Su principal función es que nos permite podes combinar múltiples acciones y correrlas como una unidad (va de la mano con la idea de transacción).

``` scala
// ejemplo de Slick essentials
val actions: DBIO[Seq[Message]] = (
    messages.schema.create
    andThen
    (messages ++= freshTestData) andThen
    halSays.result
)
```

* `Slick` es asíncrono, por lo cual los valores retornados al performar la query se almacenan en `Future`

## notas Capitulo 2

* una `Query` tiene los siguientes type parameters:

  * M: mixed type. Es el tipo de la tabla. Ej: `MessageTable`. Este tipo determina el tipo del input de la función. En el caso de una typed query, nosotros tenemos, por ejemplo: `MessageTable`

  * U: unpacked type. Este es el tipo de dato del output

  * C: collection type. Es el tipo de la collection que va a recolectar nuestros elementos (en una tabla query suele ser `Seq`)

Recordar, podemos ver a la query como una función: `M => C(U)`, donde `M` es el input (o sea el tipo de la tabla) y `U` es el tipo del elemento, y `C` es el tipo de la collection. Ejemplo:

``` scala
messageTable.map(_.content)       // M: MessageTable, U
```

Ahora tal vez nos haga más sentido ver el tipo de una `TableQuery`:

``` scala
trait TableQuery[T <: Table[_]] extends Query[T, T#TableElementType, Seq] {
  // ...
}
```

El tener estos tres tipos de la query hace que usar combinators como `map` y `filter` sea `Type-Safe` porque ya sabemos el tipo de las columnas de la tabla, entonces el compilador puede alertarnos en caso de estar definiendo mal nuestras queries.

* en `Query`, `flatMap` es usado para hacer joins

* Las actions son una forma de ejecutar múltiples queries
* `DBIOAction` tienen tres tipos:

  * `R`: es el tipo de dato que esperamos obtener de la base de datos (ej: `Person`, `Message`)

  * `S`: indica si el resultado es streameado (`Streaming[T]`) o no (`NoStream`)

  * `E` es el `effect type` y es inferido.

Generalmente, utilizamos un `type alias` llamado `DBIO[T]`, que es un alias de `DBIOAction[T, NoStream, Effect.All]`

* `Effect` es una forma de clasificar el tipo de la action a ejecutar, es decir, el tipo de acción de la query. Los tipos de acciones que tiene `Slick` están asociados a los tipos de acciones que podemos ejecutar sobre una DB y estos son:

  * Read
  * Write
  * Schema
  * Transactional
  * All

Generalmente, `Slick` infiere el `effect` en base a la query.

Ejemplo:

``` scala
message.result      // DBIOAction[Seq[String], NoStream, Effect.Read]
```

`Slick` infiere que el effecto es `Read`

* Executing `Actions`

  * `db.run(...)` ejecuta la accion y retorna el resultado en una collection. A esto se lo conoce también como `materialized` result.

  * `db.stream(...)` ejecuta la acción y retorna el resultado en un `Stream`

* Column Expressions

los métodos filter y map reciben expressions (columns expressions), es decir, expresiones sobre las columnas de nuestras tablas. `Rep` es el tipo que usa `Slick` para representar expresiones y también columnas individuales.

Slick provee métodos sobre `Rep` que nos permite podes crear expresiones

Equality and Inequality methods:

message.filter(_.sender === "Dave").result.statements

messages.filter(_.sender =!= "Dave")

vemos que === y =!= se aplican sobre Rep[String] (MessageTable#Element.sender), creando una expresion (es decir, creando una `Rep`)

* String methods: Slick provee metodos utilies para trabajar con `String`

  ++ : concatenacion

messages.map(m => m.sender ++ "> " ++ m.content).result.statements.mkString

  like:

messages.filter(_.content like "%pod%").result.statements.mkString

  startsWith, length, toUpperCase, trim

* así también, Slick provee metodos para trabajar con Rep de tipos de datos numericos y booleans

* Sort, Take, Drop

`sortBy`

messages.sortBy(_.sender).result

messages.sortBy(_.sender.desc).result   // hace el sort en reverse order

tambien se puede orderan por multiples columnas

messages.sortBy(m => (m.sender, m.content))

`take`
`drop`

combinando take y drop, podemos implementar paginado de queries

messages.sortBy(_.sender).drop(5).take(5)

esto se traduce a:

select "sender", "content", "id"
from "message"
order by "sender"
limit 5 offset 5

* existen los metodos filterOpt y filterIf que hacen un filter si se cumple la condicion (ej: el filterOpt ejecuta el filter si el Optional es Some(algo)). Es decir, hacen el filter de una forma mas dinamica. No se usan muy a menudo, pero son utiles pare reducir el boilerplate en caso de querer hacer un filter condicionalmente

ejemplo filterOpt

def query1(name: Option[String]) =
messages.filter(msg => msg.sender === name)

ejemplo filterIf:

val hideOldMessages = true
// hideOldMessages: Boolean = true
val queryIf = messages.filterIf(hideOldMessages)(_.id > 100L)


## notas Capitulo 3

* insert

tenemos que usar el += method. Esto lo que hace es crear un action directamente

++= es para insertar multiples elementos

``` scala
val insertAction = messages += Message("sergio", "hola como andas?")
```

hacer un insert (dependiendo del motor de db con el cual nos comuniquemos) puede retornar como resultado el numero de rows afectados por la operacion. Nosotros podemos hacer que retorne la primary key de la row recien insertada, utilizando el método `returning` (la manera de especificar esto, la verdad me parecio un poco contra intuitiva,así que no la voy a documentar  ).


Agregar las options `O.AutoInc` hace que `Slick` omita agregar este campo en la sentencia de insert que genere.

``` scala
class MessageTable(tag: Tag) extends Table[Message](tag, "message") {
  def id = column[Long]("id", O.PrimaryKey, O.AutoInc)
  def sender = column[String]("sender")
  def content = column[String]("content")
  def * = (sender, content, id).mapTo[Message]
}
```

Es importante recordar que en nuestra `case class` definimos convenientemente el valor id seteando un default al final (esto es así, un truquillo, para no tener inconvenientes al realizar inserts, dado que este campo va a ser omitido por Slick en ese tipo de sentencias)

``` scala
case class Message(sender: String, content: String, id: Long = 0L)
```

* si al hacer un insert queremos insertar solo un conjunto de campos (supongamos que la tabla tiene como 20 campos donde 16 tienen un valor por defecto y no vale la pena insertar todo), lo que se recomienda es hacer un map indicando los campos a insertar, y hacer el insert sobre dichos campos. Ejemplo:

``` scala
messages.map(_.sender) += "HAL"
```


* batch inserts (o insertando multiples rows): para hacer eso, tenemos que usar el operador ++=

``` scala
val testMessages = Seq(
  Message("Dave", "Hello, HAL. Do you read me, HAL?"),
  Message("HAL",
  "Affirmative, Dave. I read you."),
  Message("Dave", "Open the pod bay doors, HAL."),
  Message("HAL",
  "I'm sorry, Dave. I'm afraid I can't do that.")
)

messages ++= testMessages
```

Es importante saber que Slick sabe como hacer optimizaciones de este tipo de inserts segun el tipo de base con la cual nos estamos comunicando.


## Notas chapter 04

* combinar acciones: andThen (o >>)

combina ambas acciones, retornando el resultado de la segunda accion.

* es importante mencionar que cuando se combinan acciones no estamos ejecutando transacciones. Es importante diferenciar esto del concepto de transaccionalidad

* `DBIO.seq` es parecido a `andThen`, pero con la diferencia que el resultado final también es descartado.

``` scala
val resetSeq: DBIO[Unit] = DBIO.seq(messages.delete, messages.size.result)
```

* `map`: mapea sobre el resultado de la acción

``` scala
val text: DBIO[Option[String]] = messages.map(_.content).result.headOption
```

* `flatMap` da mas control sobre la secuencia de acciones a ejecutar, y decidir qué ejecutar en cada step.

``` scala
val logResetAction: DBIO[Int] =
  delete.flatMap {
    case 0 => DBIO.successful(0)
    case n => insert(n)
}
```

* a pesar de que podemos usar flatMap para secuenciar acciones, generalmente es recomendable es recomendable combinar varias acciones en una con métodos como `DBIO.seq` and `andThen`

* `DBIO.sequence`

* `DBIO.fold` para combinar los resultados de varios `DBIO`

``` scala
val report1: DBIO[Int] = DBIO.successful(41)
val report2: DBIO[Int] = DBIO.successful(1)

val summary: DBIO[Int] = DBIO.fold(reports, default) {
  (total, report) => total + report
}
```

* `DBIO.zip`

``` scala
val zip: DBIO[(Int, Seq[Message])] = messages.size.result zip messages.filter(_.sender === "HAL").result
```

* `cleanUp` corre luego de que finaliza el action y tiene acceso a la info del error como un `Option[Throwable]`:

``` scala
// An action to record problems we encounter:
def log(err: Throwable): DBIO[Int] = messages += Message("SYSTEM", err.getMessage)

// Pretend this is important work which might fail:
val work = DBIO.failed(new RuntimeException("Boom!"))
val action: DBIO[Int] = work.cleanUp {
  case Some(err) => log(err)
  case None => DBIO.successful(0)
}
```

* Logging

podemos configurar el logging de slick en `src/main/resources/logback.yml`

``` scala
<logger name="slick.jdbc.JdbcBackend.statement" level="DEBUG"/>
```

* Transacciones

``` scala
val willRollback = (
  (messages += Message("HAL", "Daisy, Daisy...")) >>
  (messages += Message("Dave", "Please, anything but your singing")) >>
  (messages += Message("HAL", "Give me your answer do"))
).transactionally
```