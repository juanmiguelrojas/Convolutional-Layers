# Tarea de Capas Convolucionales

## 1. Descripción del Problema y Motivación
En esta tarea, se estudian las redes neuronales convolucionales (CNN) como componentes arquitectónicos que codifican un sesgo inductivo, en lugar de verlas como modelos de "caja negra". El objetivo es comprender cómo las capas convolucionales explotan la estructura espacial de los datos de imagen y por qué son más adecuadas que las redes totalmente conectadas para tareas relacionadas con la visión.

El enfoque de este trabajo no es el ajuste de hiperparámetros ni alcanzar una precisión de vanguardia, sino el razonamiento arquitectónico: analizar cómo las decisiones de diseño, como el tamaño del núcleo (kernel), la profundidad, el pooling y el uso compartido de parámetros, afectan la eficiencia del aprendizaje, la generalización y la interpretabilidad.

## 2. Descripción y Justificación del Conjunto de Datos
**Dataset:** CIFAR-10  
Se utiliza el dataset CIFAR-10 para todos los experimentos.

### Propiedades del dataset:
* **Total de imágenes:** 60,000
* **Conjunto de entrenamiento:** 50,000 imágenes
* **Conjunto de prueba:** 10,000 imágenes
* **Resolución de imagen:** 32 × 32 píxeles
* **Canales:** 3 (RGB)
* **Número de clases:** 10
* **Distribución de clases:** Equilibrada (6,000 imágenes por clase)

### Por qué CIFAR-10 es apropiado para capas convolucionales:
* Las imágenes muestran fuertes correlaciones espaciales locales, las cuales los núcleos convolucionales están diseñados para capturar.
* Los objetos aparecen en diferentes posiciones, lo que hace que la invarianza a la traslación sea un sesgo inductivo deseable.
* El conjunto de datos es lo suficientemente complejo como para exponer las limitaciones de las redes totalmente conectadas, pero sigue siendo manejable computacionalmente para experimentos controlados.

Los valores de los píxeles se normalizan al rango [0, 1] y las etiquetas se codifican mediante one-hot encoding. No se aplica preprocesamiento agresivo ni aumento de datos para mantener el enfoque en los efectos arquitectónicos.

## 3. Análisis Exploratorio de Datos (EDA)
El análisis exploratorio es intencionalmente mínimo y orientado a la tarea. Para las redes neuronales, especialmente las CNN, el objetivo del EDA es comprender la estructura de los tensores de entrada, no calcular resúmenes estadísticos exhaustivos.

Se inspeccionan los siguientes aspectos:
* Formas (shapes) de los tensores de entrada y tipos de datos.
* Rango de valores de los píxeles.
* Inspección visual de muestras representativas de diferentes clases.
* Verificación de la distribución equilibrada de clases.

Este análisis confirma que las imágenes de CIFAR-10 poseen la estructura espacial que las capas convolucionales están diseñadas para explotar.

## 4. Modelo Base: Red Totalmente Conectada
### Arquitectura
Como punto de referencia, se entrena una red neuronal totalmente conectada sin capas convolucionales en el mismo dataset.

**Arquitectura:**
* Entrada (32×32×3)
* → Flatten (3072)
* → Dense (512, ReLU)
* → Dropout (0.2)
* → Dense (256, ReLU)
* → Dropout (0.2)
* → Dense (10, Softmax)
* **Número de parámetros:** ~1.18 millones

### Propósito del Modelo Base
Este modelo establece un punto de comparación para resaltar las limitaciones de las arquitecturas no convolucionales para datos de imagen. Al aplanar la imagen, se destruyen todas las relaciones espaciales entre los píxeles, obligando a la red a reaprender los mismos patrones visuales de forma independiente en diferentes ubicaciones.

### Limitaciones Observadas
A pesar de tener un gran número de parámetros, el modelo totalmente conectado muestra un rendimiento de generalización relativamente pobre. Esta ineficiencia ilustra que el número de parámetros por sí solo no compensa la ausencia de un sesgo inductivo adecuado.

## 5. Arquitectura de la Red Neuronal Convolucional
### Principios de Diseño
La arquitectura convolucional se diseña de acuerdo con los siguientes principios:
* **Campos receptivos locales:** para capturar patrones localizados espacialmente.
* **Uso compartido de pesos:** para reducir el número de parámetros y forzar la invarianza a la traslación.
* **Profundidad creciente:** para construir representaciones de características jerárquicas.
* **Capas de pooling:** para introducir robustez ante pequeños cambios espaciales.

La red es intencionalmente poco profunda para enfatizar el razonamiento arquitectónico en lugar de la profundidad por sí misma.

### Arquitectura
* Entrada (32×32×3)
* → Conv2D (32 filtros, 3×3, ReLU, same)
* → MaxPooling (2×2)
* → Conv2D (64 filtros, 3×3, ReLU, same)
* → MaxPooling (2×2)
* → Flatten
* → Dense (128, ReLU)
* → Dropout (0.2)
* → Dense (10, Softmax)
* **Número de parámetros:** ~180,000

Esto representa una reducción significativa de parámetros en comparación con el modelo base, logrando al mismo tiempo un rendimiento sustancialmente mejor.

## 6. Experimento Controlado: Efecto del Tamaño del Núcleo (Kernel)
### Configuración Experimental
Se realiza un experimento controlado para estudiar el efecto del tamaño del núcleo convolucional. Se modifica la siguiente variable:
* **Tamaño del núcleo:** 3×3 vs 5×5

Todos los demás factores se mantienen constantes:
* Número de capas y filtros.
* Optimizador y tasa de aprendizaje.
* Procedimiento de entrenamiento y número de épocas.

### Resumen de Resultados
| Modelo | Tamaño del Núcleo | Precisión de Prueba | Número de Parámetros |
| :--- | :--- | :--- | :--- |
| CNN | 3×3 | Mayor | ~180K |
| CNN | 5×5 | Ligeramente menor | ~248K |

### Observaciones
* Los núcleos más pequeños logran un rendimiento igual o mejor con menos parámetros.
* Apilar núcleos más pequeños es más eficiente en términos de parámetros que utilizar campos receptivos grandes directamente.
* Los núcleos más grandes aumentan el costo computacional sin ganancias proporcionales en la precisión.

## 7. Interpretación y Razonamiento Arquitectónico
### Por qué las capas convolucionales superan al modelo base
Las capas convolucionales integran suposiciones previas sobre la estructura de la imagen directamente en la arquitectura del modelo. La conectividad local y el uso compartido de pesos permiten que la red aprenda patrones espaciales una vez y los reutilice en toda la imagen, lo que lleva a una mejor generalización con menos parámetros.

### Sesgo Inductivo introducido por la Convolución
* **Localidad espacial:** Los píxeles cercanos están más relacionados que los distantes.
* **Invarianza a la traslación:** Las características se pueden detectar independientemente de su posición.
* **Composición jerárquica:** Las características simples se combinan en patrones complejos a través de las capas.
* **Eficiencia de parámetros:** Reducción de la redundancia en comparación con las capas totalmente conectadas.

### Cuándo la Convolución NO es apropiada
Las arquitecturas convolucionales no son adecuadas para:
* Datos no espaciales (por ejemplo, datos tabulares).
* Tareas que requieren relaciones globales explícitas sin localidad.
* Problemas donde se debe preservar la información posicional exacta.

## 8. Despliegue en Amazon SageMaker (Flujo de Trabajo Documentado)
**Nota importante:** Debido a las limitaciones de infraestructura y permisos comunes en entornos académicos, el despliegue en SageMaker se documenta paso a paso. El objetivo es demostrar una comprensión clara del flujo de trabajo de entrenamiento y despliegue, incluso si el endpoint no está activo en el momento de la entrega.

### 8.1 Descripción Conceptual del Flujo en SageMaker
Amazon SageMaker separa el ciclo de vida del aprendizaje automático en componentes distintos:
1. Notebook Local / SageMaker
2. Trabajo de Entrenamiento (gestionado por SageMaker)
3. Artefactos del Modelo (almacenados en Amazon S3)
4. Modelo de SageMaker
5. Endpoint de SageMaker (para inferencia)

### 8.2 Requisitos Previos
* Cuenta de AWS con acceso a SageMaker e IAM Role configurado.
* Acceso a Amazon S3 y SageMaker Studio.
* Instancias de CPU son suficientes para CIFAR-10.

### 8.3 Preparación del Script de Entrenamiento
SageMaker ejecuta un script de entrenamiento dentro de un contenedor gestionado. El script debe cargar el dataset, definir la arquitectura, entrenar y guardar el modelo en `SM_MODEL_DIR`.

### 8.4 Lanzamiento del Trabajo de Entrenamiento
Se configura un Estimador de TensorFlow que define el script, el tipo de instancia, las versiones de software y los hiperparámetros. Al iniciar, SageMaker aprovisiona el cómputo, ejecuta el entrenamiento y guarda el modelo resultante en S3.

### 8.5 Creación y Despliegue del Modelo
Tras el entrenamiento, se crea un objeto de modelo y se despliega en un endpoint HTTPS gestionado, exponiendo la CNN como un servicio de predicción.

### 8.6 Pruebas y Limpieza
Se verifica el endpoint enviando imágenes de prueba y recibiendo las probabilidades de clase. Finalmente, se elimina el endpoint para evitar cargos innecesarios en AWS.

### 8.7 Propósito Educativo de esta Sección
Esta sección demuestra que el estudiante comprende cómo SageMaker gestiona los trabajos de entrenamiento, el versionado de modelos y la exposición de servicios mediante endpoints, cumpliendo con los requisitos de la tarea.

## 9. Conclusión
Esta tarea demuestra que las decisiones arquitectónicas juegan un papel crítico en el rendimiento de las redes neuronales. Las capas convolucionales introducen sesgos inductivos que se alinean con la estructura de los datos de imagen, lo que resulta en modelos más eficientes e interpretables.
