# Sesión 1 · Arquitectura web y protocolo HTTP

**Módulo 1** · Zona 1: El Puerto de Entrada
**Lo que sale de esta noche:** tu repositorio creado y tu primer Pull Request fusionado
**Tu misión al terminar:** Misión 01 — Ficha de personaje (100 XP · 7 h de trabajo autónomo)

---

## Por qué empezamos por aquí

Ya sabes programar. Sabes qué es una variable, un ciclo, una función. Lo que probablemente no tengas todavía es un modelo mental de qué pasa cuando un programa deja de correr en una sola máquina y empieza a hablar con otra a través de una red.

Ese modelo es el que vamos a construir hoy, y es la base de todo lo demás. Casi todos los errores confusos que vas a encontrar en el semestre —el navegador que no carga tu imagen, la API que devuelve 401, la página que se ve bien en tu máquina y mal en otra— se explican por algo de esta noche.

Al final de la sesión vas a tener dos cosas: el modelo mental, y tu infraestructura de trabajo funcionando. La segunda parte importa tanto como la primera, porque sin repositorio no hay entregas.

---

## El modelo cliente-servidor

Antes de leer lo que sigue, respondete esto: **cuando escribes una dirección en el navegador y presionas Enter, ¿cuántas máquinas distintas participan antes de que veas la página?**

Piénsalo un segundo. La mayoría dice dos: la tuya y el servidor.

La respuesta real, para un sitio comercial cualquiera, incluye el resolutor DNS, posiblemente varios servidores DNS en cascada, una red de distribución de contenido, un balanceador de carga, uno o varios servidores de aplicación y al menos una base de datos. Entre siete y quince máquinas, según cómo se cuente.

Esa es la primera idea que conviene interiorizar: **la web es un sistema distribuido**, y buena parte de los errores raros vienen de que algo pasa *entre* máquinas y no dentro de una.

### El ciclo, en cuatro movimientos

**Tu navegador pide.** El navegador es un programa cuyo trabajo es pedir recursos y dibujarlos. No sabe nada del servidor más allá de su dirección. Cuando escribes `mi-juego.vercel.app`, primero hay que traducir ese nombre a un número, porque las máquinas se encuentran por número y no por nombre. Eso es el DNS, y funciona como una guía telefónica distribuida: le pregunta a un servidor, que le pregunta a otro, hasta que alguien responde "ese nombre corresponde a esta dirección IP".

**La red enruta.** Con la IP en mano se abre una conexión TCP, que es el protocolo que garantiza que los datos lleguen completos y en orden. Si es HTTPS, encima de TCP se negocia TLS, que cifra la conversación. Aquí hay un detalle que explica algo que quizá ya notaste: ese saludo inicial —el *handshake*— cuesta viajes de ida y vuelta antes de pedir un solo byte de contenido, y por eso HTTPS parece más lento la primera vez.

**El servidor responde.** Recibe la petición, decide qué hacer con ella (consultar una base de datos, leer un archivo, calcular algo) y devuelve una respuesta. Lo que devuelve es texto: HTML, CSS, JavaScript, JSON. **El servidor no dibuja nada**, solo manda instrucciones.

**Tu navegador dibuja.** Lee el HTML, descubre que necesita más cosas (hojas de estilo, imágenes, scripts), pide cada una con una petición nueva, y va construyendo la página. De ahí sale algo que sorprende al medirlo: **una página no es una petición, son decenas**.

### Las dos ideas que hay que llevarse

Estas dos vuelven varias veces en el semestre. Vale la pena entenderlas bien hoy.

**HTTP no tiene estado.** Cada petición es completamente independiente de las anteriores. El servidor, por defecto, no recuerda nada de ti entre una petición y la siguiente. Suena a limitación y es una decisión de diseño deliberada: es lo que permite que la web escale a miles de millones de peticiones, porque cualquier servidor puede atender cualquier petición sin necesitar contexto previo.

La consecuencia práctica es la que te va a importar: si el servidor no recuerda nada, ¿cómo sabe una tienda en línea que el carrito es tuyo? La respuesta son las cookies, los tokens y las sesiones, que son mecanismos construidos *encima* de un protocolo sin estado para simular que sí hay memoria.

Una forma de verlo: HTTP es como escribirle cartas a alguien con amnesia total. Cada carta tiene que incluir todo el contexto necesario, porque quien la recibe no recuerda la anterior.

**Nadie confía en nadie.** El cliente no confía en el servidor y el servidor no confía en el cliente. Toda validación crítica se repite en ambos lados, y eso no es redundancia inútil: la validación del cliente existe para dar buena experiencia (avisarte rápido que el correo está mal escrito) y la del servidor existe para seguridad, porque cualquiera puede saltarse el navegador y mandar la petición a mano con una herramienta de línea de comandos.

Dicho de forma más directa: **todo lo que llega del cliente es una mentira potencial.** En el módulo 4, cuando integres un modelo de lenguaje, vas a aplicar exactamente el mismo principio a la salida del modelo.

### Ponte a prueba

Si puedes responder estas dos sin releer, el bloque quedó:

*Si el servidor no recuerda nada entre peticiones, ¿cómo sabe una red social que sigues con la sesión abierta?*

*¿Por qué es una mala idea validar la contraseña solo en el navegador?*

---

## Práctica en clase: leer peticiones reales

Ya viste el diagrama. Ahora vas a ver la misma cosa en la realidad, en tu propia máquina. Abre las herramientas de desarrollo con **F12** y ve a la pestaña **Red**. Recarga la página y observa la cascada de peticiones apareciendo.

Este recorrido lo hacemos juntos en clase, pero está aquí para que puedas repetirlo solo:

**Mira el documento principal.** Haz clic en la primera petición de la lista, la del documento HTML. Se abre el panel de detalles. Observa los encabezados de la petición: el método (`GET`), la ruta, el `Host`, el `User-Agent` que identifica a tu navegador. Después los de la respuesta: el código de estado, el `Content-Type` que dice qué tipo de contenido es, el `Cache-Control` que dice cuánto se puede guardar. Es exactamente lo que estaba en el diagrama, pero de verdad.

**Cuenta el volumen.** Al pie del panel hay un contador de peticiones y peso total. Antes de mirarlo, adivina cuántas son. Una página de noticias cualquiera hace entre cincuenta y doscientas peticiones. El rendimiento web es en buena parte un problema de cuántas cosas pides y cuán grandes son.

**Ordena por tamaño.** Haz clic en la columna de tamaño. El recurso más pesado casi siempre es una imagen o un archivo de JavaScript. Guarda esa observación: en el módulo 5 la vas a necesitar.

**Provoca un 404.** Añade basura al final de la URL —`/estonoexiste`— y recarga. Vas a ver la línea en rojo con el código 404.

**Provoca un 304.** Recarga la página normal dos veces seguidas. En la segunda, varios recursos aparecen con código 304 (`Not Modified`) y tamaño casi cero: tu navegador ya los tenía y el servidor solo confirmó que siguen siendo válidos. Es caché funcionando, y verlo convence más que leerlo.

**Asómate a la Consola.** Cambia a esa pestaña en cualquier sitio real. Casi siempre hay algún error o advertencia. Ese panel es donde vas a vivir cuando tu código falle, así que conviene familiarizarte hoy.

Dos cosas que probablemente te llamen la atención. Vas a ver peticiones a dominios que no son el del sitio: son CDN, analítica y publicidad de terceros. Y vas a ver peticiones que se repiten cada pocos segundos: son consultas en segundo plano hechas por JavaScript, algo que vas a programar tú mismo en el módulo 3.

---

## Verbos y códigos de estado

### Los verbos

El método de la petición dice qué pretendes hacer con el recurso. Los cinco que importan: `GET` trae un recurso y no debe cambiar nada en el servidor, `POST` crea algo nuevo, `PUT` reemplaza un recurso completo, `PATCH` modifica una parte y `DELETE` elimina.

Lo que hay que entender no son los nombres, que se aprenden usándolos, sino el concepto de **idempotencia**: una operación es idempotente si llamarla diez veces produce el mismo resultado que llamarla una vez. `GET` lo es. `DELETE` también, curiosamente, porque borrar algo ya borrado deja el mismo estado final. `POST` no lo es, porque cada llamada crea algo nuevo.

Esto ya lo viviste: cuando recargas la página después de enviar un formulario y el navegador te pregunta si quieres reenviar los datos, **esa advertencia existe precisamente porque POST no es idempotente** y el navegador sabe que reenviar podría duplicar una compra o un mensaje.

### Los códigos

Cinco familias, de las cuales tres importan hoy.

Los **2xx** dicen que todo salió bien. `200 OK` es el común, `201 Created` es la respuesta correcta a un POST que creó algo, `204 No Content` significa "funcionó y no tengo nada que devolverte".

Los **3xx** son redirecciones. `301` es permanente y le dice al navegador y a los buscadores que actualicen el enlace. `304 Not Modified` es el que acabas de ver en la práctica.

Los **4xx** significan que el cliente se equivocó: `400` es petición mal formada, `401` es "no sé quién eres", `403` es "sé quién eres y no puedes", `404` es "eso no existe", `422` es "entendí tu petición pero los datos no son válidos". La distinción entre 401 y 403 cuesta al principio y vale la pena fijarla: uno es falta de identidad, el otro es falta de permiso.

Los **5xx** significan que el servidor falló. Cuando tu propio código devuelva 500, el problema es tuyo; cuando lo devuelva una API ajena, no.

### El error que más se comete

**Devolver `200 OK` con un cuerpo que dice "error".**

Si el código es 200, el cliente asume que todo salió bien y sigue adelante. Para descubrir que hubo un error tendría que leer el contenido y buscar la palabra "error", lo cual rompe todo el manejo automático: los bloques `catch` no se disparan, los reintentos no funcionan, la caché guarda una respuesta inválida.

El código de estado es un contrato legible por máquinas, no decoración. En la Misión 04 esto se evalúa explícitamente.

---

## Misión 00 · Registro de jugador (25 XP, en clase)

Tu primera entrega es minúscula a propósito: crear el repositorio, configurar Git, escribir tu perfil de jugador y abrir tu primer Pull Request. Se hace completa en clase para que nadie arranque el semestre trabado en la herramienta.

Son seis pasos. El detalle de cada comando lo vemos juntos en el salón, pero este es el mapa:

```bash
# 1. Configurar Git (una sola vez en tu máquina)
git config --global user.name "Tu Nombre"
git config --global user.email "tu.correo@ucaldas.edu.co"
git config --global init.defaultBranch main

# 2. Copiar la plantilla en GitHub con "Use this template"
#    Nombre exacto del repositorio: web1-2026-tuapellido
#    Debe ser PÚBLICO.

# 3. Clonar
git clone https://github.com/tuusuario/web1-2026-tuapellido.git
cd web1-2026-tuapellido
code .

# 4. Rama, cambios y commit
git switch -c feat/practica-00
git add .
git commit -m "docs(practica-00): completar perfil de jugador"
git push -u origin feat/practica-00

# 5. Abrir el Pull Request en GitHub y completar la plantilla

# 6. Después del merge: etiquetar
git switch main && git pull
git tag practica-00-v1 && git push --tags
```

**Dos cosas que causan problemas.** El correo de Git debe ser **el mismo de tu cuenta de GitHub**, o tus commits no se asociarán a tu perfil: no falla nada visible, simplemente tus contribuciones no aparecen, y eso importa cuando se evalúe el trabajo en equipo. Y GitHub ya no acepta contraseña por HTTPS, así que si te pide autenticación necesitas un **token de acceso personal** (Configuración → Developer settings → Personal access tokens) o configurar SSH.

Cuando abras el PR va a aparecer un check amarillo que se vuelve verde o rojo. Eso es **GitHub Actions**: un robot que revisa la estructura de tu entrega antes que el docente. Si está en rojo, tu código no se revisa hasta que lo arregles. Vas a entender cómo funciona por dentro en la sesión 17, pero desde hoy es parte de tu flujo.

Sobre el `IA.md` de esta primera misión: llénalo aunque no hayas usado ningún asistente. Si no lo usaste, dilo y explica cómo resolviste el paso más difícil por tu cuenta. Eso también es una declaración válida, y que la primera sea fácil es deliberado.

---

## Tu misión de la semana: Ficha de personaje (100 XP)

Construyes la ficha de un personaje de videojuego como página HTML, **sin una sola línea de CSS**.

Necesita: nombre, imagen con texto alternativo descriptivo, tabla de estadísticas, lista de habilidades, historia en varios párrafos con jerarquía de encabezados correcta, y un formulario de contacto con etiquetas asociadas a sus campos.

### Por qué la restricción de no usar CSS

Te lo vas a preguntar, así que aquí está la razón completa.

Una página bien marcada semánticamente se ve **razonable** sin ninguna hoja de estilo, porque el navegador aplica estilos por defecto que respetan la estructura: los encabezados salen grandes y en negrita, las listas con viñetas, las tablas con filas y columnas. Una página mal marcada —todo con `div`— se ve como un bloque indiferenciado de texto plano.

Entonces la restricción no es un capricho: es un **test de diagnóstico** que puedes correr tú mismo. Si tu ficha se ve legible sin CSS, tu marcado está bien. Si se ve como una lista de basura, está mal, y lo vas a descubrir sin que nadie te lo diga.

Hay algo más: esa estructura es lo que "ve" un lector de pantalla y lo que "lee" un buscador. La semántica no es estética, es la estructura que consumen todos los agentes que no son un ojo humano mirando una pantalla.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: la página abre y cumple lo pedido | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos de esta misión: el documento pasa el validador del W3C sin errores (5), la jerarquía de encabezados es correcta y sin saltos (5), usas elementos de sección en lugar de `div` genéricos (5), y todas las imágenes y campos de formulario tienen texto alternativo y etiquetas útiles (5).

**Misión secundaria opcional (+25 XP):** consigue 100/100 en accesibilidad con Lighthouse y adjunta la captura. Desbloquea la insignia ♿ Acceso para todos.

**Plazo:** antes de la medianoche del día anterior a la sesión 2. Presupuesto estimado: **7 horas** repartidas en 1 de planear, 4 de marcar, 1 de validar y 1 de documentar. Si te está tomando mucho más, escribe en lugar de perder la noche.

---

## Errores que probablemente vas a cometer

**Creer que el navegador "ejecuta el sitio".** Es fácil llegar con la idea de que el sitio web es un programa que corre en algún lugar y tú lo ves. En realidad tu navegador ejecuta código que le mandaron, en tu máquina, y puedes modificarlo, inspeccionarlo o ignorarlo. Esa distinción es la base de por qué el servidor no puede confiar en el cliente.

**Confundir HTTPS con "seguro" en sentido amplio.** HTTPS garantiza que nadie en el camino lea ni altere la conversación. No garantiza que el sitio del otro lado sea honesto ni que su código no tenga fallos. Un sitio de estafa puede tener candado verde perfectamente.

**Pensar que el 404 lo genera el navegador.** Lo genera el servidor; el navegador solo lo muestra. La diferencia importa cuando en el módulo 4 tengas que devolver códigos desde tus propias rutas de servidor.

**Asumir que una URL corresponde a un archivo en un disco.** En la web moderna casi nunca es así: la ruta es un identificador que el servidor interpreta, y la respuesta suele construirse en el momento. Esto se aclara solo en el módulo 4 con el enrutamiento de Next.js.

---

## Fuentes de esta sesión

- IETF. *RFC 9110: HTTP Semantics*. https://www.rfc-editor.org/rfc/rfc9110.html
- IETF. *RFC 9112: HTTP/1.1*. https://www.rfc-editor.org/rfc/rfc9112.html
- MDN Web Docs. *An overview of HTTP*. https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview

Los RFC son la autoridad final cuando algo se discute. No hace falta leerlos completos, pero conviene saber que existen y que la documentación de MDN es una lectura de ellos, no una fuente independiente.

---

## Antes de la sesión 2

Lee la sección del módulo 1 en la guía del curso y dale una ojeada a la lista de elementos de sección del HTML Living Standard de WHATWG. Diez minutos, no más: la idea es que llegues habiendo visto los nombres de las etiquetas, no que las domines.
