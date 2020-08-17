testing

comprobar que cierta funcionalidad cumple con los requisitos que le dieron existencia en su totalidad, o al menos en un margen aceptable pactado entre el cliente y el desarrollador.

Beneficios:

* repetitividad
* prevención contra regresiones / mantenibilidad
* sirve como documentación
* aumentar la calidad del software entregado

# Tipos de testing

## Unit test

Test que abarca la prueba de componentes individuales de software. Diseñados para validar el correcto comportamiento de dichas unidades de codigo. 

En OOP, la unidad a testear son los métodos, en estructurado, los procedimientos y funciones.

### Qué testear?

Como la unidad de código es una función o método, generalmente se suele testear cada flujo/branch de ejecución que tenga. Es decir, si tiene un if-else, se testean los dos flujos. Es decir, hay una relación con la complejidad ciclomática de la función (es mayor o igual)

### Convenciones

#### Generales

* usar Scalatest
* si estamos usando Play Framework, heredar de PlaySpec
* cuando se extiende de PlaySpec se toma como base el estilo de test propuesto por el trait WordSpecLike

* Las clases a testear deben nombrarse como: nombre de la clase bajo test o SUT (Subject Under Test) seguido del sufijo Spec.

* son importantes los métodos before and after, que se ejecutan antes y despues de cada test de la test suite para tener un setup/clean del environment que necesita cada test. Ambos métodos vienen en el trait BeforeAndAfter

* cada test suele tener tres etapas durante su ejecución: inicialización del escenario, ejecución de la unidad del SUT y las aserciones. Es una buena práctica añadir los comentarios given, when, then para demarcar estas secciones en el código para facilitar su comprensión.

Es importante que cada test tenga 3 fases: "quién", "cuando ocurre qué", "qué debería hacer"

Ejemplo:

``` scala

// Class to test
class Calculator {
  def clean() = {
    // Cleaning logic
  }

  def sum(a: Int, b: Int): Int = a + b
}

// Test suite class
class CalculatorSpec extends PlaySpec with BeforeAndAfter {
  var subject: Calculator

  before {
    subject = new Calculator()
  }

  after {
    subject.clean()
  }

  "sum method" when {
    "given two valid numbers" should {
      "sum both values correctly and return the result" in {
        // given
        val firstNumber = 10
        val secondNumber = 13

        // when
        val result = subject.sum(firstNumber, secondNumber)

        // then
        result mustBe 23
      }
    }
  }
}
```

#### Asincronia

Cuando queremos testear componentes asincrónicos en Scalatest + Play Framework, existen utilidades que nos pueden servir:

* AsyncWordSpec: nos provee de herramientas/construcciones para realizar pruebas asíncronas (el codigo retorna Future[Assertion]) y nos permite seguir utilizando el DSL de WordSpec.

``` scala
class AddSpec extends AsyncWordSpec {
  def addSoon(addends: Int*): Future[Int] = Future { addends.sum }

  "addSoon" should {
    "eventually compute a sum of passed Ints" in {
      val futureSum: Future[Int] = addSoon(1, 2)
      // Se puede mapear el resultado de un Future de tal manera que 
      // las aserciones sobre el mismo se devuelvan a ScalaTest como
      // un Future[Assertion]:
      futureSum map { sum => assert(sum == 3) }
    }
  }
}
```

Interesante ver cómo se hacen las asserciones asincrónicamente (es decir, aplicando un map sobre el future)

* PlaySpec + WordSpec: Cuando se heredan estas clases, generalmente se suelen escribir test que, el componente a testear es un metodo asincrono, testean el valor que devuelve el Future. Es decir, esperan el resultado del future (hacen una especie de await) y luego performan las aserciones sobre eso. O sea, son test más bien blocking.. lo cual es aceptable porque el código no es productivo.

##### Extra tips

Cuando se realicen test sobre futures, una buena opción puede ser usar el trait `ScalaFutures`, que contiene constructos para el manejo de Future

Otro trait a tener en cuenta es `Eventually` , que permite provee estructuras para realizar chequeos periodicos sobre cierta condición.

Eventually se utiliza cuando queremos chequear si se cumple una condición en algún momento del futuro. Es decir, si nuestro metodo del SUT al ejecutarse produce que una condicion sea true luego de pasada una x cantidad de tiempo, Eventually nos sirve para evaluar dicha condición.

`TODO ejemplo de Eventually`

#### Test Doubles

Cuando hacemos unit test, testeamos el funcionamiento de nuestro objeto de test (función, método). No obstante, dicho trozo de código se encuentra dentro de una clase, la cuál tiene dependencias/colaboradores. Como no queremos que la verificación a realizar se vea influenciada por dichos colaboradores, dado que el core de la cuestión es testear el funcionamiento de nuestra sección de código, debemos recurrir a `Test Doubles`.

Los `Test Doubles` son objetos, componentes que emulan a los colabores reales de la clase donde se encuentra nuestra funcionalidad a testear.

Los diferentes tipos de `Test Doubles` son:

* Dummies
* Fake
* Stub
* Spies
* Mocks

`TODO completar descripción de cada uno de estos tipos`

# Conclusion

* siempre balancear el uso de testing, ergo: usarlo a consciencia, ya que un mal uso puede dificultar la mantenibilidad del sistema (ej: modifico 1 linea de codigo y tengo que arreglar 20 test que tal vez no aportan tanto valor), entre otras cosas.

# Bookmark

* Pagina 8, Test doubles

# TODO

* ver Eventually
* ejemplos de Eventually
* ver ScalaFuture
* ver AsyncWordSpec
* test doubles. Martin Fowler (https://martinfowler.com/bliki/TestDouble.html)