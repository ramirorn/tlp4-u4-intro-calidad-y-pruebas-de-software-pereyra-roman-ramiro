# Respuestas del laboratorio

## Registro de resultados

| Comprobación | Resultado | Observación |
| --- | --- | --- |
| Respuesta de `GET /salud` | `{"estado":"ok"}` | La guía dice `/healthy` pero en `app.ts` la ruta es `/salud`. |
| Primera ejecución de `npm test` | 4 pasan, 1 falla | Falla "rechaza descuentos superiores al 100 %": la función no lanzaba error. |
| Corrección del servicio | 5 pasan | Se valida precio y descuento con `Number.isFinite` y rangos. |
| Nuevas pruebas unitarias | 9 pasan | Descuento 100 %, precio negativo, descuento negativo y `NaN`. |
| Prueba de integración HTTP 400 | 10 pasan | La ruta devuelve 400 con `error` cuando el servicio lanza `RangeError`. |
| Verificación `npm run typecheck` | Sin errores | |

## Preguntas de la sección 2

- La regla de cálculo está en `src/services/precios.ts` y la ruta HTTP en `src/app.ts`.
- El servicio solo sabe calcular; la ruta se encarga de leer el body, llamar al servicio y traducir el resultado (o el error) a una respuesta HTTP.

## Preguntas de la sección 4

1. Devuelve `-500` (1000 - 1000 * 150 / 100).
2. Un descuento no puede ser mayor al precio: el total queda negativo.
3. Porque la prueba espera que se lance un `RangeError`. Que la función termine y devuelva un número es justamente lo que no tiene que pasar.

## Desafío de la sección 6

Sí, `Number.isFinite(NaN)` da `false`, así que la validación lo rechaza. Lo dejé cubierto con una prueba.

## Sección 7

Es de integración porque no se llama a la función directamente: se manda una petición a la app de Express y se revisa la respuesta. Se prueba que la ruta, el parseo del JSON, el servicio y el manejo de errores funcionen juntos.

## Sección 8

`npm test` ejecuta el código y comprueba comportamientos; `typecheck` no ejecuta nada, solo analiza tipos. Node corre los `.ts` borrando los tipos, así que un error de tipos no aparece si no se corre `tsc`. Cada uno detecta problemas que el otro no ve.

## Sección 9

Después de versionar `package-lock.json` cambié el workflow a `npm ci`, para que en CI se instalen exactamente las mismas versiones.

## Preguntas de cierre

1. La prueba manual depende de que alguien la repita y mire el resultado; la automatizada se corre con un comando, siempre igual, y avisa sola si algo cambió.
2. `calcularPrecioFinal` no validaba la entrada, entonces un descuento de 150 % devolvía un total negativo en lugar de lanzar error.
3. Porque los errores suelen estar en los bordes y en lo que no se esperaba recibir. Si solo se prueban casos normales, el bug del 150 % nunca aparece.
4. `precios.test.ts` prueba solo la función del servicio. `api.test.ts` prueba la app de Express completa: rutas, parseo del body, servicio y códigos de estado.
5. Se detectan regresiones en el momento, cuando todavía se sabe qué cambio las provocó, y se puede modificar el código con más confianza.
