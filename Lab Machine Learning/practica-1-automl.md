# Práctica 1 · Entrenar un modelo con ML automatizado en Azure Machine Learning

**Duración:** 30 minutos (el entrenamiento se ejecuta en segundo plano)
**Nivel:** introductorio, sin código
**Qué aprenderás:** el ciclo *datos → trabajo de entrenamiento → comparación de modelos → mejor modelo* en Azure Machine Learning Studio.

## Escenario

Una empresa de telecomunicaciones quiere predecir qué clientes se van a dar de baja (`baja = Si/No`) a partir de su antigüedad, su cuota, sus llamadas a soporte, sus incidencias, el tipo de contrato y el método de pago.

Es un problema de **clasificación**: el modelo aprende de datos históricos y devuelve una de dos clases.

> Los datos de `clientes-baja.csv` son **sintéticos**: se han generado con reglas sencillas para esta práctica. No son datos reales.

## Requisitos previos

- Un workspace de Azure Machine Learning ya creado (ver la práctica anterior).
- Rol de **Contributor** o **Owner** sobre el grupo de recursos.
- El archivo [`clientes-baja.csv`](./clientes-baja.csv) en tu equipo.
- **Cuota de cómputo** disponible en la región del workspace para crear un clúster de CPU. Si no la tienes, el trabajo no podrá arrancar (ver *Problemas frecuentes*).

## El dataset

| Columna | Tipo | Descripción |
|---|---|---|
| `antiguedad_meses` | número | Meses como cliente |
| `cuota_mensual` | número | Cuota mensual en euros |
| `llamadas_soporte` | número | Llamadas a soporte en el último periodo |
| `incidencias_trimestre` | número | Incidencias abiertas en el trimestre |
| `tipo_contrato` | texto | `Mensual`, `Anual` o `Bienal` |
| `metodo_pago` | texto | `Tarjeta`, `Domiciliacion` o `Transferencia` |
| `baja` | texto | **Columna a predecir**: `Si` o `No` |

600 filas, sin valores vacíos. Alrededor del 42 % son bajas.

---

## Paso a paso

### 1. Abrir Azure Machine Learning Studio

1. Entra en <https://ml.azure.com> (o, desde el Portal de Azure, abre tu workspace y pulsa **Launch studio**).
2. Selecciona tu suscripción y tu workspace.

### 2. Crear el trabajo de ML automatizado

1. En el menú izquierdo, ve a **Authoring → Automated ML**.
2. Pulsa **New Automated ML job**.
3. En **Training method**, elige **Train automatically** y pulsa **Start configuring job**.
4. En **Basic settings**, elige **Create new** y escribe el nombre del experimento: `practica1-bajas`.
5. Pulsa **Next**.

### 3. Subir los datos

1. En **Task type & data**, en *Select task type* elige **Classification**.
2. En *Select data*, pulsa **Create** y rellena los formularios:

   | Formulario | Valor |
   |---|---|
   | Data type | Nombre: `clientes-baja` · Type: **Tabular** |
   | Data source | **From local files** |
   | Destination storage type | Deja el predeterminado (`workspaceblobstore`) |
   | File or folder selection | **Upload files or folder → Upload files** → elige `clientes-baja.csv` |
   | Settings | File format: **Delimited** · Delimiter: **Comma** · Encoding: **UTF-8** · Column headers: **All files have same headers** |
   | Schema | Revisa que las columnas numéricas se detectan como números y las demás como texto |
   | Review | **Create** |

3. Selecciona el activo `clientes-baja` en la lista, revisa la pestaña **Preview** y pulsa **Next**.

### 4. Configurar la tarea

1. En **Task settings**, en *Target column* elige **`baja`**.
2. Abre **View additional configuration settings** y revisa:
   - **Primary metric:** `AUCWeighted` (la métrica por defecto para clasificación binaria es válida).
   - **Explain best model:** actívalo (opcional, útil para el paso 8).
   - **Blocked models:** ninguno.
3. Pulsa **Save**.
4. En validación, elige **k-fold cross-validation** con **2** particiones, o deja la opción automática que proponga Studio.
5. Pulsa **Next**.

> Si Studio muestra un apartado de límites (número máximo de intentos o tiempo máximo), puedes acotarlo para que la práctica no se alargue.

### 5. Elegir el cómputo

1. Elige **Compute cluster** como tipo de cómputo. Si tu workspace ofrece cómputo *serverless*, también sirve.
2. Crea un clúster nuevo con estos valores orientativos:

   | Campo | Valor |
   |---|---|
   | Virtual machine tier | **Dedicated** |
   | Virtual machine type | **CPU** |
   | Virtual machine size | Un tamaño pequeño de CPU con cuota disponible en tu región |
   | Compute name | `automl-compute` |
   | Minimum number of nodes | **0** (para que no consuma cuando está parado) |
   | Maximum number of nodes | **2** a **4** |
   | Idle seconds before scale down | **120** |

3. Pulsa **Create**. La creación del clúster tarda unos minutos.
4. Selecciona el clúster y pulsa **Next**.

### 6. Lanzar el trabajo

1. Revisa el resumen y pulsa **Submit training job**.
2. Se abre la página del trabajo. Espera:
   - La preparación inicial puede tardar entre 10 y 15 minutos.
   - Después, cada intento tarda unos minutos.

> Mientras espera, ve al paso 9 *Preguntas para la clase*.

### 7. Comparar los modelos

1. En la página del trabajo, abre la pestaña **Models + child jobs**.
2. Los modelos aparecen ordenados por la métrica principal, con el mejor arriba.
3. Fíjate en que cada fila es un **algoritmo distinto** con su puntuación. Esa comparación es lo que ha automatizado el servicio.
4. Abre uno de los modelos y revisa sus pestañas **Overview** y **Metrics**.

### 8. Entender por qué decide así (opcional)

Si activaste *Explain best model*, o si lanzas la explicación sobre el mejor modelo:

1. Abre el mejor modelo.
2. Pulsa **Explain model**, elige tu clúster y pulsa **Create**. Tarda unos minutos.
3. Abre la pestaña **Explanations (preview)** y revisa **Aggregate feature importance**.

**Qué debes observar:** las variables que más pesan en la predicción. Los datos se generaron para que influyan el tipo de contrato, la antigüedad, las llamadas a soporte y las incidencias, así que lo normal es que aparezcan entre las primeras.

### 9. Preguntas para la clase

1. ¿Qué diferencia hay entre este modelo y GPT en Foundry? (Este lo hemos entrenado con nuestros datos.)
2. ¿Por qué se comparan varios algoritmos en lugar de elegir uno?
3. ¿Qué pasaría si volviéramos a entrenar con datos de otro año?
4. ¿Qué haría falta para poner este modelo en producción?

---

## Resultado esperado

- Un trabajo de ML automatizado completado, con una lista de modelos comparados por su métrica.
- Un mejor modelo identificado, con sus métricas y, si lo pediste, sus explicaciones.

## Siguiente paso

En la práctica 2 desplegamos el mejor modelo en un endpoint online y lo llamamos desde Postman. Según el flujo del tutorial oficial, el modelo se **registra al desplegarlo**.

## Problemas frecuentes

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| No se puede crear el clúster | Falta de cuota de CPU en la región | Prueba otro tamaño o pide una ampliación de cuota |
| El trabajo se queda en *Preparing* | Es el comportamiento normal de la primera ejecución | Espera 10-15 minutos |
| Falla al leer el CSV | Delimitador incorrecto | Comprueba que el delimitador es **Comma** |
| No puedes elegir `baja` como objetivo | La columna se detectó con otro tipo | Revisa el paso *Schema* |

## Limpieza

Los nodos de cómputo se facturan mientras están encendidos. Con mínimo 0 nodos el clúster se apaga solo, pero al terminar:

- Para borrar solo lo desplegado: **Endpoints** (en *Assets*) → selecciona el endpoint → **Delete**.
- Para borrar todo: en el Portal de Azure, abre el grupo de recursos y pulsa **Delete resource group**.

## Referencias

- [Tutorial: Train a classification model with no-code AutoML in Azure Machine Learning studio](https://learn.microsoft.com/en-us/azure/machine-learning/tutorial-first-experiment-automated-ml)
