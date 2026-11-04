# Investigación de la problemática: un tutor conversacional de repaso para universitarios

**Equipo VRAM Squad · IN1050 Herramientas de Procesamiento para IA · Otoño 2026**

Este documento amplía la ficha de la Fase 1. Explica qué problema atiende el proyecto, cómo se resuelve hoy, por qué proponemos un modelo de lenguaje abierto (`Llama-3.2-1B-Instruct`), qué necesita el sistema y qué riesgos éticos implica. Los números entre corchetes, como [1], remiten a la sección de [Referencias](#6-referencias).

## 1. El problema

Un estudiante universitario aprende gran parte de su carrera **fuera del salón**: repasa apuntes, resuelve ejercicios y se prepara para exámenes. Cuando en ese momento aparece una duda (por qué funciona un algoritmo, cómo se despeja una ecuación, qué significa un término), no siempre hay un profesor o un compañero a quien preguntar, sobre todo en semanas de exámenes y entregas. Las asesorías académicas tienen horarios limitados, los profesores atienden grupos grandes y, sin respuesta, el estudiante deja la duda sin resolver o la busca en explicaciones dispersas por internet, de calidad desigual. El proyecto atiende justo ese hueco: **la falta de una primera explicación disponible en cualquier momento durante el repaso**, sin sustituir al profesor.

**A quién afecta.** A los estudiantes de licenciatura en general, y con más fuerza a quienes trabajan, viven lejos del campus o estudian en horarios en los que no hay asesores. También afecta a los profesores, que reciben las mismas preguntas básicas una y otra vez.

**Qué tan grande es.** Los datos oficiales no miden directamente "dudas sin resolver", pero sí muestran la escala del sistema y los problemas de permanencia:

- En el ciclo 2023-2024 se registraron **poco más de 4 millones de estudiantes de nivel superior** en México, según cifras de la SEP [4].
- El abandono escolar en la **educación media superior**, el nivel que alimenta a la universidad, subió de **10.3 % a 11.3 %** entre los ciclos 2019-2020 y 2023-2024, según las Principales Cifras del Sistema Educativo Nacional de la SEP [5].

Estas cifras no prueban que un tutor evite el abandono. Muestran que el acompañamiento académico es un problema con una población grande y que los apoyos actuales no alcanzan a todos. Nuestra hipótesis es más modesta: un tutor disponible a cualquier hora puede reducir el tiempo que un estudiante pasa atorado con una duda.

## 2. Cómo se resuelve hoy

| Alternativa | Qué ofrece | Limitaciones |
|---|---|---|
| **Asesorías y tutorías de la universidad** | Atención de una persona experta, con retroalimentación personalizada. | Horarios fijos, cupo limitado, hay que agendar y no siempre cubren todas las materias. |
| **Videos y cursos en línea** (YouTube, plataformas de cursos) | Explicaciones gratuitas o baratas de casi cualquier tema, a cualquier hora. | No responden preguntas concretas, no se adaptan al nivel del estudiante y la calidad varía mucho. |
| **Tarjetas de repaso (Anki, Quizlet)** | Repetición espaciada para memorizar definiciones y fórmulas. | Sirven para recordar, no para entender; el estudiante debe crear o conseguir las tarjetas. |
| **Chatbots generales en la nube** (servicios de IA con API o cuenta) | Responden casi cualquier pregunta con lenguaje natural. | Dependen de un proveedor externo, pueden requerir cuenta o cobrar por petición, y no vienen configurados como tutor de repaso (nivel universitario, respuestas breves, pedir aclaración). |
| **Tutores de IA comerciales** (por ejemplo, Khanmigo de Khan Academy) | Están diseñados para guiar con preguntas en lugar de resolver la tarea. | Dependen de una plataforma cerrada, no se pueden adaptar al contenido de un curso concreto y suelen estar en inglés primero. |

**Hueco que dejan:** ninguna alternativa combina a la vez *disponibilidad inmediata*, *conversación en español*, *enfoque de repaso* y *control del comportamiento del tutor* sin depender de un proveedor externo. Nuestro proyecto apunta a esa combinación.

## 3. Por qué con IA y por qué este modelo

### Por qué con IA

Un modelo de lenguaje puede mantener una **conversación**: recibe la pregunta tal como el estudiante la escribe, la explica con otras palabras si no se entendió y propone ejemplos nuevos. Eso es lo que no hacen los videos ni las tarjetas. También funciona a cualquier hora y con una paciencia que un asesor humano no puede ofrecer a decenas de estudiantes a la vez. No sustituye al profesor: cubre las dudas rápidas del repaso para que el tiempo con personas se use en lo difícil.

### Por qué `Llama-3.2-1B-Instruct`

Elegimos este modelo por cuatro razones, tomadas de su Model Card [1] y su licencia [2]:

1. **Es una versión de instrucciones (`Instruct`)**: está ajustada para seguir indicaciones y chatear, justo lo que necesita un tutor.
2. **Soporta español** de forma oficial, junto con otros siete idiomas.
3. **Es pequeño** (1.23 B de parámetros): ocupa unos 2.46 GB en 16 bits (1.23 B × 2 bytes) y, con activaciones y KV cache, ronda entre 4 y 6 GB de VRAM en una sesión de chat; la GPU T4 gratuita de Colab tiene 14.56 GB, así que cabe con margen amplio.
4. **Es abierto** y su licencia permite uso comercial y de investigación, con las condiciones de la *Llama 3.2 Community License*.

Su principal límite es su tamaño: los modelos de 1 B de parámetros razonan y recuerdan hechos peor que los grandes. En la propia Model Card, por ejemplo, el modelo de 1 B obtiene 32.2 en MMLU, frente a 58 del de 3 B [1]. Por eso el proyecto lo usa para **repasar temas de forma general**, y no como autoridad en temas avanzados ni para resolver exámenes o tareas calificadas.

### Comparación con un modelo alternativo

| | **Llama-3.2-1B-Instruct** [1][2] | **Qwen2.5-1.5B-Instruct** [3] |
|---|---|---|
| **Parámetros** | 1.23 B | 1.54 B |
| **Licencia** | Llama 3.2 Community License (personalizada, con condiciones de uso) | Apache 2.0 (permisiva) |
| **Idiomas** | 8 oficiales, entre ellos español | Más de 29 idiomas, entre ellos español |
| **Memoria en 16 bits (≈)** | 2.5 GB | 3.1 GB |
| **Ventajas** | Más ligero; documentación y comunidad muy amplias; ficha con evaluaciones publicadas por Meta. | Licencia más libre; más idiomas; suele rendir mejor en tareas de razonamiento y código según su ficha. |
| **Desventajas** | Licencia con más restricciones que Apache 2.0; capacidades limitadas por su tamaño. | Algo más pesado; menor familiaridad del equipo con su ecosistema. |

**Decisión.** Mantenemos Llama-3.2-1B-Instruct porque es el modelo que el equipo eligió en la Fase 1, es el más ligero de la comparación y tiene la documentación más completa. Qwen2.5-1.5B queda como plan B si en las pruebas del Parcial III el modelo principal responde con demasiados errores en español.

### Qué añade el equipo al modelo

La retroalimentación de la Fase 1 pidió definir qué aporta el equipo y qué viene ya en el modelo. El modelo trae la capacidad de conversar; el equipo añade:

- **Un prompt de sistema** que define al tutor: respuestas breves en español, a nivel universitario, que pide aclaración cuando la duda es ambigua.
- **Un modo quiz** que, a petición del estudiante, genera preguntas de práctica sobre el tema explicado para que compruebe si lo entendió.
- **Un aviso de verificación** visible en cada respuesta.

El modelo se cargará en Colab con la librería Transformers de Hugging Face [7].

## 4. Usuarios y requisitos

### Usuarios

- **Usuario principal:** estudiante de licenciatura que repasa una materia por su cuenta. No necesita conocimientos de IA ni instalar nada; solo escribir.
- **Actor relacionado:** el profesor. No usa el sistema, pero sigue siendo quien explica, resuelve lo que el tutor no puede y evalúa; el tutor solo ofrece una primera explicación.

### Requisitos funcionales

1. **RF1.** Recibir una pregunta o tema en texto y en español.
2. **RF2.** Responder con una explicación breve, a nivel universitario, y pedir aclaración cuando la duda sea ambigua.
3. **RF3.** Mantener el hilo de la conversación durante la sesión para que el estudiante pida más detalle o un ejemplo distinto.
4. **RF4.** Ofrecer, a petición del estudiante, un **modo quiz** con preguntas de práctica sobre el tema explicado.
5. **RF5.** Mostrar en cada respuesta, incluidas las del quiz, un aviso de que el tutor puede equivocarse y que debe contrastarse con el material del curso.

### Requisitos no funcionales

1. **RNF1.** Ejecutarse en una GPU T4 de Colab con menos de 6 GB de VRAM en una sesión de chat típica (pesos ≈ 2.46 GB más activaciones y KV cache), dentro de los 14.56 GB de la T4.
2. **RNF2.** Responder en pocos segundos para que la conversación se sienta fluida.
3. **RNF3.** No almacenar datos personales ni guardar las conversaciones fuera de la sesión.
4. **RNF4.** Ser reproducible: cualquier compañero debe poder ejecutarlo siguiendo las instrucciones del repositorio.

## 5. Impacto ético

Tomamos como punto de partida la **Magnífica Humanitas** de la universidad: en cada riesgo nos preguntamos a quién perjudica y quién responde.

**Quién responde si se equivoca (riesgo principal).** Si el modelo explica mal un concepto, el estudiante puede memorizar información falsa y llegar así al examen. El tutor es una herramienta de apoyo; no es un profesor ni una fuente oficial. La responsabilidad del sistema es del **equipo desarrollador**, que debe probarlo, documentar sus límites y corregir las fallas que encuentre. La responsabilidad de contrastar la información con el material del curso es del **estudiante**, y la evaluación sigue siendo del **profesor**. Mitigamos con tres medidas: un aviso visible en cada respuesta de que puede contener errores (RF5), limitar el caso de uso a temas de repaso general (no a la resolución de exámenes o tareas calificadas) y pruebas con preguntas de distintas materias, incluidas las del quiz, registrando los errores. Meta, como desarrollador del modelo, fija además condiciones de uso en su licencia [2].

**Sesgos heredados del modelo.** Llama 3.2 se entrenó con grandes cantidades de texto de internet, donde predominan el inglés y las culturas con más presencia en línea, y la Model Card advierte que las pruebas se hicieron principalmente en inglés [1]. El español de México y los ejemplos locales están menos representados, por lo que el modelo puede usar ejemplos ajenos al contexto del estudiante o expresarse con menos precisión. Los más perjudicados serían los estudiantes con menos recursos para verificar lo que el tutor dice. Por eso el Parcial III incluye evaluar si la calidad del modelo de 1 B en español es suficiente.

**Privacidad de los datos.** Un estudiante puede escribir sus dudas, sus errores o datos personales. Ejecutar el modelo dentro de Colab evita enviar los mensajes a una API de un tercero, aunque Colab corre en servidores de Google, así que los usuarios no deben escribir datos sensibles. El prototipo no guardará conversaciones fuera de la sesión (RNF3).

**Costo energético de la infraestructura.** Una GPU T4 tiene una potencia máxima de diseño de 70 W [6]. Una hora continua equivale a unos 0.07 kWh solo de la GPU, sin contar el resto del servidor. Para un proyecto de clase el impacto es pequeño, pero crece con el número de usuarios y de consultas. Elegir un modelo de 1 B de parámetros en lugar de uno de decenas de miles de millones reduce de forma importante el consumo, y por eso lo consideramos también una decisión ética. Evitaremos dejar sesiones de Colab encendidas sin uso.

## 6. Referencias

1. Meta. *Llama-3.2-1B-Instruct, Model Card*. Hugging Face. <https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct>
2. Meta. *Llama 3.2 Community License*. <https://github.com/meta-llama/llama-models/blob/main/models/llama3_2/LICENSE>
3. Qwen Team. *Qwen2.5-1.5B-Instruct, Model Card*. Hugging Face. <https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct>
4. Revista Dilemas Contemporáneos: Educación, Política y Valores. Artículo que cita a la SEP (2024) sobre la matrícula de nivel superior 2023-2024. <https://dilemascontemporaneoseducacionpoliticayvalores.com/index.php/dilemas/article/download/4752/4560>
5. SEP. *Principales Cifras del Sistema Educativo Nacional*, reportadas por Xataka México: *SEP termina otro ciclo sin saber cuántos alumnos abandonaron la escuela*. <https://www.xataka.com.mx/educacion/sep-termina-otro-ciclo-saber-cuantos-alumnos-abandonaron-escuela-mexico-sigue-medir-tamano-problema>
6. NVIDIA. *Tesla T4 Tensor Core GPU*. <https://www.nvidia.com/en-us/data-center/tesla-t4/>
7. Hugging Face. *Documentación de Transformers*. <https://huggingface.co/docs/transformers>
