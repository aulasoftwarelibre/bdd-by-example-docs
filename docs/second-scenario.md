# Segundo escenario

Vamos a implementar ahora el segundo escenario

```gherkin
Escenario: Pagar con dinero y puntos
Dado que he comprado 5 menús del número 1
Cuando pido la cuenta recibo una factura de 55 euros
Y pago con 10 puntos y 54 euros
Entonces la factura está pagada
Y he obtenido 0 puntos
```

Si intentamos ejecutar la prueba directamente vemos que hay ya muchos pasos que pasan:

```sh
vendor/bin/behat features/menu.feature:23
Característica: Pagar un menú
Reglas:

- 1 punto por cada euro.
- 10 puntos equivalen a un descuento de 1 euros.
- El IVA es del 10%

Antecedentes: # features/menu.feature:9
Dados los siguientes menús: # FeatureContext::losSiguientesMenus()
| número | precio |
| 1 | 10 |
| 2 | 12 |
| 3 | 8 |

Escenario: Pagar con dinero y puntos # features/menu.feature:23
Dado que he comprado 5 menús del número 1 # FeatureContext::queHeCompradoMenusDelNumero()
Cuando pido la cuenta recibo una factura de 55 euros # FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
Y pago con 10 puntos y 54 euros # FeatureContext::pagoConPuntosYEuros()
TODO: write pending definition
Entonces la factura está pagada # FeatureContext::laFacturaEstaPagada()
Y he obtenido 0 puntos # FeatureContext::heObtenidoPuntos()

1 scenario (1 pending)
6 steps (3 passed, 1 pending, 2 skipped)
0m0.02s (9.98Mb)
```

Solo tenemos que implementar el pago con puntos.

## Describir la funcionalidad

El primer paso, es escribir la descripción en nuestra especificación de la clase _Bill_:

```php hl_lines="14 15 16 17 18 19 20 21"

<?php

namespace spec\Restaurant;

use Restaurant\Bill;
use PhpSpec\ObjectBehavior;
use Prophecy\Argument;
use Restaurant\Priced;

class BillSpec extends ObjectBehavior
{
    // ...

    function it_can_be_paith_with_money_and_points_and_get_no_points(Priced $item)
    {
        $this->addItem($item);
        $this->payWithMoney(1000);
        $this->payWithPoints(10);
        $this->restToPay()->shouldBe(0);
        $this->getPoints()->shouldBe(0);
    }
}
```

De nuevo, ejecutamos las pruebas de _phpspec_ para que se generen los métodos necesarios en nuestra clase _Bill_ y refactorizamos nuestro código para pasar la prueba:

```php hl_lines="16 37 56 57 58 59 61 62 63 64"
<?php
declare(strict_types=1);

namespace Restaurant;

class Bill
{
    const VAT = '1.10';
    private $items;
    private $amount;
    private $points;

    public function __construct()
    {
        $this->items = [];
        $this->points = 0;
        $this->amount = 0;
    }

    public function getTotal(): int
    {
        return (int) round($this->totalWithoutVAT() * self::VAT);
    }

    public function addItem(Priced $item): void
    {
        $this->items[] = $item;
    }

    public function payWithMoney(int $amount): void
    {
        $this->amount = $amount;
    }

    public function restToPay(): int
    {
        return $this->getTotal() - $this->amount - $this->getMoneyPoints();
    }

    public function getPoints(): int
    {
        if ($this->points > 0) {
            return 0;
        }

        return (int) floor($this->totalWithoutVAT() / 100);
    }

    private function totalWithoutVAT(): int
    {
        return array_reduce($this->items, function ($carry, Priced $priced) {
            return $carry + $priced->price();
        }, 0);
    }

    public function payWithPoints(int $points): void
    {
        $this->points = $points;
    }

    private function getMoneyPoints(): int
    {
        return 100 * ($this->points / 10);
    }
}
```

Comprobamos que pasamos las pruebas y pasamos a implementar la historia de usuario.

## Completando la historia de usuario

Creamos el código que implementa el paso que nos falta:

```php
<?php

use Behat\Behat\Context\Context;
use Behat\Behat\Tester\Exception\PendingException;
use Behat\Gherkin\Node\PyStringNode;
use Behat\Gherkin\Node\TableNode;

/**
* Defines application features from the specific context.
*/
class FeatureContext implements Context
{
    // ...

    /**
    * @When pago con :points puntos y :money euros
    */
    public function pagoConPuntosYEuros($points, $money)
    {
        $this->bill->payWithMoney($money * 100);
        $this->bill->payWithPoints($points);
    }
}
```

Y ya hemos conseguido terminar otro escenario:

```sh
vendor/bin/behat features/menu.feature:23
Característica: Pagar un menú
  Reglas:

  - 1 punto por cada euro.
  - 10 puntos equivalen a un descuento de 1 euros.
  - El IVA es del 10%

  Antecedentes:                 # features/menu.feature:9
    Dados los siguientes menús: # FeatureContext::losSiguientesMenus()
      | número | precio |
      | 1      | 10     |
      | 2      | 12     |
      | 3      | 8      |

  Escenario: Pagar con dinero y puntos                   # features/menu.feature:23
    Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
    Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
    Y pago con 10 puntos y 54 euros                      #     FeatureContext::pagoConPuntosYEuros()
    Entonces la factura está pagada                      #     FeatureContext::laFacturaEstaPagada()
    Y he obtenido 0 puntos                               #     FeatureContext::heObtenidoPuntos()

1 scenario (1 passed)
6 steps (6 passed)
0m0.02s (10.09Mb)
```

## Conclusiones

Como vamos viendo, el código que se genera en _phpspec_ es realmente el código que implementa nuestras reglas de negocio. En _behat_ solo implementamos código para poder usar el dominio en las pruebas y comprobar que nuestras dos clases (_Menu_ y _Bill_) trabajan bien juntas.
