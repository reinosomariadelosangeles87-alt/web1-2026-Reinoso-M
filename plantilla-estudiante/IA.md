# Declaración de uso de IA

> Obligatoria en todas las prácticas. Si usaste un asistente, descríbelo aquí con
> precisión. Si NO usaste ninguno, escribe eso y explica cómo resolviste la parte
> más difícil por tu cuenta — también cuenta como declaración válida.
>
> Recuerda: código de IA sin declarar se califica en CERO y no admite reintento.
> Declararlo honestamente NO baja tu nota. Lo que se evalúa es tu capacidad de auditar.

## Herramientas que usé
Deepseek a través de la interfaz de escritorio/web. No usé GitHub Copilot ni Cursor.

## Qué le pedí
Cómo clonar el repositorio en mi computadora.

Cómo crear una rama (feat/practica-00), hacer cambios y subirlos con git push.

Cómo solucionar el error en GitHub donde main y feat/practica-00 aparecían como "idénticos" y no me dejaba abrir el Pull Request.

Cómo editar el archivo README.md

```
```

## Qué me devolvió
Me explicó paso a paso cada comando, por qué debía usarlo, y me ayudó a diagnosticar errores. Por ejemplo, cuando me salió el mensaje "There isn't anything to compare", me explicó que mi rama no tenía cambios porque solo la había creado pero no había editado archivos. Luego me recordó que debía hacer git add ., git commit y git push para ver los cambios reflejados en GitHub.

```javascript
```

## Qué estaba mal
El error más común fue que, al abrir el Pull Request, GitHub mostraba que main y feat/practica-00 eran idénticos. La razón fue que creé la rama pero no edité ningún archivo dentro de ella. Pensé que solo con crear la rama ya podía abrir el PR. No fue hasta que edité el README.md, guardé y subí los cambios que GitHub detectó diferencias y me permitió crear el PR.

También me costó entender que los cambios en Visual Studio Code no se ven automáticamente en GitHub hasta que no hago git push.

## Qué corregí y por qué
 No edité ningún archivo antes de intentar abrir el Pull Request.
 Edité el archivo README.md con mis datos personales, actualicé la tabla de progreso, hice git add ., git commit -m "docs(practica-00): completar perfil de jugador" y git push. Luego, GitHub ya detectó diferencias y pude abrir el PR sin problemas.

```javascript
```

## Qué escribí yo desde cero
No delegué completamente ninguna parte, porque fui yo quien escribió y guardó los cambios en VS Code. La IA me guió, pero yo ejecuté los comandos, edité el archivo manualmente y decidí qué datos poner en mi perfil. La IA me dio plantillas y sugerencias, pero yo ajusté los textos según mi experiencia personal.

## Reflexión
Definitivamente me ahorró tiempo. Sin la IA, probablemente habría pasado horas buscando en la documentación , y no habría entendido por qué no podía abrir el Pull Request. La IA me explicó el error paso a paso y en un lenguaje ás sencillo.

Lo que más valoro fue la ayuda para resolver el problema de "There isn't anything to compare". Aunque la guía de la misión decía que debía crear la rama y hacer el push, no mencionaba explícitamente que debía hacer cambios en la rama para que el PR tuviera sentido. La IA lo detectó rápido y me lo explicó con claridad.

Sí volvería a usarla para futuras misiones, pero tomando más en cuenta  que debo leer con atención los mensajes de error y no asumir que todo está bien. La IA me ayudó a aprender a leer mejor la terminal y los mensajes de GitHub.
