
# Proyecto de Pipeline de Datos

## Descripción de la Arquitectura

El proyecto final del pipeline de datos está diseñado utilizando una arquitectura basada en contenedores Docker, Kafka, Apache Spark y Hadoop HDFS. La siguiente descripción ilustra los componentes principales y el flujo de datos a través del sistema:

### Componentes

#### Contenedor Docker
Toda la solución está encapsulada dentro de un contenedor Docker, lo que asegura la portabilidad y facilita la gestión de dependencias. Docker permite desplegar aplicaciones de manera consistente en diferentes entornos, reduciendo problemas de compatibilidad y facilitando la escalabilidad horizontal.

#### Productor de Streaming de Kafka
**Twitter:** La fuente de datos en tiempo real es la API de Twitter, que actúa como productor de mensajes para Kafka. Los datos de Twitter (tweets) se extraen y se envían al broker de Kafka para su procesamiento posterior. Este paso implica la configuración de un productor de Kafka que pueda interactuar con la API de Twitter y transmitir los datos al broker de Kafka.

#### Broker de Streaming de Kafka
**Kafka:** Kafka se utiliza como intermediario que recibe los mensajes del productor (Twitter) y los almacena temporalmente. Kafka es conocido por su alta capacidad de manejo de datos en tiempo real, lo que lo hace ideal para aplicaciones que requieren procesamiento continuo de grandes volúmenes de datos.

#### Consumidor de Streaming de Kafka
**Apache Spark:** Apache Spark actúa como consumidor de los mensajes almacenados en Kafka. Spark es una plataforma de procesamiento de datos en tiempo real que permite el análisis y la transformación de grandes volúmenes de datos. En esta arquitectura, Spark está configurado para consumir los mensajes de Kafka y aplicar modelos de aprendizaje automático.

#### Modelo de Clasificación
Usamos un clasificador para categorizar los tweets. Este modelo se aplica a los datos en streaming consumidos de Kafka para realizar predicciones o clasificaciones en tiempo real.

#### Visualización
Una vez que hemos preprocesado los tweets con Apache Spark y Kafka, queremos visualizarlos a través de una interfaz y no por consola. Para esto, usamos Flask con templates para poder ver en tiempo real el análisis de los tweets que se van generando.

#### Almacenamiento de Datos en Hadoop HDFS
**Hadoop HDFS:** Los resultados del procesamiento en Spark se escriben en sistemas de almacenamiento para su posterior análisis y consulta. HDFS proporciona un almacenamiento distribuido y fiable, adecuado para manejar grandes volúmenes de datos. Permite realizar consultas y análisis posteriores de manera eficiente.

## Pipeline de Datos

1. **Extracción de Datos:** Los datos se extraen de Twitter en tiempo real utilizando su API, y se producen mensajes que se envían al broker de Kafka.
2. **Streaming a Kafka:** Los tweets se transmiten al broker de Kafka, que actúa como un buffer intermedio para transmitir los datos en streaming.
3. **Consumo de Datos:** Apache Spark, configurado como consumidor de Kafka, recibe los mensajes y los procesa utilizando modelos de aprendizaje automático.
4. **Procesamiento de Datos:** Spark MLlib aplica un modelo de regresión logística a los datos en streaming para realizar análisis y predicciones en tiempo real.
5. **Almacenamiento de Resultados:** Los resultados del procesamiento se almacenan en Hadoop HDFS, permitiendo consultas y análisis posteriores.

## Configuración para la Instalación

Primero, crea un entorno virtual e instala las dependencias necesarias dentro de ese entorno virtual utilizando los siguientes comandos:

```bash
python -m venv virtualenv
pip install -r requirements.txt
```

## Ejecución del Proyecto

Luego, realiza la ejecución de Docker utilizando el comando:

```bash
docker-compose -f zk-single-kafka-single.yml up -d
```

Este comando levanta todos los contenedores interconectados, creando un HDFS distribuido con 3 nodos y 3 brokers.

Después, crea los tópicos. En este ejemplo, solo crearemos un tópico utilizando el comando:

```bash
docker exec -it kafka1 /bin/bash
kafka-topics --create --topic twitter --bootstrap-server localhost:9092
kafka-topics --describe --topic twitter --bootstrap-server localhost:9092
```

Ahora, ejecuta el productor y el consumidor de Kafka en terminales separadas:

```bash
python producer.py
python consumer.py
```

Puedes ver que los servicios se están ejecutando correctamente en el puerto de Apache Flink.

## Resultados

El formato de los datos de un tweet es el siguiente:

```json
"data": {
    "id": str(uuid.uuid4()),
    "text": fake.text(max_nb_chars=280),
    "created_at": datetime.utcnow().isoformat() + "Z"
},
"includes": {
    "users": [
        {
            "id": str(uuid.uuid4()),
            "name": fake.name(),
            "username": fake.user_name()
        }
    ]
}
```

Cada 2 segundos se genera un tweet, que está conectado a Apache Kafka. El consumidor, ejecutado en otra ventana, clasifica cada tweet en una categoría basada en el texto.

### Categorías

```python
CATEGORIES = {
    'religion': ['god', 'church', 'faith', 'bible', 'pray'],
    'food': ['pizza', 'burger', 'sushi', 'coffee', 'restaurant'],
    'drinks': ['beer', 'wine', 'cocktail', 'juice', 'water'],
    'travel': ['vacation', 'trip', 'flight', 'hotel', 'destination'],
    'sports': ['football', 'basketball', 'soccer', 'tennis', 'exercise'],
    'technology': ['computer', 'software', 'hardware', 'internet', 'tech'],
    'movies': ['film', 'cinema', 'actor', 'director', 'trailer'],
    'music': ['song', 'album', 'concert', 'band', 'artist'],
    'fashion': ['clothing', 'shoes', 'accessories', 'style', 'design'],
    'health': ['medicine', 'doctor', 'fitness', 'diet', 'wellness']
}
```

El resultado en la consola indica la categoría a la que pertenece cada tweet, basándose en el texto. Para el procesamiento en batch, los tweets se almacenan en archivos en HDFS:

```python
tweet_text = tweet.get('data', {}).get('text', '')
category = categorize_tweet(tweet_text)

logging.info(f"Tweet: {tweet_text}")
logging.info(f"Categoría: {category}")

tweet_buffer.append(f"Tweet: {tweet_text}\nCategoría: {category}\n\n")

if len(tweet_buffer) == 2:
    file_name = f'user/data/file_{file_index}.txt'
    try:
        with hdfs_client.write(file_name, encoding='utf-8') as writer:
            writer.write(''.join(tweet_buffer))
        logging.info(f"Guardado en HDFS: {file_name}")
    except Exception as e:
        logging.error(f"Error al escribir en HDFS: {e}")

    tweet_buffer.clear()
    file_index += 1
```

## Visualización con Flask

Para proporcionar una interfaz adecuada en lugar de la salida en consola, se utiliza Flask. La visualización muestra el usuario, la categoría y el texto del tweet para su análisis. También muestra la cantidad de categorías generadas a lo largo del tiempo.

```python
# Código de Flask
```
```
