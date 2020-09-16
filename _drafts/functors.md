# Intro

We need to think in a functor as a way to transform an element inside a context without changing the context itself.

Map is a way of sequencing computations over a value which is inside a context.

# Functors

`Functor` is the abstraction behind `map`. A `Functor` is a class that allows sequencing operations.

``` scala
trait Functor[F[_]] {
    def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

# Laws

`Functors` must obey some laws for guaranting that wheter we sequence many operationes one by one, or combining them into a largar function before mapping, we would we the same result. In order to do that, it must to obey:

* Identity:

``` python
fa.map(a => a) == fa
```

* Composition

``` python
fa.map(g(f(_))) == fa.map(f).map(g)
```

# Example of Functors

* Futures: a Future sequence asynchronous computations by queueing them and applying them as their predecessors complete.

----

We can also think in mapping as function composition