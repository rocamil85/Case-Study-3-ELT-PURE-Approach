## Documentación del Proceso de Transformación de Datos en GCP utilizando un enfoque ELT Puro.
Este documento detalla la implementación de un sistema de procesamiento de órdenes en tiempo real utilizando un enfoque ELT sobre la plataforma Google Cloud Platform (GCP) con la particularidad de que la información en streaming va a pasar directamente a un datawarehouse sin una mínima transformación intermedia. El sistema mejora la eficiencia y reduce los costos al minimizar los servicios de GCP a Pub/Sub, BigQuery, y Dataform. Esta documentación se enfoca en la extracción de datos, el proceso de carga y transformación, y la automatización de pipelines de datos para análisis de negocio.

### 1. Introducción
La necesidad de procesar grandes volúmenes de datos de órdenes en tiempo real nos llevó a evaluar y migrar de un enfoque tradicional ETL a un enfoque más flexible y escalable ELT. Este cambio permite la manipulación de datos directamente en nuestro almacén de datos, BigQuery, optimizando costos y tiempo de procesamiento.

### 2. Arquitectura del Sistema
La arquitectura diseñada se centra en la ingestión de eventos de órdenes en tiempo real, pero sin procesamiento inicial de limpieza y transformación intermedia, derivando directamente los datos tal cual  van llegando en BigQuery para transformaciones más complejas y análisis futuros.

### 2.1 Flujo de Datos
Extracción: Las órdenes son capturadas y enviadas en formato JSON cada vez que cambian de estado.
Pub/Sub: Una función Cloud recibe las órdenes y las publica en un tema de Pub/Sub diseñado para la ingestión de eventos en tiempo real.
BigQuery y Dataform: Dentro de BigQuery, se utilizan scripts de Dataform para transformaciones adicionales, estructuración de datos y preparación para análisis de negocio.
### 2.2 Diagrama de Arquitectura
![Arquitectura ELT](Img/ELT_Pure_architect.png) 


### 3. Detalle de Componentes
### 3.1 Google Cloud Pub/Sub
Google Cloud Pub/Sub se utiliza para decoupling de servicios productores de datos de los consumidores. Este enfoque asegura un procesamiento de mensajes escalable y confiable. Se configuró un tema específico para las órdenes, donde cada cambio de estado es publicado en tiempo real por una Cloud Functions.
El detalle en este caso es que Pub/Sub tiene una suscripción de tipo **Write to bigquery** que permite precisamente transmitir el mensaje que llega directamente a una tabla existente en Bigquery. De esta forma la información llega en tiempo real al Datawarehouse aunque de forma cruda.  Los datos se van ingestando un campo llamado “data” de la tabla existente.

![PubSub](Img/wirte_to_bigquery.png) 

![Bigquery](Img/data_Bigquery.png) 
 
Posteriormente a través de Dataform se despliega un pipeline en SQL que permite hacer transformaciones que anteriormente se hacían con Dataflow, quedando lista para su análisis posterior y ahorrando costos significativos.

### 3.2 BigQuery y Dataform
Una vez en BigQuery, utilizamos Dataform para gestionar transformaciones complejas y la preparación de datos para análisis. Dataform permite definir y automatizar workflows de transformación de datos SQL, facilitando la orquestación de tareas y la dependencia entre ellas. Además, su integración con sistemas de control de versiones como Git permite una colaboración eficaz y el seguimiento de cambios en los scripts de transformación.

### Programación de Dataform
Los scripts de Dataform se programan para ejecutarse cada 10 minutos, utilizando triggers basados en tiempo. Esta frecuencia garantiza que los datos estén constantemente actualizados y listos para análisis. La configuración se realiza a través de la interfaz de Dataform, especificando la rama del repositorio Git a utilizar, lo que permite una integración continua y entrega continua (CI/CD) de cambios en los scripts de transformación.

### 4. Conclusiones
La implementación de este sistema ELT Puro en GCP ha permitido un procesamiento de datos más eficiente y costo-efectivo. La arquitectura diseñada asegura que los datos estén disponibles y actualizados para su análisis, apoyando la toma de decisiones de negocio en tiempo real. Este proyecto destaca la versatilidad y potencia de las herramientas de GCP para enfrentar desafíos complejos de ingeniería de datos.
