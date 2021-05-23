# Intro

# Notes

Streaming data is a way of transfering and consuming info that regards the foundamentals of how the underlying protocols works (example: TCP). When you have to deal with a big amount of data (such a big file), processing all the data as a whole can't be possible.

Actos can be used to send streamed data, but implementing that is not so straight forward because you have to deal different issues:

* mailboxes and buffer can overflow so you have to control that
* message can be lost and you have to retransmit

# Akka Stream goals

* offer an easy way of formulating stream processing pipelines
* bound resource usage => through back-pressure

# Backpressure

// TODO

# What is Reactive Streams?

Reactive Streams is an initiative that provides an estandard of how to deal with asynchronous boundaries without data lost, buffering or resource exhaustion.

Akka Streams is an API that is compliant with the `Reactive Streams` specifications, but it offers it own API which is nicer for end users. Remeber, Akka Streams is about building/expressing asynchronous pipelines.

# Notes

* materialized value is an auxiliary value that the stream exposes to the external world. You can materialize a value if needed.

* cuando se ejecuta un stream se puede notar que la aplicación no termina, esto es porque el actor system sigue funcionando.

val done: Future[Done] = source.runForeach(i => println(i))

implicit val ec = system.dispatcher
done.onComplete(_ => system.terminate())

* la idea de Akka Streams es que cada componente (source, flow, sink) puedan ser reutilizables. Esto es posible porque cada uno de ellos son en realidad una descripción.

* para hacer un flatten usamos mapConcat (esto es una decision de akka, para no confundir con flatMap, que está mas asociado a monadic compositions y porque los streams no cumplen con las leyes de las monadas)

* backpressure es un protocolo. Es una comunicación desde el Sink hacia el Source.

* backpressure es una un mecanismo de flow-control, donde el consumer envía una notificación al producer, cuantos elementos puede recibir (esto hace que el consumer pueda regular el rate con el que produce elementos)

* Flow>>buffer() es un metodo que te permite especificar el buffer del componente y la estrategia a utilizar en caso de overflow (esta estrategia puede ser un simple drophead o algo que genere backpressure downstream)

* importante: hay buena documentación para crear tus operadores custom en caso que quieras crear tus propios source/flow/sink, etc.. por ejemplo, si te queres comunicar con Redis Streams

* materializar: es el proceso de alocar los recursos que necesita el graph para ser ejecutado, lo que habitualmente se traduce en startup de actores. También puede significar abrir archivos, conexiones de socket, etc.

* en un escenario de backpressure, el producer es signaled con la cantidad de mensajes (demanda) que puede aceptar el suscriber. Para lograr esto, se pueden implementar las siguientes estrategias:

	* que el producer deje de emitir elementos (si es posible)
	* que el producer bufferee elementos hasta que se signalize que el suscriber puede recibir mas elementos (que aumentó la demanda)
	* dropear elementos
	* tear down del stream si no se puede aplicar ninguna de las estrategias anteriores.

* materializacion: es llevada a cabo sincronicamente por el thread usando el Materializer del ActorSystem. El procesamiento del stream (es decir, del run del grafo), es llevado a cabo por los actores que se crearon en dicha materializacion, que correran en los thread pools configurados (por default, corren en el dispatcher del actor system)


* operator fusion: por defecto akka streams fusion los operadores. Esto quiere decir que todos los processing stages son procesados por el mismo actor. Esto tiene las siguientes consecuencias:

	* pasar elementos de un operador a otro es facil porque no tenemos messaging overhead en el actor que hace el procesamiento (como todo pasa en el mismo actor, el tipo no se envia un mensaje asi mismo sino que procesa todo y ya)

	* fused operators no se ejecutan en paralelo (dado que corren en un mismo actor), por lo cual se ejecutan en un mismo CPU

	ejemplo:

	sourceSource(List(1, 2, 3)).map(_ + 1).async.map(_ * 2).to(Sink.ignore)


	con esto agregamos una async boundary:

	Actor 1 ====> Source(1,2,3) , map(x => x + 1)

	Actor 2 ====> map(x => x * 2), Sink.ignore 			// es decir, ejecuta el resto del stream

* source pre-materialization: hay situaciones donde se quiere un source materializado previo a que el source este conectado con el resto de la topologia. Generalmente esto se usa cuando se trabaja con Source.queue, Source.actorRef o Source.maybe.

En estos casos se puede usar preMaterialize sobre el source. Esto nos devuelve un valor materializado y otro Source. Este otro source lo que hace es consumir mensajes del source original, y es el que debemos usar para conectar con el resto del grafo.

	val completeWithDone: PartialFunction[Any, CompletionStrategy] = { case Done => CompletionStrategy.immediately }
	
	val originalSource =
	  Source.actorRef[String](
	    completionMatcher = completeWithDone,
	    failureMatcher = PartialFunction.empty,
	    bufferSize = 100,
	    overflowStrategy = OverflowStrategy.fail)

	val (actorRef, source) = originalSource.preMaterialize()

	actorRef ! "Hello!"

	// pass source around for materialization
	source.runWith(Sink.foreach(println))



