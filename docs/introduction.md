# Introducción

## Desarrollo orientado a pruebas (TDD)

Test Driven Development (TDD), o desarrollo orientado a pruebas, es un proceso de desarrollo de software muy conocido, que consiste en la repetición de ciclos de desarrollo muy cortos.

1. Los requisitos se convierten en pruebas, unidades de código que prueban una determinada funcionalidad.
1. Entonces, se produce el código justo que permite pasar dicha prueba.

Este proceso es opuesto a otros sistemas de desarrollo que permiten incorporar código que no se sabe con certeza si concuerdan o no con los requisitos.

Los test que se pueden realizar pueden ser de unidad, funcionales y de aceptación. Para todos estos tipos de test, en PHP podemos usar PHPUnit.

### Ciclo del desarrollo orientado a pruebas

1. Añade un test
1. Ejecuta los test y comprueba que el nuevo test falla
1. Escribe el código
1. Ejecuta los test
1. Refactoriza el código
1. Repite

### Ejemplo de un test de unidad con PHPUnit

Si queremos implementar una clase que implemente funciones matemáticas, este sería el código de una prueba unitaria que, con PHPUnit, escribiríamos para implementar la funcionalidad _suma_:

    #!php
    <?php
    use PHPUnit\Framework\TestCase;

    class CalculatorTest extends TestCase
    {
        public function testAdd()
        {
            $calculator = new Calculator();
            $result = $calculator->add(3, 2);

            $this->assertEquals(5, $result);
        }
    }

Solo cuando tengamos el test escrito podremos desarrollar el código:

    #!php
    <?php
    class Calculator
    {
        public function add (int $a, int $b): int
        {
            return $a + $b;
        }
    }


### Las tres reglas

Robert Martin, en su libro "Clean Code. A Handbook of agile software craftsmanship", especifica las tres reglas que deben seguirse cuando se desarrolla con TDD:

1. No debes escribir código de producción hasta que hayas escrito una prueba unitaria que falle.
1. No debes escribir más de una prueba unitaria que sea suficiente para fallar y el sistema no compile.
1. No debes escribir más código de producción que el suficiente para superar la prueba que esté fallando actualmente.

## Desarrollo orientado a comportamiento (BDD)

Behaviour Driven Development (BDD), o desarrollo orientado a comportamiento, es un proceso de desarrollo similar a TDD, solo que con BDD lo que se realiza es definir el comportamiento y las especificaciones, al contrario que con TDD que se limita solamente a comprobar funcionalidades.

Los test que se pueden realizar pueden ser de unidad, funcionales y de aceptación. Para los test de unidad el software más apropiado en PhpSpec y para el resto Behat.

### Ciclo del desarrollo orientado a comportamiento

1. Describe el comportamiento del software a través de pruebas
1. Ejecuta los test y comprueba que el nuevo test falla
1. Implementa el nuevo comportamiento
1. Ejecuta los test
1. Mejora el código
1. Repite

### Ejemplo de un test de unidad con PHPSpec

Si usamos el ejemplo de la calculadora, esta sería la forma de describir el comportamiento de la calculadora:

    #!php
    <?php
    use PhpSpec\ObjectBehavior;

    class CalculatorSpec extends ObjectBehavior
    {
        function it_is_initializable()
        {
            $this->shouldHaveType(Calculator::class);
        }

        function it_adds_to_numbers()
        {
            $this->add(2, 3)->shouldReturn(5);
        }
    }

Las diferencias principales con PHPUnit son dos:

1. El nombre de la función describe el comportamiento a implementar. Debe empezar por `it_` o `its_`.
1. No es necesario instanciar la clase que queremos probar. La clase _CalculatorSpec_ adquiere los métodos de la clase que estamos describiendo.

Hay muchas más diferencias, pero a grandes rasgos nos vamos a quedar con esas dos.

### Ejemplo de un test con Behat

Behat utiliza un lenguaje llamado Gherkin para describir historias de usuario. Es más apropiado para hacer test de aceptación. Este sería un ejemplo de un test de aceptación con Gherkin:

    #!gherkin
    Característica: Crear una calculadora

        Escenario: Sumar dos números
            Dado que quiero sumar dos números
            Cuando introduzco el número 2
            E introduzco el número 3
            Entonces recibo el número 5

Esta prueba es una simplificación de lo que permite hacer Gherkin. La forma en cómo se ejecuta e implementa este tipo de test lo veremos más adelante.
