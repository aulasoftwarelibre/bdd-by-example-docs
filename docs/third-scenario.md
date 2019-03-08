# Tercer escenario

Vamos a implementar el tercer escenario

```gherkin
Escenario: Pagar con puntos
    Dado que he comprado 5 menús del número 1
    Cuando pido la cuenta recibo una factura de 55 euros
    Y pago con 500 puntos y 5 euros
    Entonces la factura está pagada
    Y he obtenido 0 puntos
```

Si ejecutamos la prueba:

```sh
vendor/bin/behat features/menu.feature:30
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

Escenario: Pagar con puntos                            # features/menu.feature:30
    Dado que he comprado 5 menús del número 1            # FeatureContext::queHeCompradoMenusDelNumero()
    Cuando pido la cuenta recibo una factura de 55 euros # FeatureContext::pidoLaCuentaReciboUnaFacturaDeEuros()
    Y pago con 500 puntos y 5 euros                      # FeatureContext::pagoConPuntosYEuros()
    Entonces la factura está pagada                      # FeatureContext::laFacturaEstaPagada()
    Y he obtenido 0 puntos                               # FeatureContext::heObtenidoPuntos()

1 scenario (1 passed)
6 steps (6 passed)
0m0.02s (10.09Mb)
```

¡La prueba pasa! No tenemos que implementar nada nuevo. Una cuestión importante a la hora de escribir los pasos (_steps_), es intentarlos escribir siempre de la misma manera, de tal manera que podamos reutilizarlos en sucesivas pruebas. De esta manera, en muchas ocasiones comprobaremos que no tenemos que implementar escenarios porque ya se ejecutan con los pasos definidos en los anteriores.
