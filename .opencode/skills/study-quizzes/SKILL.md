---
name: Study Quizzes
description: Diseñar y sincronizar quizzes de study-review (tipos de pregunta, opciones, modos de respuesta, imágenes SVG y criterios pedagógicos) para los temas de Sol/Primaria, y mantener el documento .md alineado con la plataforma.
---

## Cuándo usar esta skill

Úsala cuando haya que crear, editar o revisar preguntas de quiz para un tema del repo, o cuando haya que sincronizar un documento `.md` de tema con la plataforma study-review mediante el MCP.

## Fuente de verdad

- El documento `.md` del tema es la única fuente del contenido. No inventes preguntas, respuestas ni explicaciones que no estén en él.
- Si el documento y la plataforma no coinciden, la decisión la toma el usuario. Pregunta antes de cambiar uno de los dos.
- Antes de escribir, comprueba que las herramientas están disponibles con `search({ namespace: "study-review" })`. Si faltan `create_quiz` o `update_quiz`, para y avisa: la sesión necesita recargar el MCP.

## Tipos de pregunta del MCP

Cada pregunta de `create_quiz` y `update_quiz` tiene un `type`. Estos son los cuatro tipos disponibles.

### `multiple_choice`

- Opciones: `options` con 2 a 8 entradas. Cada una tiene `text` y `explanation`.
- `correctAnswer` debe coincidir exactamente con el texto de una opción.
- Usa `explanationCorrect` y `explanationIncorrect` solo si aportan algo además de la explicación de cada opción.
- Cuándo usarlo: cuando el alumno debe reconocer la respuesta entre varias alternativas, y los distractores representan errores concretos (cifras invertidas, olvido de unidades, confundir mayor y menor).

### `true_false`

- `correctAnswer`: `"true"` o `"false"`. También acepta `"Verdadero"` o `"Falso"`.
- Usa `explanationCorrect` (por qué la afirmación es correcta o no) y `explanationIncorrect` (qué error se corrige si la respuesta es la contraria).
- Cuándo usarlo: para afirmaciones que contienen un error habitual que el alumno debe detectar. Evita afirmaciones triviales. Un buen ejemplo es "El 81 son 1 decena y 8 unidades" (falso: se confunden las cifras).

### `short_answer`

- `correctAnswer`: el texto de la respuesta, por ejemplo `"dieciséis"` o `"27"`.
- `inputMode`: `"text"` (por defecto) o `"numeric"`. Aplica a `short_answer` y a `fill_blank`.
- No admite respuestas alternativas: solo `correctAnswer`.
- Usa `explanationCorrect` para explicar la respuesta.
- Cuándo usarlo: cuando el alumno debe escribir algo libre, como un número con letras, una palabra o una frase corta.
- Regla de `inputMode`: `"text"` si la respuesta se escribe con letras (por ejemplo "dieciséis"). `"numeric"` si la respuesta son cifras (por ejemplo "27").

### `fill_blank`

Una pregunta de hueco tiene uno o varios huecos marcados con `___` en `prompt`. Hay dos formas.

**Un hueco**

- `correctAnswer`: el valor del hueco, por ejemplo `"93"` o `">"`.
- `options` (opcional): fichas para elegir. Si las das, cada una lleva `text` y `explanation`.
- Ejemplo de enunciado: "El número 46 tiene ___ decenas."

**Varios huecos**

- `blanks`: una entrada por hueco, en el orden en que aparecen en el enunciado. Cada entrada tiene `correctAnswer` y, si aplica, `options`.
- No pongas `correctAnswer` ni `options` a nivel de pregunta cuando uses `blanks`.
- Ejemplo de enunciado: "El 52 tiene ___ decenas y ___ unidades."
- Si usas `options` en un hueco, dalas para todos los huecos de esa pregunta.

**Modo de respuesta (solo `fill_blank`)**

- `answerMode`: `"chips"` (por defecto) o `"keyboard"`.
  - `chips`: el alumno elige entre las fichas. Úsalo para símbolos (`>`, `<`, `=`, `+`, `−`) o cuando hay pocos candidatos y el error que se quiere enseñar está en elegir uno de ellos.
  - `keyboard`: el alumno escribe el valor y no ve las fichas. Úsalo para números que el alumno debe producir (decenas, unidades, resultados). En este modo no se usan `options`.
- `inputMode`: `"numeric"` o `"text"`. Por defecto es `"numeric"` en `fill_blank`. Usa `"text"` solo para símbolos o texto.
- El enunciado debe llevar al menos un `_`. Si falta el hueco, el MCP rechaza la pregunta (por ejemplo, "Observa la figura. La línea tiene ___ segmentos rectos.").

## Imágenes de apoyo (`imageSvg`)

Una pregunta puede llevar una figura incrustada. Es útil para geometría, rectas numéricas o bloques de decenas y unidades.

- `imageSvg`: el SVG en línea. Máximo 500 KB. Empieza por `<svg` y termina en `</svg>`.
- No admite scripts, eventos (`onclick`…), `foreignObject`, `DOCTYPE` ni recursos externos. Va incrustado, así que no hay que alojar el archivo ni usar una URL.
- `imageAlt`: obligatorio si hay `imageSvg`. Es lo que leen los lectores de pantalla y lo que se ve si la imagen no carga.
- El enunciado debe invitar a mirarla: "Observa la figura. …".

Criterios para quizzes:

- El SVG del quiz **no debe llevar texto** que nombre la respuesta. Si la pregunta es identificar el tipo de línea, no escribas "Línea poligonal" dentro de la figura.
- El `imageAlt` tampoco debe revelar la respuesta. Usa etiquetas neutras, como "Figura 1".
- Conviene guardar dos juegos de figuras: uno con etiqueta para el resumen, y otro sin texto y con nombres neutros para las preguntas.
- Un mismo `imageSvg` puede repetirse en varias preguntas.

Ejemplo de pregunta con figura:

```json
{
  "type": "multiple_choice",
  "prompt": "Observa la figura. ¿Es una línea poligonal o una línea curva?",
  "imageSvg": "<svg ...>...</svg>",
  "imageAlt": "Figura 1",
  "correctAnswer": "Una línea poligonal",
  "options": [
    { "text": "Una línea poligonal", "explanation": "¡Sí! Está formada por segmentos rectos unidos. ✔" },
    { "text": "Una línea curva", "explanation": "La figura tiene tramos rectos; no se dobla." }
  ]
}
```

### Imágenes en el resumen

El resumen también admite dibujos, mediante `upsert_summary`:

- `images`: hasta 15 dibujos, cada uno con `name` y `svg`.
- `name`: nombre corto (letras, números, guion o guion bajo) para referenciarlo desde el markdown.
- En `contentMarkdown` se referencian con `![descripción](svg:nombre)`. El texto entre corchetes es obligatorio (texto alternativo).
- Cada dibujo debe referenciarse al menos una vez, y todo `svg:nombre` debe existir. Si no, se rechaza.
- El SVG sigue las mismas reglas que en el quiz (en línea, máx. 500 KB, sin scripts ni recursos externos).
- A diferencia del quiz, en el resumen las figuras sí pueden llevar etiqueta, porque allí se explica el concepto.

## Criterios para elegir el tipo

1. **¿El alumno debe producir un número o un valor?** Usa `fill_blank` con `answerMode: "keyboard"` e `inputMode: "numeric"`.
2. **¿El alumno debe elegir un símbolo (`>`, `<`, `=`)?** Usa `fill_blank` con `answerMode: "chips"` y las tres fichas como `options`.
3. **¿El alumno debe elegir entre varias respuestas que representan errores concretos?** Usa `multiple_choice`. Ejemplo: "¿Qué número tiene 9 decenas y 3 unidades?" con 93, 39 y 90.
4. **¿El alumno debe juzgar una afirmación con un error habitual?** Usa `true_false`.
5. **¿El alumno debe escribir una palabra o una frase libre?** Usa `short_answer`. Si la respuesta son cifras, `inputMode: "numeric"`; si son letras, `"text"`.
6. **¿Hay dos valores que dependen uno del otro (decenas y unidades)?** Usa `fill_blank` con `blanks`.
7. **¿La pregunta necesita una figura para resolverse?** Añade `imageSvg` e `imageAlt` a cualquier tipo. El enunciado empieza por "Observa la figura.".

## Criterios de redacción

- El enunciado debe ser una pregunta directa o una frase completa. Evita construcciones confusas como "9 decenas y 3 unidades son el número ___". Prefiere "El número con 9 decenas y 3 unidades es el ___."
- Cada hueco debe estar claro: un `___` por valor que el alumno tiene que rellenar.
- Una pregunta no debe depender de un detalle que no se ve en el enunciado.
- Usa la redacción exacta del documento. Si el documento no tiene una pregunta, no la añadas sin preguntar.

## Explicaciones

- `explanationCorrect`: qué hace correcta la respuesta. Úsalo en todas las preguntas.
- `explanationIncorrect`: una pista general para cuando el alumno se equivoca. En preguntas `keyboard`, no hay explicaciones por respuesta incorrecta, así que esta pista es lo único que ve el alumno en caso de error.
- En preguntas `chips`, cada ficha lleva su `explanation`, y se muestra cuando el alumno la elige.
- No añadas explicaciones por respuesta incorrecta en preguntas `keyboard`. El usuario lo decidió porque no tenía sentido mostrarlas cuando el alumno no ha visto las opciones.

## Límites del MCP

- `questions`: entre 1 y 100 por quiz.
- `options` en `multiple_choice`: entre 2 y 8.
- `blanks`: entre 1 y 10 por pregunta.
- `imageSvg`: máximo 500 KB por imagen.
- `images` en `upsert_summary`: entre 1 y 15, y cada una debe referenciarse en el markdown.
- `fill_blank` exige al menos un `_` en `prompt`. Si falta el hueco, el MCP rechaza toda la operación.
- `short_answer` no admite respuestas alternativas.
- `update_quiz` reemplaza todo el array de `questions`. Reenvía todas las preguntas, no solo las que cambian.

## Flujo de trabajo para sincronizar un tema

1. Confirma que las herramientas de study-review están disponibles.
2. Lee el documento `.md` del tema completo.
3. Obtén los ids de espacio, materia, tema y quiz con `list_spaces`, `list_subjects`, `list_topics` y `get_topic_content`. No repitas estas búsquedas si el usuario ya te dio los ids.
4. Para crear: `create_quiz` con el título exacto del encabezado del documento y la descripción de la línea siguiente.
5. Para actualizar: `update_quiz` con todas las preguntas del quiz.
6. Verifica con `get_topic_content`: número de quizzes, número de preguntas, tipo, `answerMode`, `inputMode` y si tienen `imageSvg`.
7. Actualiza el documento para que coincida con la plataforma. En cada pregunta, escribe la respuesta correcta, el modo de respuesta si no es el de por defecto, la pista general si existe y la explicación correcta.
8. No toques flashcards ni resúmenes salvo que el usuario lo pida. Si el resumen lleva figuras, usa `images` en `upsert_summary` y referéncialas con `![descripción](svg:nombre)`.
9. No pegues ni repitas el token del MCP.

## Formato del documento

Cada quiz es una sección `## Quiz X: título`, con una descripción y las preguntas numeradas en negrita. Ejemplos de bloques:

```markdown
**1. Completa: El 52 tiene \_\_\_ decenas y \_\_\_ unidades**
- Respuesta: 5 decenas y 2 unidades (teclado numérico, sin fichas)
- Pista si se equivoca: Recuerda que la primera cifra indica las decenas y la segunda las unidades.
- Explicación: ¡Sí! 52 tiene 5 decenas (50) y 2 unidades. ✔

**3. Verdadero o falso: El 27 son 2 decenas y 7 unidades.**
- Verdadero ✔ — 2 decenas son 20, y 20 + 7 = 27.
- Falso — La frase es correcta: 27 tiene 2 decenas y 7 unidades.

**6. ¿Qué número es «veintisiete»? Escríbelo con cifras.**
- Respuesta: 27 (teclado numérico)
- Explicación: «veintisiete» es el 27.

**7. Observa la figura. ¿Es una línea poligonal o una línea curva?**

![Figura](img/quiz/figura-1.svg)

- Una línea poligonal — ¡Sí! Está formada por segmentos rectos unidos. ✔
- Una línea curva — La figura tiene tramos rectos; no se dobla.
```

Las figuras del quiz viven en `img/quiz/`, sin texto y con nombres neutros. En el resumen se usan las versiones con etiqueta de `img/`.

Las preguntas de `chips` pueden mostrar sus fichas con sus explicaciones, como en la pregunta 2 del Quiz A antes de cambiarla a teclado.
