# Quinto escenario

Vamos a implementar el último escenario

    #!gherkin
    Escenario: Comprar menús de varios tipos
        Dado que he comprado 1 menú del número 1
        Y que he comprado 2 menús del número 2
        Y que he comprado 2 menús del número 3
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago en efectivo con 55 euros
        Entonces la factura está pagada
        Y he obtenido 50 puntos

Si ejecutamos la prueba:

    #!sh
    bash$  vendor/bin/behat features/menu.feature:43
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
    
      Escenario: Comprar menús de varios tipos               # features/menu.feature:43
        Dado que he comprado 1 menú del número 1             #     FeatureContext::queHeCompradoMenuDelNumero()
          TODO: write pending definition
        Y que he comprado 2 menús del número 2               #     FeatureContext::queHeCompradoMenusDelNumero()
        Y que he comprado 2 menús del número 3               #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago en efectivo con 55 euros                      #     FeatureContext::pagoEnEfectivoConEuros()
        Entonces la factura está pagada                      #     FeatureContext::laFacturaEstaPagada()
        Y he obtenido 50 puntos                              #     FeatureContext::heObtenidoPuntos()
    
    1 scenario (1 pending)
    8 steps (1 passed, 1 pending, 6 skipped)
    0m0.02s (9.65Mb)

Falla en el primer paso. Realmente la frase "que he comprado 1 menú del número 1" difiere de "que he comprado 2 menús del número 2" en el plural de _menús_. Al ser frases distintas behat las interpreta como pasos distintos. Tenemos varias formas de solucionar esto que no pasen por escribir dos veces el mismo código, es decir, conseguir que un mismo paso se ejecute con sentencias distintas.

Por lo pronto el código siguiente:

    #!php
    <?php

    /**
     * @Given que he comprado :arg1 menú del número :arg2
     */
    public function queHeCompradoMenuDelNumero($arg1, $arg2)
    {
        throw new PendingException();
    }

Que pertenece a nuestro _FeatureContext.php_ sobra y lo eliminamos.

## Primer método: Escribir varias sentencias @Given

Una posible solución es añadir a la cabecera del _step_ todas las anotaciones que queramos que lo activen:

    #!php hl_lines="5"
    <?php

    /**
     * @Given que he comprado :arg1 menús del número :arg2
     * @Given que he comprado :arg1 menú del número :arg2
     */
    public function queHeCompradoMenusDelNumero($count, $menuNumber)
    {
        $menu = $this->menus[$menuNumber];

        for($i = 0; $i < $count; $i++) {
            $this->bill->addItem($menu);
        }
    }

## Segundo método: Escribir las reglas como expresiones regulares

Podemos usar expresiones regulares:

    #!php hl_lines="4"
    <?php

    /**
     * @Given /^que he comprado (\d+) menús? del número (\d+)$/
     */
    public function queHeCompradoMenusDelNumero($count, $menuNumber)
    {
        $menu = $this->menus[$menuNumber];

        for($i = 0; $i < $count; $i++) {
            $this->bill->addItem($menu);
        }
    }

En el caso de usar expresiones regulares, debemos usar grupos de captura (el contenido entre paréntesis) de aquellos valores que queramos pasar a la función.

## Estado final

Vamos a poner el código del _FeatureContext_ usando el primer método:

    #!php
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
        private $menus;
        private $bill;

        /**
        * Initializes context.
        *
        * Every scenario gets its own context instance.
        * You can also pass arbitrary arguments to the
        * context constructor through behat.yml.
        */
        public function __construct()
        {
            $this->menus = [];
            $this->bill = new \Restaurant\Bill();
        }

        /**
        * @Given los siguientes menús:
        */
        public function losSiguientesMenus(TableNode $table)
        {
            foreach ($table->getHash() as $menu) {
                $this->menus[$menu['número']] = new \Restaurant\Menu($menu['número'], $menu['precio'] * 100);
            }
        }

        /**
        * @Given que he comprado :arg1 menús del número :arg2
        * @Given que he comprado :arg1 menú del número :arg2
        */
        public function queHeCompradoMenusDelNumero($count, $menuNumber)
        {
            $menu = $this->menus[$menuNumber];

            for($i = 0; $i < $count; $i++) {
                $this->bill->addItem($menu);
            }
        }

        /**
        * @When pido la cuenta recibo una factura de :arg1 euros
        */
        public function pidoLaCuentaReciboUnaFacturaDeEuros($total)
        {
            \PHPUnit\Framework\Assert::assertEquals($total * 100, $this->bill->getTotal());
        }

        /**
        * @When pago en efectivo con :arg1 euros
        */
        public function pagoEnEfectivoConEuros($amount)
        {
            $this->bill->payWithMoney($amount * 100);
        }

        /**
        * @Then la factura está pagada
        */
        public function laFacturaEstaPagada()
        {
            \PHPUnit\Framework\Assert::assertEquals(0, $this->bill->restToPay());
        }

        /**
        * @Then he obtenido :arg1 puntos
        */
        public function heObtenidoPuntos($points)
        {
            \PHPUnit\Framework\Assert::assertEquals($points, $this->bill->getPoints());
        }

        /**
        * @When pago con :points puntos y :money euros
        */
        public function pagoConPuntosYEuros($points, $money)
        {
            $this->bill->payWithMoney($money * 100);
            $this->bill->payWithPoints($points);
        }

        /**
        * @Then quedan :amount euros por pagar
        */
        public function quedanEurosPorPagar($amount)
        {
            \PHPUnit\Framework\Assert::assertEquals($amount * 100, $this->bill->restToPay());
        }#
    }

Y ejecutamos, ahora sí, todas las pruebas:

    #!sh
    bash$ vendor/bin/behat features/menu.feature
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
    
      Escenario: Ganar puntos al pagar en efectivo           # features/menu.feature:16
        Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago en efectivo con 55 euros                      #     FeatureContext::pagoEnEfectivoConEuros()
        Entonces la factura está pagada                      #     FeatureContext::laFacturaEstaPagada()
        Y he obtenido 50 puntos                              #     FeatureContext::heObtenidoPuntos()
    
      Escenario: Pagar con dinero y puntos                   # features/menu.feature:23
        Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago con 10 puntos y 54 euros                      #     FeatureContext::pagoConPuntosYEuros()
        Entonces la factura está pagada                      #     FeatureContext::laFacturaEstaPagada()
        Y he obtenido 0 puntos                               #     FeatureContext::heObtenidoPuntos()
    
      Escenario: Pagar con puntos                            # features/menu.feature:30
        Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago con 500 puntos y 5 euros                      #     FeatureContext::pagoConPuntosYEuros()
        Entonces la factura está pagada                      #     FeatureContext::laFacturaEstaPagada()
        Y he obtenido 0 puntos                               #     FeatureContext::heObtenidoPuntos()
    
      Escenario: Intentar pagar el IVA con puntos            # features/menu.feature:37
        Dado que he comprado 5 menús del número 1            #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago con 550 puntos y 0 euros                      #     FeatureContext::pagoConPuntosYEuros()
        Entonces quedan 5 euros por pagar                    #     FeatureContext::quedanEurosPorPagar()
    
      Escenario: Comprar menús de varios tipos               # features/menu.feature:43
        Dado que he comprado 1 menú del número 1             #     FeatureContext::queHeCompradoMenusDelNumero()
        Y que he comprado 2 menús del número 2               #     FeatureContext::queHeCompradoMenusDelNumero()
        Y que he comprado 2 menús del número 3               #     FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros #     FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago en efectivo con 55 euros                      #     FeatureContext::pagoEnEfectivoConEuros()
        Entonces la factura está pagada                      #     FeatureContext::laFacturaEstaPagada()
        Y he obtenido 50 puntos                              #     FeatureContext::heObtenidoPuntos()
    
    5 scenarios (5 passed)
    31 steps (31 passed)
    0m0.04s (10.07Mb)

Podemos comprobar que hemos conseguido pasar todas las pruebas y nuestra aplicación cumpliría todas las especificaciones.
