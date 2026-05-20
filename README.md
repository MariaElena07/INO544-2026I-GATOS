# DO YOU TENSORFLOW? — Ves lo que veo 👁️🤖
## IUJO — Feria de Haceres Período I-2026
### Unidad Curricular: INO-544 (Investigación de Operaciones)

---

## 👥 Integrantes y Roles

| Integrante | Cédula | Rol |
|---|---|---|
| María Elena Pérez Silva | V-27.487.703 | Dataset · Preprocesamiento · Modelado · Entrenamiento · Exportación ONNX · Pruebas |

---

## 🎯 1. Clase/Tema Seleccionado

- **Tema asignado:** Gatos — Reconocimiento de Razas
- **Descripción del Objeto:** El modelo identifica si una imagen contiene un gato y,
  en caso afirmativo, clasifica su raza entre 12 razas posibles del dataset Oxford-IIIT Pet.
  Las características visuales clave son: forma del cráneo, patrón del pelaje, color, textura
  y proporción facial, que varían notablemente entre razas como Abyssinian, Bengal, Maine Coon,
  entre otras.

---

## 📊 2. Gestión del Dataset (Ingeniería de Datos)

- **Cantidad de imágenes originales recopiladas:** Oxford-IIIT Pet Dataset — 12 razas de gatos, 200 imágenes por raza.
- **Cantidad de imágenes originales recopiladas:** [AGREGAR TOTAL DESPUÉS DEL ENTRENAMIENTO]
- **Estrategia de Data Augmentation aplicada:**
  - *Volteo Horizontal:* Aplicado a todas las imágenes de entrenamiento
  - *Rotación:* Rango de -20° a +20°
  - *Zoom:* 20%
  - *Desplazamiento (Width/Height Shift):* 10% lateral y vertical
  - *Cambio de Brillo:* Rango de 0.8 a 1.2
-**Total de imágenes generadas para el entrenamiento:** 8,608
- **Resolución y formato estandarizado:** 224×224 píxeles, RGB, float32 — Tensor: `[1, 224, 224, 3]`
- **Split:** 80% entrenamiento / 20% prueba

---

## 🧠 3. Arquitectura del Modelo y Entrenamiento

- **Framework:** TensorFlow 2.x / Keras
- **Descripción de la Red (CNN):**
  se diseñó una cabeza de clasificación personalizada con la siguiente estructura secuencial: una capa de reducción global (GlobalAveragePooling2D), una capa densa intermedia (Dense con 256 neuronas y activación ReLU), normalización por lotes (BatchNormalization) para estabilizar el aprendizaje, y una capa de apagado aleatorio (Dropout al 50%) para evitar que el modelo memorice las imágenes. A esto le sigue una segunda capa densa (Dense con 128 neuronas y ReLU), otro Dropout al 30%, y finalmente la capa de salida (Dense con 13 neuronas y activación Softmax) para clasificar las 13 categorías. El entrenamiento se realizó en dos fases: primero con la red base congelada y luego mediante un ajuste fino (fine-tuning) liberando las últimas 30 capas del extractor.

- **Hiperparámetros:**

| Parámetro | Fase 1 (cabeza) | Fase 2 (fine-tuning) |
|---|---|---|
| Función de pérdida | Categorical Crossentropy | Categorical Crossentropy |
| Optimizador | Adam | Adam |
| Tasa de Aprendizaje | 1e-3 | 1e-5 |
| Épocas máximas | 15 | 10 |
| Batch Size | 32 | 32 |
| Early Stopping | patience=3 | patience=4 |
| Regularización L2 | 1e-4 | 1e-4 |

### 💡 Justificación Crítica (Control de Autoría)

La tasa de aprendizaje se eligió estratégicamente según cada fase del entrenamiento. En la Fase 1 se utilizó un learning rate de 1e-3 porque solo se entrenaban las capas nuevas añadidas al modelo, las cuales iniciaban con pesos aleatorios y necesitaban aprender rápidamente los patrones generales de las 13 clases. En la Fase 2 se redujo a 1e-5 al descongelar las últimas 30 capas del modelo preentrenado, ya que un valor alto podía alterar bruscamente el conocimiento previamente aprendido. Este cambio tuvo un impacto positivo en las gráficas de pérdida: el loss y el val_loss comenzaron a bajar de forma más estable y controlada, pasando de 0.1657 a 0.1622 y de 0.2607 a 0.2377 respectivamente. Además, el Early Stopping evitó overfitting al detener la Fase 1 en la época 8 y restaurar los mejores pesos obtenidos. 


## 📈 4. Métricas de Rendimiento (Testing — 20%)

- **Precisión final (Accuracy) en data de test:** 94.32%
- **Pérdida final (Loss) en data de test:** 0.2378

![Gráfica de Entrenamiento Final](cat_recognition\models\curvas_v4.png)

---

### 🔄 Evolución del Modelo y Gráficas (Versiones 1 a 4)

El rendimiento final se logró tras un proceso iterativo de experimentación. Haz clic en cada versión para desplegar su descripción y gráfica de entrenamiento correspondientes:

<details>
<summary><b>📊 Versión 1 — Modelo Base Convencional (Precisión: ~82%)</b></summary>
<p>

Se entrenó la red base congelada conectada a una capa densa simple de salida. El modelo mostró un aprendizaje rápido pero con un techo de precisión bajo debido a la falta de capas intermedias para procesar las 13 clases de forma óptima.

![Gráfica Versión 1](cat_recognition\models\curvas_v1.png)

</p>
</details>

<details>
<summary><b>📊 Versión 2 — Rediseño de la Cabeza de Clasificación (Aumento de Precisión)</b></summary>
<p>

Se expandió la arquitectura añadiendo bloques densos más complejos (`Dense 256` y `Dense 128`). La precisión aumentó significativamente, pero las curvas de entrenamiento y validación comenzaron a separarse, mostrando los primeros síntomas de sobreentrenamiento (*overfitting*).

![Gráfica Versión 2](cat_recognition\models\curvas_v2.png)

</p>
</details>

<details>
<summary><b>📊 Versión 3 — Control de Overfitting (Curvas Estabilizadas)</b></summary>
<p>

Se integraron capas de `Dropout` (0.5 y 0.3) junto con `BatchNormalization` y aumento de datos (*Data Augmentation*) en tiempo real. Esta versión estabilizó por completo las curvas de pérdida, asegurando que el modelo aprendiera características generales y no memorizara las imágenes.

![Gráfica Versión 3](cat_recognition\models\curvas_v4.png)

</p>
</details>

<details>
<summary><b>📊 Versión 4 — Ajuste Fino Definitivo (Fine-Tuning Final: 94.32%)</b></summary>
<p>

Partiendo de la estabilidad de la V3, se descongelaron las últimas 30 capas del extractor y se aplicó un entrenamiento de alta precisión con una tasa de aprendizaje ultra baja ($3 \times 10^{-6}$). Esta configuración permitió especializar la red en los rasgos finos de las razas de gatos, logrando el resultado definitivo.

![Gráfica Versión 4](src/curvas_final.png)

</p>
</details>

---

---

## ⚙️ 5. Especificación de Exportación ONNX

El modelo se ha homologado bajo los estándares requeridos por la interfaz centralizada:

* **Nombre del archivo:** `cat_recognition\models\ReconocimientoDeGatos_v4.onnx`
* **Tensor de Entrada (Input Shape):** `[1, 224, 224, 3]` (Tipo: `float32`)
* **Tensor de Salida (Output Shape):** `[1, 1]` (Tipo: `float32`)
* **Función de activación final:** Sigmoide (Rango de salida de 0.0 a 1.0 para conversión a porcentaje).

**Lógica del wrapper:**
- Si la clase ganadora es un **gato** → `confidence_score` = confianza de esa raza (≈ 1.0)
- Si la clase ganadora es **no_gato** → `confidence_score` = 1 − confianza de no_gato (≈ 0.0)

**Nota:** El modelo internamente clasifica 13 clases (12 razas + no_gato)
con activación Softmax. El wrapper exportado adapta esta salida al formato binario
`[1,1]` solicitado.

---

## 🚀 6. Instrucciones de Ejecución Local
Para replicar el preprocesamiento y el entrenamiento del modelo:

```bash
# 1. Clonar el repositorio
git clone https://github.com/MariaElena07/INO544-2026I-GATOS.git
cd ReconocimientoDeGatos

# 2. Crear entorno virtual con Python 3.11
python -m venv venv311
venv311\Scripts\activate        # Windows
# source venv311/bin/activate   # Linux/Mac

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Ejecutar la aplicación
cd cat_recognition
python app.py

# 5. Abrir en el navegador
# http://localhost:5000
```

**Requisitos principales:** Python 3.11, TensorFlow 2.x, Flask, OpenCV, NumPy < 2

---