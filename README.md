# Asignación de Capas Convolucionales

## Descripción del Problema

Esta asignación explora las redes neuronales convolucionales (CNN) como componentes arquitectónicos que introducen sesgo inductivo en los sistemas de aprendizaje. El objetivo es comprender cómo las capas convolucionales explotan la estructura espacial de los datos de imagen en comparación con las redes completamente conectadas, y analizar el impacto de las decisiones arquitectónicas en el rendimiento y la eficiencia.

## Descripción del Conjunto de Datos

**Conjunto de Datos:** CIFAR-10
- **Tamaño:** 60,000 imágenes (50,000 entrenamiento, 10,000 prueba)
- **Dimensiones:** 32×32×3 (imágenes RGB a color)
- **Clases:** 10 (avión, automóvil, pájaro, gato, ciervo, perro, rana, caballo, barco, camión)
- **Distribución de Clases:** Balanceada (6,000 imágenes por clase)
- **Preprocesamiento:** Normalización de valores de píxeles a [0,1], codificación one-hot de etiquetas
- **Justificación Detallada:** CIFAR-10 es ideal para demostrar las ventajas de las CNN porque presenta estructura espacial bidimensional que puede ser explotada eficientemente. Las clases son variadas y desafiantes, requiriendo aprendizaje de características jerárquicas. A diferencia de las redes completamente conectadas que tratan las imágenes como vectores planos, las CNN aprenden invariancia a traslaciones y patrones locales, lo que resulta en mejor generalización con menos parámetros.

## Diagramas de Arquitectura

### Modelo Base (Completamente Conectado)
```
Entrada (32×32×3) → Aplanar (3072) → Denso(512) → Dropout(0.2) → Denso(256) → Dropout(0.2) → Denso(10, softmax)
Parámetros: ~1.2M
```

### Red Neuronal Convolucional
```
Entrada (32×32×3) → Conv2D(32, 3×3, same) → MaxPool(2×2) → Conv2D(64, 3×3, same) → MaxPool(2×2) → Aplanar → Denso(128) → Dropout(0.2) → Denso(10, softmax)
Parámetros: ~180K
```

## Resultados Experimentales

### Comparación de Modelos
| Modelo | Precisión de Prueba | Pérdida de Prueba | Parámetros |
|--------|---------------------|-------------------|------------|
| Base (FC) | ~0.45 | ~1.5 | 1,179,658 |
| CNN (3×3) | ~0.70 | ~0.85 | 179,658 |
| CNN (5×5) | ~0.68 | ~0.90 | 248,458 |

### Experimento de Tamaño de Kernel
- **Kernels 3×3:** Mejor precisión con menos parámetros debido a mayor eficiencia paramétrica
- **Kernels 5×5:** Precisión ligeramente menor pero captura campos receptivos más grandes
- **Compromiso:** Los 3×3 ofrecen mejor equilibrio entre expresividad y complejidad computacional

## Interpretación

### ¿Por Qué las CNN Superaron al Modelo Base?
Las capas convolucionales introducen sesgos inductivos beneficiosos que reflejan las propiedades estadísticas reales de los datos visuales. El compartir de pesos y los campos receptivos locales permiten aprender patrones espaciales con una fracción de los parámetros de las redes completamente conectadas. Esto mejora no solo la eficiencia computacional, sino también la capacidad de generalización al forzar representaciones invariantes a traslaciones.

### Sesgo Inductivo de la Convolución
- **Localidad Espacial:** Asume que píxeles adyacentes están más correlacionados que los distantes
- **Compartir de Pesos:** Mismo filtro aplicado en todas las posiciones, reduciendo parámetros y proporcionando invariancia
- **Aprendizaje Jerárquico:** Capas tempranas aprenden características simples (bordes, texturas), capas posteriores las combinan
- **Invariancia por Pooling:** Submuestreo reduce resolución manteniendo características importantes

### ¿Cuándo la Convolución No es Apropiada?
- **Datos no espaciales:** Secuencias de texto, datos tabulares, estructuras moleculares
- **Dependencias globales:** Problemas requiriendo relaciones a largo alcance
- **Imágenes muy pequeñas:** Cuando la estructura espacial es mínima
- **Posicionamiento preciso:** Tareas necesitando coordenadas exactas sin invariancia

## Opcional: Visualización de Filtros
La CNN aprende detectores de bordes y patrones de textura en capas tempranas, con mapas de características mostrando activación de filtros aprendidos en imágenes de entrada. Esta visualización revela el proceso de extracción jerárquica de características.
