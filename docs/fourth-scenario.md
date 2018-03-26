# Cuarto escenario

Vamos a implementar el cuarto escenario

    #!gherkin
    Escenario: Intentar pagar el IVA con puntos
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago con 550 puntos y 0 euros
        Entonces la factura no está pagada
        Y quedan 5 euros por pagar

Si ejecutamos la prueba:

    #!sh
    vendor/bin/behat features/menu.feature:37
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

    Escenario: Intentar pagar el IVA con puntos            # features/menu.feature:37
        Dado que he comprado 5 menús del número 1            # FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros # FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago con 550 puntos y 0 euros                      # FeatureContext::pagoConPuntosYEuros()
        Entonces quedan 5 euros por pagar                    # FeatureContext::quedanEurosPorPagar()
        TODO: write pending definition

    1 scenario (1 pending)
    5 steps (4 passed, 1 pending)
    0m0.02s (9.98Mb)


En esta ocasión, las funcionalidades que queremos comprobar ya las tenemos, solo que no con esas sentencias. Vamos a implementar directamente esas sentencias en el _FeatureContext_ y ver si nuestra clase funciona:

    #!php hl_lines="20"
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
        //...

        /**
        * @Then quedan :amount euros por pagar
        */
        public function quedanEurosPorPagar($amount)
        {
            \PHPUnit\Framework\Assert::assertEquals($amount * 100, $this->bill->restToPay());
        }
    }

Ejecutamos de nuevo la prueba:

    #!sh
    bash$ vendor/bin/behat features/menu.feature:37
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
    
      Escenario: Intentar pagar el IVA con puntos            # features/menu.feature:37
        Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago con 550 puntos y 0 euros                      #     FeatureContext::pagoConPuntosYEuros()
        Entonces quedan 5 euros por pagar                    #     FeatureContext::quedanEurosPorPagar()
          Failed asserting that 0 matches expected '5'.
    
    --- Failed scenarios:
    
        features/menu.feature:37
    
    1 scenario (1 failed)
    5 steps (4 passed, 1 failed)
    0m0.02s (10.20Mb)

El escenario falla porque en la especificación no hemos indicado que el IVA no se puede pagar con puntos. Así que creamos una nueva regla en _BillSpec_ para tener en cuenta este comportamiento.

    #!php hl_lines="14 15 16 17 18 19 "
    <?php

    namespace spec\Restaurant;

    use Restaurant\Bill;
    use PhpSpec\ObjectBehavior;
    use Prophecy\Argument;
    use Restaurant\Priced;

    class BillSpec extends ObjectBehavior
    {
        // ...

        function it_can_not_pay_VAT_with_points(Priced $item)
        {
            $this->addItem($item);
            $this->payWithPoints(110);
            $this->restToPay()->shouldBe(100);
        }
    }

Y modificamos nuestra clase _Bill_ para pasar la especificación:

    #!php hl_lines="63 64 65 66"
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
            $maxMoneyPoints = $this->totalWithoutVAT();
            $moneyPoints = 100 * $this->points / 10;
    
            return $moneyPoints > $maxMoneyPoints ? $maxMoneyPoints : $moneyPoints;
        }
    }

Y comprobamos que esto consigue que la prueba pase:

    #!sh
    bash$  vendor/bin/behat features/menu.feature:37
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
    
      Escenario: Intentar pagar el IVA con puntos            # features/menu.feature:37
        Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago con 550 puntos y 0 euros                      #     FeatureContext::pagoConPuntosYEuros()
        Entonces quedan 5 euros por pagar                    #     FeatureContext::quedanEurosPorPagar()
    
    1 scenario (1 passed)
    5 steps (5 passed)
    0m0.02s (9.97Mb)
