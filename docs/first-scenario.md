# Implementando el primer escenario

El primer escenario es el siguiente:

    #!gherkin
    Escenario: Ganar puntos al pagar en efectivo
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago en efectivo con 55 euros
        Entonces la factura está pagada
        Y he obtenido 50 puntos

Para implementar esta escenario, que es realmente el primero de nuestro proyecto que vamos a implementar, necesitamos una clase cuenta (_Bill_) que guarde los menús que se han consumido e informe del coste total, de la cantidad que se ha ingresado (en dinero o puntos), de lo que resta por pagar y de los puntos obtenidos.

## Describir la clase _Bill_

Usamos _PhpSpec_ para crear la especificación, siguiendo los pasos que anteriormente hicimos con la clase _Menu_.

    #!sh
    bash$ vendor/bin/phpspec desc Restaurant/Bill
    Specification for Restaurant\Bill created in /home/sergio/Developer/curso-web/bdd-by-example/bdd-by-example/spec/Restaurant/BillSpec.php.

    bash$ vendor/bin/phpspec run
    
          Restaurant\Bill
    
      11  ! is initializable
            class Restaurant\Bill does not exist.
    
          Restaurant\Menu
    
      19  ✔ is initializable
      24  ✔ has a menu number
      29  ✔ has a price
    
    ----  broken examples
    
            Restaurant/Bill
      11  ! is initializable
            class Restaurant\Bill does not exist.
    
    
    2 specs
    4 examples (3 passed, 1 broken)
    71ms
                                                                                    
      Do you want me to create `Restaurant\Bill` for you?                           
                                                                             [Y/n] 
    
    Class Restaurant\Bill created in /home/sergio/Developer/curso-web/bdd-by-example/bdd-by-example/src/Restaurant/Bill.php.
    
    
          Restaurant\Bill
    
      11  ✔ is initializable
    
          Restaurant\Menu
    
      19  ✔ is initializable
      24  ✔ has a menu number
      29  ✔ has a price
    
    
    2 specs
    4 examples (4 passed)
    71ms

Ya tenemos nuestra especificación `spec/Restaurant/BillSpec.php` y nuestra clase `Restaurant/Bill.php`

Ahora tenemos que describir la API de nuestra clase. Concretamente para este escenario nuestra clase debe proporcionar una API para:

* Añadir un menú a la cuenta
* Obtener el total de la cuenta
* Permitir pagar una cantidad de dinero
* Determinar cuánto resta por pagar
* Determinar cuántos puntos se han ganado

Para esta prueba vamos a suponer que tenemos ya una instancia de _Menu_ que cuesta 10€. ¿Importa el número de menú? No, en realidad no. En más, ¿podríamos añadir elementos a la cuenta que no fueran menús? Lo lógico es que sí. Entonces, ¿cómo hacemos nuestra clase compatible con cualquier clase que tenga un precio? Pues utilizando interfaces. Vamos a crear una interfaz _Priced_ que obligue a las clases que lo implementen a devolver _price()_. Entramos en la primera refactorización de nuestra clase _Menu_.

## Refactorizar _Menu_

Añadimos esta especificación a _MenuSpec_:

    #!php hl_lines="4 5 6 7"
    <?php
    class MenuSpec extends ObjectBehavior
    {
        function it_implements_price_interface()
        {
            $this->shouldImplement(\Restaurant\Priced::class);
        }
    }

!!! info

    Ponemos el namespace completo de `\Restaurant\Price`, pero podemos importarla con `use Restaurant\Price;` en la cabecera de nuestro archivo.

En este caso `Priced` aún no existe y la prueba fallará, como es normal. Pero en esta ocasión _phpspec_ no creará la clase, simplemente se limitará a fallar. Vamos a crear nuestra interfaz en `Restaurant/Priced.php`:

    #!php
    <?php

    namespace Restaurant;


    interface Priced
    {
        public function price(): int;
    }

Ahora necesitamos que nuestra clase _Menu_ implemente la interfaz _Priced_:

    #!php hl_lines="5"
    <?php

    namespace Restaurant;

    class Menu implements Priced
    {
        private $number;
        private $price;

        public function __construct(int $number, int $price)
        {
            $this->number = $number;
            $this->price = $price;
        }

        public function number(): int
        {
            return $this->number;
        }

        public function price(): int
        {
            return $this->price;
        }
    }

Y ya hemos conseguido que la prueba pase:

    #!sh
    vendor/bin/phpspec run
    
          Restaurant\Bill
    
      11  ✔ is initializable
    
          Restaurant\Menu
    
      19  ✔ implements price interface
      24  ✔ is initializable
      29  ✔ has a menu number
      34  ✔ has a price
    
    
    2 specs
    5 examples (5 passed)
    82ms

## Añadiendo elementos a la cuenta

Ahora ya podemos añadir elementos a la cuenta, sin importarnos si es un menú o cualquier otra cosa, solo los importa que tenga precio. Vamos a hacer las pruebas con un solo elemento que cueste 10€.

Vamos a crear el método _let_ para configurar los datos de ejemplo:

    #!php
    <?php

    namespace spec\Restaurant;

    use Restaurant\Bill;
    use PhpSpec\ObjectBehavior;
    use Prophecy\Argument;
    use Restaurant\Priced;

    class BillSpec extends ObjectBehavior
    {
        function let(Priced $item)
        {
            $item->price()->willReturn(1000);
        }
        
        function it_is_initializable()
        {
            $this->shouldHaveType(Bill::class);
        }
    }

En esta ocasión no estamos usando _let_ para configurar el constructor de la clase, que por ahora no hemos determinado que vayamos a necesitar, sino para configurar una instancia de una clase que implementa el interfaz _Priced_ y que cuando se llamen a la función _price()_ devolverá 1000. Hay que tener en cuenta que _Priced_ es una interfaz, no una clase, y que en realidad no sería posible crear instancias de _Priced_. Sin embargo, _phpspec_ crear un _doble_, una clase que implementa la interfaz que se le indica y que simula las respuestas a los métodos con los valores que se le indican con las cláusulas _willReturn_.

    #!php
    <?php

    namespace spec\Restaurant;

    use Restaurant\Bill;
    use PhpSpec\ObjectBehavior;
    use Prophecy\Argument;
    use Restaurant\Priced;

    class BillSpec extends ObjectBehavior
    {
        function let(Priced $item)
        {
            $item->price()->willReturn(1000);
        }

        function it_is_initializable()
        {
            $this->shouldHaveType(Bill::class);
        }

        function it_has_no_items_by_default()
        {
            $this->getTotal()->shouldBe(0);
        }

        function it_adds_an_item(Priced $item)
        {
            $this->addItem($item);
            $this->getTotal()->shouldBe(1100)
        }

        function it_adds_multiple_items(Priced $item, Priced $anotherItem)
        {
            $anotherItem->price()->willReturn(2000);
            $this->addItem($item);
            $this->addItem($anotherItem);
            $this->getTotal()->shouldBe(3300);
        }
    }


Estamos describiendo que nuestra cuenta, cuando se crea, no debe tener ningún elemento, y que los elementos que se añaden incrementan la cuenta (con IVA). Debido al incremento del IVA el valor de retorno será siempre flotante. Ejecutamos las pruebas, que fallarán, y pasamos a implementar el código. Pasamos a completar el código de nuestra clase _Bill_:

    #!php
    <?php
    declare(strict_types=1);

    namespace Restaurant;

    class Bill
    {
        const VAT = '1.10';
        private $items;

        public function __construct()
        {
            $this->items = [];
        }

        public function getTotal(): int
        {
            return array_reduce($this->items, function ($carry, Priced $priced) {
                return $carry + $priced->price();
            }, 0) * self::VAT;
        }

        public function addItem(Priced $item): void
        {
            $this->items[] = $item;
        }
    }

Y ejecutamos las pruebas:

    #!sh
    vendor/bin/phpspec run Restaurant\\Bill

        Restaurant\Bill

    17  ✔ is initializable
    22  ✔ has no items by default
    27  ✔ adds an item
    33  ✔ adds multiple items


    1 specs
    4 examples (4 passed)
    112ms

## Implementando los primeros steps

Ahora estamos en posición de implementar los primeros steps:

    #!gherkin hl_lines="2 3"
    Escenario: Ganar puntos al pagar en efectivo
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago en efectivo con 55 euros
        Entonces la factura está pagada
        Y he obtenido 50 puntos

Quedando el código en el archivo `FeatureContext` como sigue:

    #!php hl_lines="14 26 44 45 46 47 48 56"
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
        public function pagoEnEfectivoConEuros($arg1)
        {
            throw new PendingException();
        }

        /**
        * @Then la factura está pagada
        */
        public function laFacturaEstaPagada()
        {
            throw new PendingException();
        }

        /**
        * @Then he obtenido :arg1 puntos
        */
        public function heObtenidoPuntos($arg1)
        {
            throw new PendingException();
        }

        /**
        * @When pago con :arg1 puntos y :arg2 euros
        */
        public function pagoConPuntosYEuros($arg1, $arg2)
        {
            throw new PendingException();
        }

        /**
        * @Then quedan :arg1 euros por pagar
        */
        public function quedanEurosPorPagar($arg1)
        {
            throw new PendingException();
        }

        /**
        * @Given que he comprado :arg1 menú del número :arg2
        */
        public function queHeCompradoMenuDelNumero($arg1, $arg2)
        {
            throw new PendingException();
        }
    }


Y comprobamos que, efectivamente, el código funciona:

    bash$ vendor/bin/behat features/menu.feature:16
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
        Dado que he comprado 5 menús del número 1            # FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros # FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago en efectivo con 55 euros                      # FeatureContext::pagoEnEfectivoConEuros()
          TODO: write pending definition
        Entonces la factura está pagada                      # FeatureContext::laFacturaEstaPagada()
        Y he obtenido 50 puntos                              # FeatureContext::heObtenidoPuntos()

    1 scenario (1 pending)
    6 steps (3 passed, 1 pending, 2 skipped)
    0m0.02s (10.09Mb)

# Implementando el pago

Para implementar el pago debemos ser capaces de indicar una cantidad pagada en metálico, ver cuánto queda por pagar y ver cuántos puntos hemos obtenido.

Esta sería la especificación:

    #!php hl_lines="41 42 43 44 45 46 48 49 50 51 52 53 55 56 57 58 59 60 61 62"
    <?php

    namespace spec\Restaurant;

    use Restaurant\Bill;
    use PhpSpec\ObjectBehavior;
    use Prophecy\Argument;
    use Restaurant\Priced;

    class BillSpec extends ObjectBehavior
    {
        function let(Priced $item)
        {
            $item->price()->willReturn(1000);
        }

        function it_is_initializable()
        {
            $this->shouldHaveType(Bill::class);
        }

        function it_has_no_items_by_default()
        {
            $this->getTotal()->shouldBe(0);
        }

        function it_adds_an_item(Priced $item)
        {
            $this->addItem($item);
            $this->getTotal()->shouldBe(1100);
        }

        function it_adds_multiple_items(Priced $item, Priced $anotherItem)
        {
            $anotherItem->price()->willReturn(2000);
            $this->addItem($item);
            $this->addItem($anotherItem);
            $this->getTotal()->shouldBe(3300);
        }

        function it_can_be_paid_with_money(Priced $item)
        {
            $this->addItem($item);
            $this->payWithMoney(1100);
            $this->restToPay()->shouldBe(0);
        }

        function it_can_give_points_when_is_payed_with_money(Priced $item)
        {
            $this->addItem($item);
            $this->payWithMoney(1100);
            $this->getPoints()->shouldBe(10);
        }

        function it_can_not_give_points_when_total_is_not_enough(Priced $anotherItem)
        {
            $anotherItem->price()->willReturn(99);

            $this->addItem($anotherItem);
            $this->payWithMoney(109);
            $this->getPoints()->shouldBe(0);
        }
    }

Estamos describiendo distintos casos:

* Cuando se paga exacto y no queda nada por pagar
* Cuando se pagan justo 10 euros
* Cuando se pagan menos de 1 euro

Podríamos ser más exhaustivos, como determinar que no se den puntos hasta que no se pague, pero lo dejamos para los casos siguientes.

Ejecutamos las pruebas para que _phpspec_ genere los métodos en nuestra clase y completamos el código en la clase _Bill_:

    #!php hl_lines="19 27 28 29 30 32 33 34 35 37 38 39 40 42 43 44 45 46 47"
    <?php
    declare(strict_types=1);

    namespace Restaurant;

    class Bill
    {
        const VAT = '1.10';
        private $items;
        private $amount;

        public function __construct()
        {
            $this->items = [];
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
            return $this->getTotal() - $this->amount;
        }

        public function getPoints(): int
        {
            return (int) floor($this->totalWithoutVAT() / 100);
        }

        private function totalWithoutVAT(): int
        {
            return array_reduce($this->items, function ($carry, Priced $priced) {
                return $carry + $priced->price();
            }, 0);
        }
    }


Y comprobamos que pasamos las pruebas:

    #!sh
    bash$ vendor/bin/phpspec run Restaurant\\Bill

        Restaurant\Bill

    17  ✔ is initializable
    22  ✔ has no items by default
    27  ✔ adds an item
    33  ✔ adds multiple items
    41  ✔ can be paid with money
    48  ✔ can give points when is payed with money
    55  ✔ can not give points when total is not enough


    1 specs
    7 examples (7 passed)
    172ms

Ya nos resta terminar de implementar la historia de usuario. Nuestra clase `FeatureContext` queda así:

    #!php hl_lines="63 71 79"
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
        * @When pago con :arg1 puntos y :arg2 euros
        */
        public function pagoConPuntosYEuros($arg1, $arg2)
        {
            throw new PendingException();
        }

        /**
        * @Then quedan :arg1 euros por pagar
        */
        public function quedanEurosPorPagar($arg1)
        {
            throw new PendingException();
        }

        /**
        * @Given que he comprado :arg1 menú del número :arg2
        */
        public function queHeCompradoMenuDelNumero($arg1, $arg2)
        {
            throw new PendingException();
        }
    }

Si ejecutamos este primer escenario debemos comprobar que se ha completado con éxito:

    #!sh
    bash$ vendor/bin/behat features/menu.feature:16
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
        Dado que he comprado 5 menús del número 1            # FeatureContext::queHeCompradoMenusDelNumero()
        Cuando pido la cuenta recibo una factura de 55 euros # FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
        Y pago en efectivo con 55 euros                      # FeatureContext::pagoEnEfectivoConEuros()
        Entonces la factura está pagada                      # FeatureContext::laFacturaEstaPagada()
        Y he obtenido 50 puntos                              # FeatureContext::heObtenidoPuntos()

    1 scenario (1 passed)
    6 steps (6 passed)
    0m0.02s (10.08Mb)
