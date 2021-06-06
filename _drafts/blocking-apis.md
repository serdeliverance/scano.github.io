Blocking and non-blocking

# Intro

Recently, I have to deal with a blocking API (`jedis`, more preciselly its `subscribe` method)

# CPU bound vs IO bound

# Blocking vs Non-blocking

# Sync vs Async

# Dealing with blocking operations on play framework

`TODO intro`

`TODO example code`

# Recommendations

* Try to use non-blocking apis whenever you can. Generally, non-blocking apis are those which returns Future[Something]
* Analize with operations can be blocking (not just blocking calls, but even CPU intensive tasks).
* Be concerned about the components you are working with (database, messaging queues, apis) and ask yourself if any of them uses blocking calls on their apis.
* Use dedicated thread pool for blocking tasks in order to identifies performance rants and can perform a fine tune if needed. 

# Conclusions

# ------------ Lo que sigue son notas mías

# Ejemplos

* algunas tecnologías de mensajería proveen APIs asíncronas para enviar o recibir mensajes (ejemplo: redis pub/sub)
* REST client APIS de terceros que sean bloqueantes
* cuando se abren archivos o sockets directamente
* CPU intensive tasks que esté tomando mucho tiempo

# Importante!

Wrapear una llamada bloqueante en un Future, no convierte a la tarea en no bloqueante, sino que hace que esa tarea se ejecute en otro thread y, por ende, termine bloqueando a este.

# APIs que no son bloqueantes

* Play WS API
* Asynchronous drivers como ReactiveMongo
* Sending/Receiving messages from actors


# Links

https://www.playframework.com/documentation/2.8.x/ThreadPools
https://blog.colinbreck.com/calling-blocking-code-there-is-no-free-lunch/