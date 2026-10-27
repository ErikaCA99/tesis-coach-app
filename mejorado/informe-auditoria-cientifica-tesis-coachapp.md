# Auditoría científica de la tesis CoachApp: conformidad con la app y el Reglamento UMSS

**Fecha:** 27 de octubre de 2026 · **Autora de la tesis:** Erika
**Documento auditado:** `tesis-coach-app` (LaTeX completo: capítulos 1–5, cinco subarchivos del capítulo 4, anexos y bibliografía; ≈26.800 palabras)
**Contrastado con:** código de `coach-app` (62 commits), archivos de métricas de `ml/results`, modelos `.tflite`, Reglamento de Titulación UMSS (2026) y, como referencia comparativa, la tesis de Silva Amado (2024).

> Convención de verificación: lo que no pudo confirmarse en una fuente abierta se marca como **no verificado**. Las referencias externas se contrastaron únicamente con las páginas listadas en la sección *Fuentes abiertas*.

---

## 1. Diagnóstico

La tesis tiene un método y unas métricas de IA sólidas y verificables, pero hoy no describe fielmente la app ni cubre todos los requisitos del Reglamento de Titulación UMSS; cinco hallazgos críticos deben corregirse antes de la defensa.

| Dimensión | Estado | Evidencia |
| --- | --- | --- |
| Métricas de IA (reconocimiento y rutinas) | Coherente | Las tablas de la tesis coinciden con los JSON de `ml/results`; 35 clases; modelos de 4.614.220 B y 16.672 B |
| Calidad estática del código | Coherente | `tsc` sin errores y `prettier` conforme; Jest **no verificado** (la ejecución no terminó en el entorno) |
| Arquitectura declarada | Parcial | `Stack.Protected`, `onSnapshot` y *long polling* coinciden; la tesis se contradice sobre la persistencia offline (el código no la activa) |
| Descripción del sistema vs. repositorio | Desactualizada | Faltan Cloud Functions, App Check, proxy USDA, entrenamiento guiado, ExerciseDB, reglas nuevas y los PR #5 y #6; archivos, líneas y pruebas no coinciden |
| Realidad aumentada | Inconsistente | Azuma (1997) exige registro 3D; la app solo superpone una tarjeta sobre la cámara (sin ARCore ni estimación de pose) |
| Cumplimiento del Reglamento | Incompleto | Falta sección de resultados y discusión; la Ficha Resumen describe 3 capítulos; fecha de defensa pendiente |
| Pruebas | Sin ejecutar | SEG-01..15, CP-01..40, usabilidad, dispositivos reales, PDF y Android 8 figuran como pendientes |
| Bibliografía APA 7 | Con defectos | Autores corporativos mal rotulados, «Obtenido de», «s.f.», citas sin entrada y entradas sin cita |
| Afirmación sobre abandono (40–65 %) | No verificada | No aparece literal en Gjestvang et al. (2020) |

**Lectura global.** Los resultados de IA son honestos y reproducibles: la tesis ya informa que el reconocedor acierta 38,5 % (top-1) y 61,4 % (top-3) en un escenario de campo simulado y que el recomendador imita reglas; reconocer esos límites es una fortaleza, siempre que el resumen y las conclusiones mantengan el mismo tono. Los problemas están en la trazabilidad entre el texto y el producto final, no en la calidad del trabajo técnico.

---

## 2. Análisis

Los siete objetivos específicos tienen una implementación identificable en la app, pero la tesis no incluye una matriz objetivo–requisito–prueba que lo demuestre, y cuatro de ellos se cumplen solo en parte.

### 2.1 Trazabilidad de objetivos específicos

La matriz se reconstruyó a partir del código, los JSON de métricas y los documentos del proyecto; los identificadores de requisitos funcionales de la tesis no se mapearon uno a uno.

| Objetivo específico | Implementación en la app | Evidencia | Cumplimiento |
| --- | --- | --- | --- |
| OE1. Guiar sobre el uso correcto de máquinas | Catálogo en Firestore + reconocimiento por cámara (MobileNetV2 + regresión logística, 35 clases, umbral 0,8) + tarjeta superpuesta | `machineRecognition.ts`, `ml/results/recognition/recognition_metrics.json` | Parcial: 38,5 % top-1 y 61,4 % top-3 en campo simulado; sin prueba con fotos tomadas en gimnasios reales |
| OE2. Instruir en la ejecución de ejercicios | Pasos con GIF, ExerciseDB, entrenamiento guiado, un tutorial 3D | Pantalla `train.tsx`, documentos del proyecto | Parcial: contenido instructivo, sin verificación automática de postura |
| OE3. Rutinas personalizadas y estructuradas | Generador por reglas (`decidirClases`, `generarPlanHeuristico`) + recomendador TFLite (MLP) | `ml/results/metrics.json`, `routine_recommender.tflite` | Sí, pero las métricas (≈99,8 %) miden la imitación de las reglas que etiquetaron los datos sintéticos, como la tesis ya advierte |
| OE4. Herramientas de planificación | Plan semanal y racha (`calcularRacha`) | Código y pruebas unitarias | Sí (Jest no verificado en esta auditoría) |
| OE5. Registro de ingesta y calorías | Proxy USDA vía Cloud Functions, `calcularPorcion`, `calcularIMC` | `functions/src/index.mjs`, `quota.mjs` | Sí; el alcance confunde «conteo calórico» con «calculadora de IMC» |
| OE6. Seguimiento del progreso físico | Registro de peso, estatura, edad e IMC con `onSnapshot` | `firestore.rules`, servicios Firebase | Sí |
| OE7. Reportes personalizados | Reportes de progreso y metas | Documento `metas-reportes-oe5-oe7` | Parcial: generación de PDF sin prueba ejecutada |

### 2.2 Afirmaciones de la tesis frente al código

| Tema | La tesis dice | El repositorio muestra | Acción |
| --- | --- | --- | --- |
| Persistencia offline | Se contradice entre capítulos | Sin persistencia offline de Firestore; *long polling* forzado y `AsyncStorage` solo para sesión | Unificar y declararlo como limitación |
| Backend | Cliente–servidor sin detalle de funciones | Cloud Functions, App Check, proxy USDA y cuotas | Documentar y añadir al diagrama |
| Reglas de seguridad | Anexo B y `anexos/codigo/*.rules` | `firestore.rules` y `storage.rules` del repo son más recientes | Reemplazar por las vigentes |
| Tamaño del código y pruebas | Cifras de archivos, líneas y casos | No coinciden con el repo (incluye `tests/security/rules.test.mjs`) | Recontar y fechar el corte |
| Fases de Mobile-D | Día de prueba y día de salida en Inicialización | Las versiones del método que se abrieron ubican los días de planificación, trabajo y entrega dentro de cada fase | Reubicar (ver 2.5) |
| Realidad aumentada | La app usa AR | Una tarjeta 2D sobre la cámara; sin ARCore, sin registro 3D (Azuma, 1997) | Acotar el término o implementar anclaje real |
| Configuración móvil | Prueba en Android 8 pendiente | `app.json` sin `minSdk`; *slug* `cocha-app`; texto de permiso de cámara con codificación dañada | Fijar `minSdk` y corregir el texto |
| Métricas de IA y modelos | Tablas y 35 clases | Coinciden con los JSON y con los 2 archivos `.tflite` | Sin cambios |

### 2.3 Cumplimiento del Reglamento de Titulación

| Requisito | Fuente | Estado en la tesis |
| --- | --- | --- |
| Seis criterios de evaluación: objetivos, dominio temático, rigurosidad técnica y metodológica, calidad del producto, calidad de la memoria, sustentación oral | Art. 64; Anexo G, Tabla 1 | La rigurosidad se sostiene con el análisis de alternativas (Anexo A); falta cerrar pruebas y resultados |
| Soporte digital con código fuente, instaladores, manuales o modelos | Art. 79(c) | Los anexos no incluyen manual de usuario ni guía de instalación |
| Demostración funcional del producto y recursos tecnológicos disponibles | Art. 83(a) y 84 | Riesgo: cámara, reconocimiento e internet dependen del entorno; no hay plan B documentado |
| Exposición máxima de 30 minutos | Art. 85 | Sin observaciones sobre la memoria |
| Ponderación 70 % tribunal y 30 % asignatura | Art. 88; Anexo G | Informativo |
| Revisión máxima de 40 días hábiles | Art. 90 | Planificar la entrega con holgura |
| Ficha Resumen de una página | Anexos del Reglamento | Describe solo 3 capítulos; la tesis tiene 5 |

### 2.4 Comparación con Silva Amado (2024)

Silva Amado (2024) propone un sistema de visión artificial con MediaPipe Pose que calcula ángulos articulares y los compara con umbrales para identificar la ejecución de ejercicios, implementado en Python con OpenCV y Tkinter y evaluado solo de forma cualitativa. CoachApp prioriza otra capa del problema: reconoce **la máquina** con un clasificador de imagen y genera rutinas, pero no estima la pose del usuario.

- **Fortaleza de la tesis:** reporta métricas cuantitativas reproducibles (reconocimiento y rutinas); Silva Amado no cuantifica su desempeño.
- **Brecha frente a esa referencia:** el objetivo «instruir en la ejecución correcta» se aborda con contenido estático, mientras que el antecedente verifica la ejecución con visión; el capítulo 2 debe discutirlo como limitación o trabajo futuro.
- **No verificado:** el marco teórico de Silva Amado (aprox. líneas 538–1355 de la extracción de texto) no se leyó completo.

### 2.5 Verificación de fuentes externas

| Fuente | Qué se abrió | Resultado |
| --- | --- | --- |
| Gjestvang et al. (2020), *Scand J Med Sci Sports*, 30(9), 1796–1805 | Registro del repositorio NIH y la introducción de Gjestvang et al. (2021, *J Sports Sci Med*, 20, 35–44) | Datos bibliográficos confirmados; **la cifra 40–65 % no aparece literal**. La fuente habla de adherencia: menos del 40 % de los socios mantiene ejercicio regular a largo plazo. El resumen de la tesis doctoral de la misma autora indica que la barrera más frecuente fue «prioridad» (p. ej., falta de tiempo) |
| Abrahamsson et al. (2004), *OOPSLA'04 Companion*, pp. 174–175, DOI 10.1145/1028664.1028736 | Registro VTT y versión de arXiv | Referencia confirmada; la versión abierta nombra las fases *set-up, core, core2, stabilize, wrap-up* y no menciona «día de prueba». Los nombres que usa la tesis **no se verificaron** en las fuentes abiertas |
| Azuma (1997), *Presence*, 6(4), 355–385 | Texto del artículo | Confirma las tres características: combina real y virtual, interactivo en tiempo real, registrado en 3D |
| Mifflin et al. (1990), *Am J Clin Nutr*, 51, 241–247 | Ficha del NCI | Cita confirmada; los coeficientes no estaban en la página abierta |
| Sandler et al. (2018), CVPR | Solo la ficha de búsqueda | Registro localizado; texto no abierto |
| Kraemer y Ratamess (2004), *Med Sci Sports Exerc*, 36(4), 674–688 | Solo la ficha de búsqueda | Registro localizado; texto no abierto |
| Schoenfeld (2010), Jaiswal et al. (2023), Kolt y Snyder-Mackler (2007) | Ninguna | **No verificado** |

---

## 3. Observaciones y mejoras

Se identificaron 16 hallazgos; los cinco críticos son los que un tribunal puede usar para objetar el trabajo según los criterios del Art. 64, y todos tienen corrección factible antes de la defensa.

| # | Prioridad | Hallazgo | Evidencia | Mejora propuesta |
| --- | --- | --- | --- | --- |
| 1 | Crítica | La descripción del sistema no refleja la versión actual (Cloud Functions, App Check, proxy USDA, entrenamiento guiado, ExerciseDB) | Cap. 3 y 4 vs. `functions/src`, `firestore.rules` | Actualizar arquitectura, diagramas y conteos con fecha de corte |
| 2 | Crítica | «Realidad aumentada» en título y objetivos: la tesis admite que no hay anclaje 3D, pero cita la definición de Azuma que lo exige | Azuma (1997); app sin ARCore ni pose | Acotar el término en el texto (fragmento F1); el título solo cambia por el trámite del Cap. IX del Reglamento, o implementar anclaje real |
| 3 | Crítica | No existe sección de resultados y discusión | Estructura de caps. 4–5 | Añadir capítulo con métricas, limitaciones y contraste con antecedentes |
| 4 | Crítica | Pruebas sin ejecutar (SEG-01..15, CP-01..40, usabilidad, dispositivos, PDF, Android 8) | Anexo C y `\pendiente{}` | Ejecutarlas o declararlas como alcance futuro; eliminar marcadores antes de imprimir |
| 5 | Crítica | Ficha Resumen describe 3 capítulos y `\fechaDefensa` queda pendiente | `preliminares/03-ficha-resumen.tex` | Reescribirla en una página con los 5 capítulos y la fecha |
| 6 | Alta | Validez de las métricas: la tesis reconoce que el recomendador imita reglas (99,8 %) y que el reconocedor acierta 38,5 % en campo simulado, pero faltan la matriz de confusión, la medición en dispositivo físico y la validación con fotos de gimnasios y con un entrenador | `pruebas.tex`, `05-conclusiones.tex`, `recognition_metrics.json` (`matriz_confusion_campo`) | Incluir la matriz de confusión ya calculada, medir latencia y precisión en un teléfono real y validar las rutinas con un entrenador |
| 7 | Alta | Bibliografía con defectos APA 7: 4 citas sin entrada (Mifflin, Sandler, Cruz, Fernández), 10 entradas sin cita, autores corporativos mal rotulados, «Obtenido de» y «s.f.» | `capitulos/bibliografia.tex`, `borradores/referencias.bib` | Regenerar desde `.bib` con `biblatex-apa`; cruzar citas y lista |
| 8 | Alta | La cifra de abandono 40–65 % y la causa atribuida (falta de guía) no se sustentan en Gjestvang et al. (2020); la referencia además mezcla dos artículos y un enlace de PsycNET | Registro NIH del grupo y texto de Gjestvang et al. (2021); `bibliografia.tex` | Reformular con adherencia (menos del 40 % mantiene ejercicio regular; barrera más frecuente: prioridad o tiempo) o citar una fuente que sí lo sustente (F7) |
| 9 | Alta | Mobile-D: fases y días descritos de forma inconsistente entre capítulos | Cap. 4 subarchivos | Alinear con Abrahamsson et al. (2004) y mantener un solo mapa de fases |
| 10 | Alta | Alcance equipara «conteo calórico» con «calculadora de IMC» | Alcance; `calcularIMC`, `calcularPorcion` | Redactar dos funciones distintas y su límite (no es consejo nutricional) |
| 11 | Alta | Privacidad y ética: la app guarda peso, estatura y edad (desde los 14 años) sin una sección sobre consentimiento, menores y tratamiento de datos | Reglas y esquemas Firestore | Añadir apartado ético y política de privacidad; el marco legal aplicable no se verificó |
| 12 | Alta | Licencias evaluadas en la exploración, pero sin tabla final de atribuciones; autorización de las fotos del catálogo pendiente | `cap4/exploracion.tex`, `05-conclusiones.tex` | Tabla de licencias y atribuciones en anexo |
| 13 | Alta | Anexo B con reglas antiguas | `anexos/codigo/*.rules` vs. repo | Sustituir por las vigentes y referenciar las pruebas de reglas |
| 14 | Media | Plan B para la demostración (Art. 83–84) | Dependencia de cámara, internet y Firebase | Preparar APK instalado, datos de prueba y video de respaldo |
| 15 | Media | Higiene de configuración móvil: `slug`, `minSdk`, texto de permisos | `app.json` | Corregir antes de la compilación final |
| 16 | Media | Jest y ESLint sin evidencia en esta auditoría | Ejecución interrumpida | Ejecutar por suite y registrar la salida en el anexo de pruebas |

**Lo que ya está bien y no debe tocarse:** las tablas de métricas coinciden con los JSON, la arquitectura MVC y la protección de rutas están respaldadas por el código, y el análisis comparativo de alternativas cumple el criterio de rigurosidad técnica.

---

## 4. Versión mejorada

Los textos completos están en la carpeta `tesis-coach-app/mejorado/`; aquí van los extractos principales.

### 4.1 Planteamiento del problema (reemplaza los tres pasajes con Gjestvang)

Gjestvang et al. (2020) realizaron un estudio de seguimiento de un año con nuevos socios de centros de fitness en Noruega para identificar los motivos y las barreras para iniciar y sostener el ejercicio. Investigaciones del mismo grupo señalan que menos del 40 % de los socios de gimnasios logra mantener un ejercicio regular a largo plazo, aunque pocos cancelan su membresía (13,4 % a los 12 meses) (Gjestvang et al., 2021). Estos datos describen un problema de adherencia, no de abandono formal de la membresía.

### 4.2 Realidad aumentada (acota el término sin cambiar el título)

Esta superposición de información digital sobre la imagen en vivo de la cámara corresponde a una forma básica de realidad aumentada: combina lo real y lo virtual y responde a la acción del usuario, pero no cumple la tercera característica definida por Azuma (1997), el registro en tres dimensiones, porque no emplea seguimiento de superficies ni ARCore. En esta memoria, el término se emplea en ese sentido acotado; el anclaje tridimensional se propone como trabajo futuro.

### 4.3 Arquitectura de servidor (añade lo que faltaba)

Además de los servicios de autenticación, base de datos y almacenamiento de Firebase, el sistema incluye dos Cloud Functions invocables (`buscarAlimentos` y `guardarRutina`), desplegadas en `us-central1` con un máximo de cinco instancias y con verificación de App Check obligatoria. La función `buscarAlimentos` actúa como *proxy* de FoodData Central y guarda la clave de la API como secreto del servidor. Ambas rechazan usuarios anónimos y aplican cuotas por usuario (15 búsquedas por minuto y 120 por hora para alimentos; 20 y 200 para rutinas), y la búsqueda de alimentos añade un límite global de 800 por hora.

### 4.4 Alcance (separa dos funciones distintas)

El registro de alimentación se limitará al conteo calórico de los alimentos consumidos; el sistema incluye además una calculadora del índice de masa corporal (IMC) para el seguimiento del progreso físico.

### 4.5 Referencias APA 7 corregidas o añadidas

Abrahamsson, P., Hanhineva, A., Hulkko, H., Ihme, T., Jäälinoja, J., Korkala, M., Koskela, J., Kyllönen, P., & Salo, O. (2004). Mobile-D: An agile approach for mobile application development. En J. Vlissides (Ed.), *OOPSLA'04: Companion to the 19th annual ACM SIGPLAN conference on object-oriented programming, systems, languages, and applications* (pp. 174–175). ACM. https://doi.org/10.1145/1028664.1028736

Azuma, R. T. (1997). A survey of augmented reality. *Presence: Teleoperators and Virtual Environments, 6*(4), 355–385. https://doi.org/10.1162/pres.1997.6.4.355

Gjestvang, C., Abrahamsen, F., Stensrud, T., & Haakstad, L. A. H. (2020). Motives and barriers to initiation and sustained exercise adherence in a fitness club setting—A one-year follow-up study. *Scandinavian Journal of Medicine & Science in Sports, 30*(9), 1796–1805. (DOI por confirmar: 10.1111/sms.13736)

Gjestvang, C., Stensrud, T., Paulsen, G., & Haakstad, L. A. H. (2021). Stay true to your workout: Does repeated physical testing boost exercise attendance? A one-year follow-up study. *Journal of Sports Science and Medicine, 20*, 35–44.

Silva Amado, H. S. (2024). *Sistema de visión artificial para la identificación de ejecución de ejercicios físicos* [Trabajo de grado, Universidad Santo Tomás]. Tunja, Colombia.

Mifflin et al. (1990) y Sandler et al. (2018) estaban citadas en el texto pero ausentes de la lista; se añadieron con sus datos bibliográficos (DOI de Mifflin por confirmar). Cruz (2024) y Fernández (2018) quedan con un marcador, porque sus datos solo los tiene la autora.

### 4.6 Archivos entregados en `tesis-coach-app/mejorado/`

| Archivo | Qué aporta |
| --- | --- |
| `preliminares/03-ficha-resumen.tex` | Ficha Resumen en una página con los seis capítulos, resultados verificados y limitaciones |
| `capitulos/05-resultados-discusion.tex` | Capítulo nuevo: cumplimiento por objetivo, resultados de IA, discusión frente a antecedentes y amenazas a la validez |
| `capitulos/bibliografia.tex` | Lista APA 7 corregida, entradas faltantes añadidas y entradas sin cita retiradas |
| `fragmentos/fragmentos-correcciones.tex` | Siete bloques listos para pegar (RA, Mobile-D, servidor, ética, licencias, alcance, adherencia) |
| `LEEME.txt` | Cómo aplicarlo y qué citas en el texto deben actualizarse |
| `../main-mejorado.tex` | Versión de revisión de `main.tex` que usa lo anterior |

La copia de prueba se compiló sin errores (93 páginas, sin referencias sin resolver en la pasada final) usando inglés como idioma de *babel*, porque el entorno no tenía el paquete de español; en tu equipo debe compilar con el idioma original.

### 4.7 Pendiente de tu parte

Ejecutar las pruebas del Anexo C y la usabilidad, completar Cruz (2024) y Fernández (2018), definir la política de privacidad y el tratamiento de menores, confirmar licencias y autorizaciones de fotos y animaciones, y verificar la denominación de las fases de Mobile-D. Ejecutar Jest y ESLint por suite también quedó sin verificar.

---

## Fuentes abiertas en esta auditoría

- [Azuma (1997), texto del artículo](https://www.hitl.washington.edu/people/tfurness/courses/inde543/READINGS-03/OTHER/Azumapaper.pdf)
- [Abrahamsson et al. (2004), versión de arXiv](https://arxiv.org/pdf/1709.06820) y [registro VTT](https://cris.vtt.fi/en/publications/mobile-d-an-agile-approach-for-mobile-application-development/)
- [Gjestvang et al. (2021), texto completo en JSSM](https://www.jssm.org/jssm-20-35.xml-Fulltext)
- [Registro NIH del grupo de Gjestvang](https://nih.brage.unit.no/nih-xmlui/handle/11250/2979992?show=full)
- [Ficha de Mifflin et al. (1990) en el NCI](https://cancercontrol.cancer.gov/brp/research/group-evaluated-measures/adopt/ree-pal)

Sandler et al. (2018) y Kraemer y Ratamess (2004) solo se localizaron en la búsqueda ([ficha de CVPR](https://openaccess.thecvf.com/content_cvpr_2018/html/Sandler_MobileNetV2_Inverted_Residuals_CVPR_2018_paper.html)); sus textos no se abrieron. El artículo de Gjestvang et al. (2020) no se pudo abrir (bloqueo de verificación en PMC), por eso la cifra 40–65 % se marca como no encontrada en las fuentes revisadas, no como falsa.
