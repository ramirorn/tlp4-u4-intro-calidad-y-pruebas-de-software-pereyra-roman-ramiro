# Laboratorio introductorio: pruebas automatizadas en Express.js

**Taller de Lenguajes de Programación IV · Unidad 4 · Octubre de 2026**

## Propósito

En esta práctica se utilizará una API de precios desarrollada con Express.js y TypeScript para comprobar comportamientos mediante pruebas automatizadas. El proyecto ya contiene pruebas: algunas pasan y una falla intencionalmente. El objetivo es comprender los resultados, localizar el defecto y verificar una corrección.

Al finalizar podrás distinguir una prueba unitaria de una prueba de integración, interpretar aserciones de Vitest y comprobar respuestas HTTP con Supertest.

## 1. Preparar el entorno

Necesitás Node.js **22.18 o posterior** (preferentemente Node.js 24), npm, un editor de código y una terminal. El proyecto usa TypeScript ejecutado directamente por Node, sin `ts-node` ni `tsx`.

Abrí la carpeta **demo_practica** en Visual Studio Code y ejecutá:

```bash
node --version
npm --version
npm install
```

El comando `npm install` instala las dependencias y genera `package-lock.json`. Si tu versión de Node es anterior a la indicada, actualizala antes de continuar.

La estructura del proyecto es:

```text
├── src/
│   ├── app.ts
│   ├── server.ts
│   └── services/
│       └── precios.ts
├── tests/
│   ├── precios.test.ts
│   └── api.test.ts
├── .github/
│   └── workflows/
│       └── pruebas.yml
├── package.json
└── tsconfig.json
```

**Importante:** no es necesario crear nuevos archivos de configuración ni reemplazar `package.json`. Los scripts y las dependencias ya están incluidos.

## 2. Conocer la aplicación

Abrí `src/services/precios.ts`. Allí se encuentra la función `calcularPrecioFinal(precio, descuento)`, que resta un porcentaje al precio recibido. Por el momento, la función **no valida** los valores de entrada.

En `src/app.ts` hay dos rutas:

- `GET /healthy`: devuelve un mensaje que confirma que la API responde.
- `POST /api/precios/calcular`: recibe `precio` y `descuento` en formato JSON y devuelve el total calculado.

Iniciá el servidor:

```bash
npm run dev
```

Abrí `http://localhost:3000/healthy` en el navegador. Deberías ver:

```json
{"estado":"ok"}
```

En otra terminal probá el cálculo con `curl`:

```bash
curl -X POST http://localhost:3000/api/precios/calcular \
  -H 'Content-Type: application/json' \
  -d '{"precio":1000,"descuento":10}'
```

Resultado esperado:

```json
{"total":900}
```

En **PowerShell** también podés utilizar:

```powershell
Invoke-RestMethod -Uri 'http://localhost:3000/api/precios/calcular' `
  -Method Post -ContentType 'application/json' `
  -Body '{"precio":1000,"descuento":10}'
```

**Preguntas:** ¿qué archivo contiene la regla de cálculo? ¿Qué archivo define la ruta HTTP? ¿En qué se diferencian sus responsabilidades?

> **Respuesta:** la regla de cálculo está en `src/services/precios.ts` y la ruta HTTP en `src/app.ts`. El servicio solo sabe calcular el precio; la ruta lee el body de la petición, llama al servicio y convierte el resultado (o el error) en una respuesta HTTP con su código de estado.
>
> Nota: en el código la ruta de salud es `GET /salud`, no `/healthy`.

## 3. Ejecutar las pruebas disponibles

Detené el servidor con `Ctrl+C` o abrí otra terminal en el proyecto. Ejecutá:

```bash
npm test
```

El proyecto incluye **cinco pruebas**: tres en `tests/precios.test.ts` y dos en `tests/api.test.ts`. Inicialmente, deberían **pasar cuatro y fallar una**.

En `tests/precios.test.ts` identificá estas instrucciones:

- `describe(...)`: agrupa pruebas relacionadas.
- `it(...)`: describe un comportamiento que se quiere comprobar.
- `expect(...)`: expresa una expectativa sobre el resultado.
- `toBe(...)`: comprueba igualdad para valores como números.
- `toThrow(RangeError)`: comprueba que una función lance un error del tipo indicado.

Leé la prueba que contiene `calcularPrecioFinal(1000, 150)`. ¿Qué comportamiento exige? ¿Qué hace realmente el código? **Todavía no modifiques la implementación.**

> **Respuesta:** la prueba exige que un descuento mayor al 100 % lance un `RangeError`. El código no valida nada: hace la cuenta y devuelve `-500`. Por eso la primera ejecución dio 4 pruebas OK y 1 fallida.

## 4. Interpretar la falla

La prueba inicial incluye la siguiente comprobación:

```typescript
it('rechaza descuentos superiores al 100 %', () => {
  expect(() => calcularPrecioFinal(1000, 150)).toThrow(RangeError);
});
```

Observá que la función se escribe dentro de `() => ...` para que Vitest pueda ejecutarla y comprobar si lanza un error. Es distinto de pasar directamente el resultado de la función a `expect`.

Respondé antes de continuar:

1. ¿Qué total devuelve actualmente un descuento del 150 % sobre $1.000?
2. ¿Por qué ese resultado no cumple la regla de negocio?
3. ¿Por qué la prueba falla si la función terminó de ejecutarse?

> **Respuestas:**
>
> 1. Devuelve `-500` (1000 - 1000 * 150 / 100).
> 2. Un descuento no puede superar el precio; el total queda negativo, cosa que no tiene sentido en una venta.
> 3. Porque la prueba espera que se lance un error. Que la función termine normalmente y devuelva un número es justo lo que no debería pasar.

## 5. Corregir la función de cálculo

Modificá **solo** `src/services/precios.ts`. Antes de realizar la operación, incorporá comprobaciones para que:

- El precio sea un número finito, igual o mayor que cero.
- El descuento sea un número finito entre 0 y 100, inclusive.
- Los valores inválidos provoquen un `RangeError`.

Podés utilizar `Number.isFinite(...)`, comparaciones e instrucciones `throw new RangeError('...')`. **Conservá la fórmula de cálculo para las entradas válidas.**

Volvé a ejecutar:

```bash
npm test
```

Ahora deberían pasar las **cinco pruebas**. Si alguna falla, leé el resultado de Vitest y revisá qué comportamiento espera.

> **Resultado:** agregué las validaciones con `Number.isFinite` y comparaciones de rango antes de la fórmula. Pasan las 5 pruebas.

## 6. Agregar pruebas unitarias

Abrí `tests/precios.test.ts` y agregá, dentro de `describe`, nuevas pruebas para las siguientes reglas:

| Entrada | Resultado esperado |
| --- | --- |
| Precio 1.000 y descuento 100 % | Total 0 |
| Precio -100 y descuento 10 % | Lanza `RangeError` |
| Precio 1.000 y descuento -10 % | Lanza `RangeError` |

Por ejemplo, para el caso límite:

```typescript
it('devuelve cero con un descuento del 100 %', () => {
  expect(calcularPrecioFinal(1000, 100)).toBe(0);
});
```

Escribí las otras dos pruebas siguiendo el modelo de `toThrow(RangeError)` que ya existe. Ejecutá `npm test` y verificá el resultado.

**Desafío adicional:** agregá una prueba para `NaN` como precio. ¿La validación actual lo rechaza?

> **Respuesta:** sí. `Number.isFinite(NaN)` devuelve `false`, así que se lanza `RangeError`. Agregué la prueba y, junto con las tres de la tabla, quedan 9 pruebas pasando.

## 7. Primera prueba de integración HTTP

Hasta ahora se probó la función de manera aislada. Ahora comprobarás cómo colaboran **la ruta HTTP de Express y el servicio de cálculo**.

Abrí `tests/api.test.ts`. Las pruebas existentes utilizan Supertest:

```typescript
const respuesta = await request(app)
  .post('/api/precios/calcular')
  .send({ precio: 1000, descuento: 10 });

expect(respuesta.status).toBe(200);
expect(respuesta.body).toEqual({ total: 900 });
```

La llamada `request(app)` permite probar la aplicación sin iniciar manualmente un servidor en el puerto 3000. El código `200` significa que la petición se procesó correctamente.

Agregá dentro del bloque `describe('API de precios', ...)` esta nueva prueba:

```typescript
it('responde HTTP 400 ante un descuento inválido', async () => {
  const respuesta = await request(app)
    .post('/api/precios/calcular')
    .send({ precio: 1000, descuento: 150 });

  expect(respuesta.status).toBe(400);
  expect(respuesta.body).toHaveProperty('error');
});
```

El código `400` indica que la petición contiene datos que el servidor no acepta. La ruta de `src/app.ts` ya contempla la respuesta `400` cuando el servicio lanza `RangeError` o `TypeError`.

Ejecutá `npm test` nuevamente. Explicá por qué esta comprobación es de **integración** y no solamente una prueba unitaria.

> **Respuesta:** no se llama a la función directamente, se envía una petición HTTP a la app de Express. Participan la ruta, el parseo del JSON, el servicio y el manejo de errores que traduce el `RangeError` a un 400. Lo que se prueba es que esas partes funcionen juntas. Con esta prueba son 10 en total y todas pasan.

## 8. Diferenciar pruebas y verificación de tipos

Ejecutá:

```bash
npm run typecheck
```

Este comando usa TypeScript para analizar tipos, pero **no ejecuta las pruebas**. Por otra parte, Node puede ejecutar los archivos `.ts` compatibles directamente, pero no comprueba sus tipos durante esa ejecución.

**Pregunta:** ¿por qué conviene utilizar tanto `typecheck` como `npm test`?

> **Respuesta:** detectan problemas distintos. `npm test` ejecuta el código y verifica comportamientos, pero solo los casos que escribimos. `typecheck` no ejecuta nada, analiza los tipos de todo el código; como Node corre los `.ts` borrando los tipos, un error de tipos pasaría desapercibido si no se corre `tsc`. En este proyecto `typecheck` terminó sin errores.

## 9. Integración continua: extensión opcional

El proyecto ya contiene `.github/workflows/pruebas.yml`. Abrí ese archivo e identificá los pasos que ejecutará GitHub Actions: descargar el repositorio, preparar Node, instalar dependencias, comprobar tipos y correr las pruebas.

Si querés probarlo en GitHub:

1. Ejecutá `npm install` antes de subir el repositorio y conservá `package-lock.json` para versionarlo.
2. Subí los archivos del proyecto a un repositorio, **sin** incluir `node_modules`.
3. Entrá a la pestaña **Actions** y revisá la ejecución iniciada con tu `push`.
4. Comprobá qué ocurre si una prueba falla y luego se corrige.

En la versión actualizada del proyecto, el workflow utiliza `npm install`. Después de generar y versionar `package-lock.json`, se recomienda cambiarlo a `npm ci` para obtener instalaciones reproducibles.

> **Resultado:** versioné `package-lock.json` y cambié el workflow a `npm ci`. El workflow está dentro de `demo_practica/.github/`, y GitHub Actions solo lee los que están en el `.github/` de la raíz del repositorio, así que para que se ejecute habría que moverlo a la raíz y agregar `working-directory: demo_practica`.

## 10. Registro de resultados

| Comprobación | Resultado | Observación |
| --- | --- | --- |
| Respuesta de `GET /healthy` | `{"estado":"ok"}` | En el código la ruta es `/salud`. |
| Primera ejecución de `npm test` | 4 pasan, 1 falla | Falla "rechaza descuentos superiores al 100 %": devolvía -500 en vez de lanzar error. |
| Corrección del servicio | 5 pasan | Validación de precio y descuento con `Number.isFinite` y rangos. |
| Nuevas pruebas unitarias | 9 pasan | Descuento 100 %, precio negativo, descuento negativo y `NaN`. |
| Prueba de integración HTTP 400 | 10 pasan | La API responde 400 con la propiedad `error`. |
| Verificación `npm run typecheck` | Sin errores | |

## 11. Preguntas de cierre

1. ¿Qué diferencia hay entre comprobar manualmente una respuesta y ejecutar una prueba automatizada?
2. ¿Cuál fue la causa de la primera prueba fallida?
3. ¿Por qué es útil incluir casos inválidos además de los casos habituales?
4. ¿Qué parte del sistema comprueba `tests/precios.test.ts` y qué partes comprueba `tests/api.test.ts`?
5. ¿Qué ventajas ofrece ejecutar las pruebas después de cada modificación?

> **Respuestas:**
>
> 1. La comprobación manual depende de que alguien la repita y mire el resultado cada vez. La automatizada se corre con un comando, siempre de la misma forma, y avisa sola si algo dejó de funcionar.
> 2. `calcularPrecioFinal` no validaba la entrada: con un descuento de 150 % devolvía un total negativo en lugar de lanzar `RangeError`.
> 3. Porque los errores suelen aparecer en los bordes y en las entradas que no se esperaban. Probando solo casos normales, el problema del 150 % nunca se hubiera visto.
> 4. `tests/precios.test.ts` comprueba solo la función del servicio. `tests/api.test.ts` comprueba la aplicación Express completa: la ruta, el parseo del body, el servicio y los códigos de estado.
> 5. Las regresiones se detectan en el momento, cuando todavía se sabe qué cambio las causó, y se puede modificar el código con más confianza.

**Referencias:** [Vitest](https://vitest.dev/guide/) · [Supertest](https://github.com/ladjs/supertest) · [Node.js: TypeScript](https://nodejs.org/api/typescript.html) · [GitHub Actions](https://docs.github.com/actions)
