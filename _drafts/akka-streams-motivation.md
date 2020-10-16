# Akka Streams motivation


## why not using actors for implementing a stream topology?

* buffering

* flow control

* error handling

* the code comes easy to read and mantain.. lot of boilerplate.

## Reactive Streams

Specification for asynchronoous stream processing

Non blocking flow control

Bounded resource constraints (not memory overflow)

Interoperability among systems (libraries/systems)

###

Intended for library developers, not por end-users API

## Backpressure

Upstream can't overwhelm downstream

Downstream requests elements/signal... all of this asynchronously

## Akka Streams

* grouping operations (useful for bulk inserts)

## Patterns

* Grouping

* Disaggregation

* filtering

	* filter

	* collect => based on pattern match, can also apply modifications over the element

	* diverTo => puede enviar un elemento a otro sink/flow [INTERESANTE]. Util cuando queremos logear o enviar un elemento invalido a otro sink luego de filtrar.

* Rate limiting request

	* mapAsync

	* parallelism

mapAsync:

Un error es mappear sobre un método asíncrono. Compila OK. El problema es que emitimos futures downstream, y esos futures posiblemente no hayan completado... entonces, eliminamos el batchpressure, porque emitis elementos como vienen. 

En estos casos es mejor usar mapAsync. mapAsync, gestiona la completitud de los futures y los pasa downstream.

*** mapAsync emite los elementos hacia downstream a medida que llegan, es decir, preserva el orden.

Entonces, los futures pueden completarse en distinto orden, pero mapAsync va a emitir los elementos hacia downstream respetando el orden en el que llegaron los mensajes originalmente.

mapAsyncUnordered:

idem mapAsync, solo que no le interesa el orden en el que llegan los mensajes... ni bien completan los futures, emite downstream (it does not wait for ordering) => Beneficio de performance

* throttling

* concurrency

	* por default un stream sobre un solo actor

	* podemos hacer que un stream por debajo se ejecute en dos actores... entonces introducimos un async boundary. Generalmente esto hace mas lento al stream, pero se gana en concurrencia.

* idle timeouts

Si un componente no recibe elementos en un cierto tiempo, dispara una excepcion... util para liberar recursos.

* watching termination