# Modelando con ejemplos

## Captura de requisitos

Imaginemos que queremos crear un sistema de fidelización de clientes de un restaurante, dándoles una serie de puntos que le permitan obtener descuentos en sucesivas visitas. Cuando usamos reglas para la captura de requisitos, corremos el riesgo de ser ambiguos.

Por ejemplo, veamos que ocurre cuando especificamos solo con reglas:

!!! example

    * **RF01**: Los clientes obtienen un punto por cada euro gastado en un menú.
    * **RF02**: Diez puntos pueden ser canjeados por un euro de descuento al pagar un menú.
    * **RF03**: El IVA es del 10%.

El problema es que nos surgen una serie de dudas que estas reglas no resuelven:

* ¿Si gasto puntos aún puedo ganar puntos?
* ¿Puedo gastar más de diez puntos en un solo menú?
* ¿El IVA se aplica al precio descontado o al total?

Si modelamos los requisitos con ejemplos obtenemos esto:

!!! example

    Si un menú para una familia de cinco personas cuesta 50 euros:

    * Si pagan en efectivo pagarán 50€ más 5€ de IVA y obtendrán 50 puntos.
    * Si pagan con 10 puntos, costará 10 puntos y 49€ + 5€ de IVA y obtendrán 0 puntos.
    * Si pagan solo con puntos, entregarán 500 puntos + 5€ de IVA y obtendrán 0 puntos.

Evidentemente el ejemplo cuenta con pocas reglas, pero lo que se quiere hacer notar es que es relativamente sencillo llegar a ambigüedades que con los ejemplos no obtendríamos. Por ello, en UML contamos con los casos de uso, que es una forma de captura de requisitos funcionales que permite evitar este tipo de problemas.

Nos vamos a ahorrar la parte de crear los casos de uso y vamos ir directamente a crear las historias de usuario con Gherkin.

## Gherkin

[Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) es un lenguaje que permite describir el comportamiento del software sin entrar en detalles de cómo se ha implementado dicho comportamiento. Cada característica que se define con _gherkin_ debe ir en un fichero con extensión `.feature`. A continuación se puede ver una plantilla de ejemplo:

    #!gherkin
    # language: es
    Característica: Internal operations
        In order to stay secret
        As a secret organization
        We need to be able to erase past agents' memory

        Antecedentes:
            [Dados|Dadas|Dada|Dado|*] there is agent A
            [*|Y|E] there is agent B

        Escenario: Erasing agent memory
            [Dados|Dadas|Dada|Dado|*] there is agent J
            [*|Y|E] there is agent K
            [Cuando|*] I erase agent K's memory
            [Entonces|*] there should be agent J
            [Pero|*] there should not be agent K

        Esquema del escenario: Erasing other agents' memory
            [Dados|Dadas|Dada|Dado|*] there is agent <agent1>
            [*|Y|E] there is agent <agent2>
            [Cuando|*] I erase agent <agent2>'s memory
            [Entonces|*] there should be agent <agent1>
            [Pero|*] there should not be agent <agent2>

            Ejemplos:
            | agent1 | agent2 |
            | D      | M      |

Las palabras entre \[ \] indican una entre las posibles. Para ver el formato en inglés lee la documentación oficial.

### Introducción de la característica

Cada archivo .feature consiste convencionalmente en una única característica. Una línea que comienza con la palabra clave _Característica_ seguida de texto tabulado que la describe. Una característica generalmente contiene una lista de escenarios y unos antecedentes. Cada escenario consiste en una lista de pasos, que debe comenzar con una de las palabras clave indicadas en la plantilla.

Los **antecedentes** son datos disponibles antes de cada prueba. Lo habitual es resetear el estado de la aplicación para que al comienzo de cada escenario esté tal y como indican los antecedentes.

Los **escenarios** son las características que deben ser implementadas y se componen de tres secciones:

1. "**Dado** unos antecedentes": Permiten establecer un estado de la aplicación específico para esta prueba.
1. "**Cuando**" ocurre o se realiza una acción: Aquí es donde se prueba la característica a programar.
1. "**Entonces**" ocurre una consecuencia: Aquí es donde se comprueba que la característica funciona correctamente.

Se pueden concatenar sentencias con las palabras **Y**, **E** y **PERO**, tal y como se ve en la plantilla.

## Creando los escenarios

Vamos a describir un posible archivo _gherkin_ para nuestro ejemplo del restaurante:

    #!gherkin
    #language: es
    Característica: Pagar un menú
        Reglas:

        - 1 punto por cada euro.
        - 10 puntos equivalen a un descuento de 1 euros.
        - El IVA es del 10%

        Antecedentes:
            Dados los siguientes menús:
            | número | precio |
            | 1      | 10     |
            | 2      | 12     |
            | 3      |  8     |

        Escenario: Ganar puntos al pagar en efectivo
            Dado que he comprado 5 menús del número 1
            Cuando pido la cuenta recibo una factura de 55 euros
            Y pago en efectivo con 55 euros
            Entonces la factura está pagada
            Y he obtenido 50 puntos

        Escenario: Pagar con dinero y puntos
            Dado que he comprado 5 menús del número 1
            Cuando pido la cuenta recibo una factura de 55 euros
            Y pago con 10 puntos y 54 euros
            Entonces la factura está pagada
            Y he obtenido 0 puntos

        Escenario: Pagar con puntos
            Dado que he comprado 5 menús del número 1
            Cuando pido la cuenta recibo una factura de 55 euros
            Y pago con 500 puntos y 5 euros
            Entonces la factura está pagada
            Y he obtenido 0 puntos

        Escenario: Intentar pagar el IVA con puntos
            Dado que he comprado 5 menús del número 1
            Cuando pido la cuenta recibo una factura de 55 euros
            Y pago con 550 puntos y 0 euros
            Entonces la factura no está pagada
            Y quedan 5 euros por pagar

        Escenario: Comprar menús de varios tipos
            Dado que he comprado 1 menú del número 1
            Y que he comprado 2 menús del número 2
            Y que he comprado 2 menús del número 3
            Cuando pido la cuenta recibo una factura de 55 euros
            Y pago en efectivo con 55 euros
            Entonces la factura está pagada
            Y he obtenido 50 puntos

El anterior archivo describe las características, en un lenguaje de negocio, de las características del software que debemos implementar. En los siguientes capítulos iremos implementando el proyecto paso a paso.
