# Sesión 17 · Persistencia en el cliente y CI/CD

**Módulo 5** · Zona 5: La Cumbre
**Lo que sale de esta noche:** entender qué hace por dentro el robot que te ha estado calificando desde la sesión 1, y saber guardar datos en el navegador sin romper el juego para los usuarios nuevos
**Lo que se evalúa hoy:** Quiz 6, el último del semestre (15 XP), y tiempo de proyecto final con asesoría

---

## Por qué esta noche importa más de lo que parece

En el papel el tema es guardar datos en el navegador y entender qué es un pipeline de despliegue. En la práctica el objetivo es otro y es más importante: **que el robot que te ha estado calificando desde la sesión 1 deje de ser una caja negra.**

Llevas dieciséis semanas viendo un check amarillo que se vuelve verde o rojo, y probablemente no tienes idea de qué pasa ahí dentro. Esta noche abres el capó. Cuando entiendas que ese archivo YAML es solo una lista de comandos que corre en una máquina limpia, vas a dejar de temerle y vas a empezar a usarlo a tu favor.

Y hay un dato de calendario: esta es la última sesión antes de las sustentaciones. La segunda mitad de la noche es tiempo de proyecto con asesoría disponible, así que trae dudas concretas y trae el proyecto abierto. Si algo está en problemas, hoy es el día de descubrirlo y no el día de la sustentación.

El Quiz 6 cubre la sesión anterior y la de hoy: por qué Git no puede resolver un conflicto solo, qué se guarda en realidad cuando haces `localStorage.setItem('partida', estado)` con `estado` siendo un objeto, y por qué un comentario de code review que dice "está bien" no sirve. Si las tres te resultan obvias después de leer esto, estás listo.

---

## Guardar datos en el navegador

Antes de leer lo que sigue, respondete esto: **tu juego de memoria guarda el mejor puntaje. Cierras el navegador, lo vuelves a abrir, y el puntaje sigue ahí. ¿Dónde estuvo ese número mientras el navegador estaba cerrado?**

La respuesta intuitiva es "en el servidor", y no es necesariamente cierta: el navegador tiene su propio almacenamiento en disco. Esa es la puerta de entrada al tema.

### Los tres mecanismos

**`localStorage`** guarda pares de clave y valor, en texto, con un límite alrededor de cinco megabytes por origen, y persiste indefinidamente hasta que alguien lo borre. Es el adecuado para preferencias del usuario, el tema claro u oscuro, y partidas guardadas simples.

**`sessionStorage`** tiene exactamente la misma interfaz, con una diferencia: se borra cuando se cierra la pestaña. Sirve para estado temporal y para cachés que solo tienen sentido durante una sesión de juego, como las preguntas de trivia que ya se descargaron.

**`IndexedDB`** es otra cosa: una base de datos real dentro del navegador, con almacenes de objetos, índices, transacciones y capacidad de cientos de megabytes. Y es **asíncrona**, lo cual conecta directamente con todo el módulo 3.

### El patrón y sus dos trampas

Este es el contenido que de verdad importa, porque son los dos errores que aparecen en las entregas todos los semestres.

**La primera trampa: solo guarda texto.**

```javascript
// MAL: guarda la cadena "[object Object]"
localStorage.setItem('partida', estado);

// BIEN: serializar al guardar, deserializar al leer
localStorage.setItem('partida', JSON.stringify(estado));

const guardado = JSON.parse(
  localStorage.getItem('partida') ?? 'null'
);
```

El `?? 'null'` no es adorno: si la clave no existe, `getItem` devuelve `null`, y `JSON.parse(null)` lanza una excepción que rompe la carga del juego la primera vez que alguien lo abre. Y ahí está lo traicionero: **es un error que solo aparece con usuarios nuevos**, así que no lo vas a ver nunca en tu propia máquina, donde ya tienes datos guardados. La forma rápida de probarlo es una ventana de incógnito, que es exactamente simular un usuario que llega por primera vez.

**La segunda trampa: es sincrónico.**

`localStorage` bloquea el hilo principal mientras escribe. Guardar unos cuantos kilobytes es imperceptible; guardar dos megabytes congela la interfaz visiblemente. Y si lo llamas dentro del bucle de juego, en cada fotograma, el juego va a dar saltos.

Conecta con la sesión 9: escribir en `localStorage` es una operación sincrónica que ocupa la pila de llamadas, y mientras está ahí nada más puede correr. Para volúmenes reales existe IndexedDB, que es asíncrona precisamente por esto.

### Las ideas que hay que llevarse

**Nunca guardes ahí nada sensible.** Ni contraseñas, ni tokens de sesión, ni datos personales. Cualquier script que corra en la página —incluido uno inyectado por una dependencia comprometida— puede leer todo el `localStorage` del origen. Es texto plano accesible desde JavaScript, y punto. Si necesitas guardar un token de autenticación, eso va en una cookie con las banderas `HttpOnly` y `Secure`, que JavaScript no puede leer.

**El almacenamiento del cliente no es una base de datos compartida.** Lo que guardas vive en ese navegador, en esa máquina. Si el usuario abre el juego en su teléfono, no está. Suena obvio dicho así, y es el malentendido detrás de la mitad de las dudas sobre la tabla de puntajes global.

### Ponte a prueba

*Si `localStorage` es del navegador, ¿cómo hace la tabla de puntajes global de tu proyecto para mostrar los puntajes de todos?* La respuesta te obliga a distinguir persistencia local de persistencia en servidor: el marcador global necesita un servidor, no hay forma de esquivarlo.

*¿Por qué guardar el token de sesión en `localStorage` es una mala idea aunque sea cómodo?* Porque cualquier script de terceros que cargue tu página puede leerlo, y en un proyecto real cargas más scripts de terceros de los que crees.

---

## Práctica en clase: abrir el capó del pipeline

Este es el tramo con más valor de la noche, y no es sobre almacenamiento. Es sobre desmitificar la automatización que te ha estado calificando todo el semestre. Los pasos están escritos para que puedas repetirlos solo, sobre tu propio repositorio.

**Primero, el archivo.** Abre `.github/workflows/validar.yml` del repositorio plantilla y léelo línea por línea, traduciéndolo a español mientras lo lees: "cuando alguien abra un pull request hacia main, arranca una máquina virtual con Ubuntu, descarga el código, busca las carpetas de práctica, y verifica que cada una tenga su README y su IA.md".

Ahí está el momento de revelación: **es solo una lista de comandos.** No hay magia, no hay servicio misterioso. Es un archivo de texto que dice qué correr y cuándo.

**Segundo, un run real en verde.** Ve a la pestaña Actions de tu repositorio y abre una ejecución exitosa. Despliega cada job y mira la salida. Fíjate en el tiempo que tomó cada paso.

**Tercero, y esto es lo importante, un run en rojo.** Busca una ejecución que haya fallado —seguro tienes varias de las primeras semanas— y ábrela. Aquí está la técnica de lectura de logs que casi nadie te enseña: **se lee de abajo hacia arriba buscando la primera línea que falló**, no la última que aparece. El final del log suele ser ruido de la máquina limpiando; el error real está antes.

Fíjate también en cómo el anotador de GitHub resalta la línea con el error cuando el workflow usa `::error::`, que es exactamente lo que hace el tuyo.

**Cuarto, rompe algo a propósito.** En un repositorio de prueba tuyo, borra el `IA.md` de una práctica y haz push. Vas a ver el check pasar a rojo en tiempo real y vas a poder leer el mensaje de error que produce. Eso cierra el círculo: el robot no es arbitrario, revisa exactamente lo que dice el archivo.

**Quinto, el concepto detrás.** Ahora que viste el mecanismo, ponle nombre. Integración continua es esto: cada cambio se valida automáticamente en una máquina limpia. La frase clave es **"si funciona ahí, funciona en cualquier máquina"**, porque el runner arranca sin nada instalado y sin la configuración particular de tu portátil. Es la respuesta al clásico "en mi máquina funciona".

Y despliegue continuo es el paso siguiente: si todo pasa, publicar. En Vercel eso ya te ocurre solo cada vez que haces merge.

**Dos dudas que te van a surgir.** Si eso cuesta dinero: para repositorios públicos, GitHub Actions es gratis con límites generosos; para privados hay minutos incluidos y luego se paga, que es una buena razón para que tus repos sean públicos. Y por qué no corre en tu máquina: porque corre en un servidor de GitHub, en un contenedor que se crea y se destruye para cada ejecución. Esa efimeridad es la característica, no un defecto.

---

## Serverless, edge y el costo real

Antes de leer lo que sigue, respondete esto: **tu juego está desplegado en Vercel y nadie lo visita durante una semana. ¿Cuánto cuesta mantenerlo encendido?**

La respuesta —prácticamente nada— sorprende si vienes con el modelo mental de un servidor alquilado por mes. Y entender por qué es entender el cambio de modelo que domina el despliegue moderno.

### El cambio de modelo

En el modelo tradicional alquilas una máquina, la enciendes, y pagas por el tiempo que está encendida, la use alguien o no. Tienes que estimar cuánta capacidad necesitas de antemano, y te equivocas en las dos direcciones: pagas de más casi siempre, y te quedas corto justo el día que algo se vuelve popular.

En el modelo **serverless** no hay servidor que administres. Subes funciones, y la plataforma las ejecuta cuando llega una petición. Pagas por ejecución y por tiempo de cómputo. Si nadie entra, no pagas. Si entran diez mil personas a la vez, la plataforma crea diez mil ejecuciones sin que hagas nada.

El nombre es engañoso y vale la pena decirlo: **claro que hay servidores, simplemente no son tu problema.**

El **edge** es la variante geográfica. En lugar de que tu código corra en un centro de datos en Virginia, corre en el nodo más cercano a quien pide: si el usuario está en Manizales, se ejecuta en un nodo de la región y no cruzando el continente. Para contenido estático esto es una CDN clásica; lo nuevo es que ahora también se puede ejecutar código ahí.

### Las ideas que hay que llevarse

**El arranque en frío existe.** Si una función no se ha usado en un rato, la primera petición tiene que esperar a que se inicialice el entorno. Puede ser cientos de milisegundos. Es el precio de no pagar por tiempo inactivo, y es un dato relevante cuando midas rendimiento en tu proyecto final.

**Pagar por uso corta en las dos direcciones.** Un bucle infinito que llama a una función serverless, o una petición a una API de pago dentro de un `useEffect` mal configurado que se dispara en cada render, produce una factura sorpresa. Conecta con lo que viste en la sesión 15 sobre límites de costo: poner topes no es paranoia.

### Ponte a prueba

*Si el código corre en un nodo distinto según quién entre, ¿dónde vive la base de datos?* La pregunta te lleva al problema real de estas arquitecturas: la latencia entre el edge y los datos. El código puede estar cerca del usuario, pero si la base de datos está en Virginia, la ganancia se diluye.

---

## Autochequeo del proyecto final

El tiempo de proyecto de esta noche es para trabajar con asesoría disponible, y conviene que llegues sabiendo en qué estás flojo. Estas tres preguntas son las que importan a una semana de la sustentación, y hay que contestarlas con el proyecto abierto delante, no de memoria.

**¿Está desplegado y se puede jugar ahora mismo?** Si la respuesta es no, esa es la prioridad absoluta y todo lo demás es secundario. Un proyecto que no se puede abrir el día de la sustentación pierde los 35 XP de "funciona" y además tumba la demostración, que es donde arranca todo lo demás. Abre la URL pública en una ventana de incógnito y juega una partida completa. Si no puedes, deja de leer y ponte en eso.

**¿Los tres tenemos commits propios?** Abre el gráfico de contribuciones del repositorio y míralo en equipo, hoy. Es la conversación incómoda que conviene tener ahora y no en la sustentación, porque los 10 XP de trabajo en equipo y buena parte de los 40 de dominio técnico dependen de esto. Si uno de los tres tiene dos commits, ese es un problema de proceso que todavía se puede corregir esta semana y que en la sustentación ya no.

**¿El `AUDITORIA.md` tiene algo escrito?** Casi siempre está vacío o tiene una frase. Vale 30 de los 100 XP de la misión 08 y 20 de los 110 de la sustentación: **es el documento con mayor peso de todo el proyecto.** Si vas a dedicar una hora a algo esta semana que no sea código, dedícasela a este archivo.

---

## Errores que probablemente vas a cometer

**Guardar objetos sin serializar.** `localStorage.setItem('x', miObjeto)` guarda la cadena `"[object Object]"` y vas a pasar media hora sin entender por qué no puedes recuperar tus datos. Falta `JSON.stringify` al guardar y `JSON.parse` al leer. Es el error más frecuente del tema y el más fácil de evitar si te acostumbras a mirar en las herramientas de desarrollo, en la pestaña de almacenamiento, lo que realmente quedó guardado.

**No manejar el caso de la primera visita.** Tu código funciona en tu máquina, donde ya hay datos guardados, y falla para cualquier usuario nuevo porque `getItem` devuelve `null` y `JSON.parse(null)` explota. La solución es un valor por defecto con `??`. Y la forma de detectarlo antes de que lo detecte alguien más es abrir el juego en una ventana de incógnito, que simula un usuario nuevo en dos segundos.

**Creer que el almacenamiento local es compartido entre dispositivos.** Guardas el puntaje en `localStorage`, lo abres en el teléfono, no está, y concluyes que algo se rompió. No se rompió nada: `localStorage` es por navegador y por origen. Una tabla de puntajes global necesita un servidor, y esa es exactamente la razón por la que la Misión 07 existía.

**Tratar el pipeline como algo mágico o arbitrario.** Cuando el check sale rojo, la reacción común es hacer push otra vez a ver si "ahora sí pasa", en lugar de leer el log. Después de esta noche ya no tienes excusa: el log dice exactamente qué falló y en qué línea, se lee de abajo hacia arriba buscando la primera falla, y el archivo YAML que decide todo lo puedes abrir y leer cuando quieras.

---

## Fuentes de esta sesión

- MDN Web Docs. *Web Storage API*. https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
- MDN Web Docs. *IndexedDB API*. https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
- GitHub. *Quickstart for GitHub Actions*. https://docs.github.com/en/actions/get-started/quickstart
- Vercel. *Deployments*. https://vercel.com/docs/deployments

---

## Antes de la sesión 18

Lee la sección de rendimiento de `GUIA-DEL-CURSO.md` y el artículo *Web Vitals* de Google en web.dev. Diez minutos. Y una tarea concreta que sí hay que hacer antes de llegar: **corre Lighthouse sobre tu propio proyecto y trae la captura del informe.** En la sesión 18 se trabaja sobre esos resultados, así que quien no lo traiga va a mirar el de otro.

Y lleva el proyecto desplegado, con una pestaña abierta y probada. Nada de "es que el wifi", nada de instalar dependencias en el momento.
