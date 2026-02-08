# Tarea: Capas Convolucionales

## 1. Descripción del Problema y Motivación
En esta tarea, las redes neuronales convolucionales (CNN) se estudian como componentes arquitectónicos que codifican un **sesgo inductivo**, en lugar de tratarlas como modelos de "caja negra". El objetivo es comprender cómo las capas convolucionales explotan la estructura espacial de los datos de imagen y por qué son más adecuadas que las redes totalmente conectadas para tareas de visión artificial.

El enfoque de este trabajo no es el ajuste de hiperparámetros ni alcanzar una precisión de vanguardia (state-of-the-art), sino el **razonamiento arquitectónico**: analizar cómo las decisiones de diseño, como el tamaño del núcleo (kernel), la profundidad, la agrupación (pooling) y la compartición de parámetros, afectan la eficiencia del aprendizaje, la generalización y la interpretabilidad.

## 2. Descripción y Justificación del Dataset
**Dataset:** CIFAR-10  
El conjunto de datos CIFAR-10 se utiliza para todos los experimentos.

### Propiedades del dataset:
* **Imágenes totales:** 60,000
* **Conjunto de entrenamiento:** 50,000 imágenes
* **Conjunto de prueba:** 10,000 imágenes
* **Resolución de imagen:** 32 × 32 píxeles
* **Canales:** 3 (RGB)
* **Número de clases:** 10
* **Distribución de clases:** Equilibrada (6,000 imágenes por clase)

### Por qué CIFAR-10 es apropiado para capas convolucionales:
1. Las imágenes exhiben fuertes **correlaciones espaciales locales**, las cuales los núcleos convolucionales están diseñados para capturar.
2. Los objetos aparecen en diferentes posiciones, lo que hace que la **invarianza a la traslación** sea un sesgo inductivo deseable.
3. El dataset es lo suficientemente complejo como para exponer las limitaciones de las redes totalmente conectadas, pero sigue siendo manejable computacionalmente para experimentos controlados.

Los valores de los píxeles se normalizan en el rango [0, 1] y las etiquetas se codifican mediante *one-hot encoding*. No se aplica preprocesamiento agresivo ni aumento de datos (data augmentation) para mantener el enfoque en los efectos arquitectónicos.

## 3. Análisis Exploratorio de Datos (EDA)
El análisis exploratorio es intencionadamente mínimo y orientado a la tarea. Para las redes neuronales, especialmente las CNN, el objetivo del EDA es comprender la estructura de los **tensores de entrada**, no realizar resúmenes estadísticos exhaustivos.

Se inspeccionan los siguientes aspectos:
* Dimensiones (shapes) de los tensores de entrada y tipos de datos.
* Rango de los valores de los píxeles.
* Inspección visual de muestras representativas de diferentes clases.
* Verificación de la distribución equilibrada de las clases.

Este análisis confirma que las imágenes de CIFAR-10 poseen la estructura espacial que las capas convolucionales están diseñadas para explotar.

## 4. Modelo de Referencia: Red Totalmente Conectada (FCN)

### Arquitectura
Como punto de referencia, se entrena una red neuronal totalmente conectada sin capas convolucionales.
* **Entrada:** (32×32×3)
* **→ Flatten:** (3072)
* **→ Dense:** (512, ReLU)
* **→ Dropout:** (0.2)
* **→ Dense:** (256, ReLU)
* **→ Dropout:** (0.2)
* **→ Dense:** (10, Softmax)
* **Número de parámetros:** ~1.18 millones

### Propósito de la Referencia
Este modelo establece un punto de comparación para resaltar las limitaciones de las arquitecturas no convolucionales en datos de imagen. Al aplanar (*flatten*) la imagen, se destruyen todas las relaciones espaciales entre píxeles, obligando a la red a reaprender los mismos patrones visuales de forma independiente en diferentes ubicaciones.

### Limitaciones Observadas
A pesar de su gran número de parámetros, el modelo totalmente conectado logra un rendimiento de generalización relativamente pobre. Esta ineficiencia ilustra que el conteo de parámetros por sí solo no compensa la ausencia de un sesgo inductivo adecuado.

## 5. Arquitectura de la Red Neuronal Convolucional (CNN)

### Principios de Diseño
La arquitectura convolucional se diseña de acuerdo con los siguientes principios:
* **Campos receptivos locales:** Para capturar patrones localizados espacialmente.
* **Compartición de pesos:** Para reducir el número de parámetros y forzar la invarianza a la traslación.
* **Profundidad incremental:** Para construir representaciones de características jerárquicas.
* **Capas de agrupación (Pooling):** Para introducir robustez ante pequeños desplazamientos espaciales.

### Arquitectura
* **Entrada:** (32×32×3)
* **→ Conv2D:** (32 filtros, 3×3, ReLU, same)
* **→ MaxPooling:** (2×2)
* **→ Conv2D:** (64 filtros, 3×3, ReLU, same)
* **→ MaxPooling:** (2×2)
* **→ Flatten**
* **→ Dense:** (128, ReLU)
* **→ Dropout:** (0.2)
* **→ Dense:** (10, Softmax)
* **Número de parámetros:** ~180,000

Esto representa una reducción significativa de parámetros en comparación con el modelo de referencia, logrando al mismo tiempo un rendimiento sustancialmente mejor.

## 6. Experimento Controlado: Efecto del Tamaño del Núcleo (Kernel Size)

### Configuración Experimental
Se realiza un experimento controlado para estudiar el efecto del tamaño del núcleo convolucional, modificando la siguiente variable:
* **Tamaño del núcleo:** 3×3 vs 5×5

Todos los demás factores se mantienen constantes: número de capas, número de filtros, optimizador, tasa de aprendizaje y procedimiento de entrenamiento.

### Resumen de Resultados
| Modelo | Tamaño del Núcleo | Precisión (Test) | Conteo de Parámetros |
| :--- | :--- | :--- | :--- |
| CNN | 3×3 | Mayor | ~180K |
| CNN | 5×5 | Ligeramente menor | ~248K |

### Observaciones
* Los núcleos más pequeños logran un rendimiento igual o mejor con menos parámetros.
* Apilar núcleos pequeños es más eficiente en parámetros que usar campos receptivos grandes directamente.
* Los núcleos más grandes aumentan el costo computacional sin ganancias proporcionales en la precisión.

## 7. Interpretación y Razonamiento Arquitectónico

### Por qué las CNN superan a la referencia
Las capas convolucionales integran suposiciones previas sobre la estructura de la imagen directamente en la arquitectura del modelo. La conectividad local y la compartición de pesos permiten que la red aprenda un patrón espacial una vez y lo reutilice en toda la imagen.

### Sesgo Inductivo introducido por la Convolución
* **Localidad espacial:** Los píxeles cercanos están más relacionados que los distantes.
* **Invarianza a la traslación:** Las características pueden detectarse independientemente de su posición.
* **Composición jerárquica:** Las características simples se combinan en patrones complejos a través de las capas.
* **Eficiencia de parámetros:** Reducción de la redundancia en comparación con las capas totalmente conectadas.

### Cuándo NO es apropiada la Convolución
* Datos no espaciales (ej. datos tabulares).
* Tareas que requieren relaciones globales explícitas sin localidad.
* Problemas donde se debe preservar la información posicional exacta.

## 8. Despliegue en Amazon SageMaker
El modelo convolucional se entrena utilizando Amazon SageMaker para demostrar el flujo de trabajo de despliegue:
1. Lanzamiento de un trabajo de entrenamiento (*training job*).
2. Guardado de artefactos del modelo.
3. Despliegue del modelo entrenado en un punto de enlace (*endpoint*) de SageMaker.

## 9. Conclusión
Esta tarea demuestra que las elecciones arquitectónicas juegan un papel crítico en el rendimiento de las redes neuronales. Las capas convolucionales introducen sesgos inductivos que se alinean con la estructura de los datos de imagen, resultando en modelos más eficientes e interpretables.
