# Data Pipeline: Docker, Kafka e Jupyter Notebook

![Machine Learning Banner](Images/Machine_Learning_Image.jfif)

## Table of Contents
1. [Project Overview](#project-overview)
2. [Project PDF](#project-pdf)
3. [Creating the Directory and Configuring the Environment](#creating-the-directory-and-configuring-the-environment)
4. [Starting Services with Docker Compose](#starting-services-with-docker-compose)
5. [Executing Kafka in the Docker Environment](#executing-kafka-in-the-docker-environment)
6. [Practical Case - Kafka with Jupyter Notebook](#practical-case---kafka-with-jupyter-notebook)
7. [Conclusion](#conclusion)
8. [Author](#author)

## Project Overview

In this project, we address the creation of a Big Data environment using Docker to orchestrate several essential services for processing and analyzing large volumes of data. We use Kafka as a tool for ingesting and managing data flows, Jupyter Notebook for interactivity and data analysis, and several other frameworks that are widely used in the Big Data ecosystem.
The configuration of the environment is done in an automated way using Docker and Docker Compose, which allows the creation and management of multiple containers to run the services in an efficient and isolated way. The integration between these tools provides a robust foundation for exploring data at scale, processing it in real-time, and performing detailed analysis with ease.


## Project PDF

To view the PDF file, you can access it via the following link:

- [Project PDF](Vehicle_price_prediction.ipynb)


### Tools and Skills Used:
- **Docker**: Container platform for creating and managing isolated environments, enabling efficient execution of applications and services.
- **Docker Compose**: Tool to define and run multi-container applications, simplifying the configuration and orchestration of services.
- **Apache Kafka**: Distributed messaging system used for ingesting, storing, and processing large volumes of data in real time.
- **Jupyter Notebook**: Interactive interface for data analysis, which facilitates the execution of Python code and visualization of results in a dynamic way.
- **Pandas**: Python library for data manipulation and analysis, essential for information processing and visualization.
- **Matplotlib**: Library for creating graphs and visualizations, allowing you to represent the distribution of logs and analyzed data.

![EDA Example](Images/Heatmap.png)

## Creating the Directory and Configuring the Environment

Clone the [Big Data Docker repository](https://github.com/Gustavo-Saffiotti/bigdata_docker) on GitHub. This repository contains all the necessary files, including the `docker-compose.yml`, to set up your Big Data environment. Follow the detailed instructions in the repository to complete the configuration.

## Starting Services with Docker Compose

### Initializing Kafka
Navigate to the project directory:
    ```
    cd "C:\docker\bigdata_docker"
    ```

    
Upload Kafka's service:
    ```
    docker-compose up -d kafka
    ```

    
### Initializing Jupyter Notebook
Upload the Jupyter Notebook service:
```bash
docker-compose up -d jupyter-notebook-custom
```

### Check Active Containers
Run the command to check the containers:

```bash
docker ps
```



Output Example:

```bash
CONTAINER ID   IMAGE                   COMMAND                  STATUS        PORTS
dd860f2ccd5d   fjardim/kafka           "start-kafka.sh"         Up 16 minutes  0.0.0.0:9092->9092/tcp
7c5671d41277   fjardim/jupyter-spark   "/opt/docker/bin/ent..." Up 24 seconds 0.0.0.0:8889->8889/tcp
a3ec15345d8a   fjardim/zookeeper       "/bin/sh -c '/usr/sb..." Up 16 minutes  0.0.0.0:2181->2181/tcp
```


## Kafka in Docker Environment

- **Accessing the Kafka Container**:
  - Enter the container:
    ```bash
    docker exec -it kafka bash
    ```
  - Navigate to the Kafka directory:
    ```bash
    cd /opt/kafka/bin
    ```

- **Creating Topics**:
  Create topics for testing:
  ```bash
  ./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic lesson
  ./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic test
  ./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic msg
  ./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic logs
  ```

- **List the topics Created**:
```bash
./kafka-topics.sh --list --zookeeper zookeeper:2181
```


- **Sending Messages to Kafka**:
Use producer to send messages:
```bash
./kafka-console-producer.sh --broker-list kafka:9092 --topic class
Send messages like:
test
Data ingestion
new msg from console
```

- **Consuming Messages**:
In the Kafka container, use the consumer to view the messages:
```bash
./kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic lesson
```

## Practical Case - Kafka with Jupyter Notebook

- **Sending Messages**:  
  In Jupyter Notebook, run the following code:  
  ```python
  from kafka import KafkaProducer
  import time
  import random

  # Configure the producer
  producer = KafkaProducer(bootstrap_servers=['kafka:9092'])

  # Log simulation
  logs = [
      "INFO - System Started",
      "WARN - High latency detected",
      "ERROR - Failed to connect to database",
      "INFO - Requisition processed",
      "ERROR - Service Unavailable"
  ]

  # Set the number of logs to be generated
  num_logs = 20  # For example, 20 messages

  # Send logs to Kafka
  for _ in range(num_logs):
      log = random.choice(logs)
      producer.send(topic='logs', value=log.encode('utf-8'))
      print(f"Log sent: {log}")
      time.sleep(2)  # Send a log every 2 seconds

  print("Log shipping completed.")
  ```



- **Consuming Messages**:
Use the code below to consume messages:
```python
import pandas as pd
from kafka import KafkaConsumer
from IPython.display import display, clear_output
import matplotlib.pyplot as plt

# Configure the consumer
consumer = KafkaConsumer(
    'logs',  # Topic Name
    group_id='log_group',
    bootstrap_servers=['kafka:9092'],
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    value_deserializer=lambda x: x.decode('utf-8')
)

# Initialize data list to store logs
logs_data = []

print("Consuming Kafka messages...")

try:
    for message in consumer:
        log = message.value  # Decode the log
        logs_data.append(log)  # Add log to list

        # Create DataFrame
        df = pd.DataFrame(logs_data, columns=['Log'])

        # Update real-time display
        clear_output(wait=True)  # Clear previous output
        display(df.tail(10))  # Show last 10 logs

except KeyboardInterrupt:
    print("\nConsumption stopped by user.")

# Create distribution chart after consuming logs
if logs_data:
    # Create the 'Type' column by extracting the category from the log
    df['Type'] = df['Log'].str.split(' - ').str[0]

    # Count the frequency of each log type
    type_counts = df['Type'].value_counts()

    # Plot bar chart
    type_counts.plot(kind='bar', color='skyblue', edgecolor='black')
    plt.title("Log Distribution")
    plt.xlabel("Log Type")
    plt.ylabel("Frequency")
    plt.xticks(rotation=45)  # Rotate labels for better viewing
    plt.tight_layout()  # Adjust layout to avoid cropping
    plt.show()
else:
    print("No logs were consumed.")
```


![Model Result](Images/Result.png)



## Conclusion

With the completion of this project, it was possible to configure a Big Data environment in a practical and effective way using Docker. The integration of Kafka and Jupyter Notebook made it possible to ingest and analyze logs in real time, while the graphical visualization in matplotlib allowed for easy and immediate tracking of the processed data.
The use of Docker and Docker Compose made the configuration of services simple and reusable, making it possible to replicate the Big Data environment on any system. In addition, the implementation of a producer and consumer for Kafka demonstrated a practical application of data ingestion and analysis, with logging and error detection.
This environment provides a solid foundation for future studies, where it is possible to expand and integrate other large-scale data processing tools, such as Apache Spark or HDFS, to perform more complex analyses and handle large volumes of information efficiently.
This project demonstrates the importance of tools such as Docker and Kafka in the Big Data ecosystem, offering a scalable and flexible architecture to explore, process, and visualize data in real time.


## Author

- [LinkedIn](https://www.linkedin.com/in/gustavo-maldonado-saffiotti) 
- [GitHub Profile](https://github.com/Gustavo-Saffiotti)


Feel free to open an issue or pull request for suggestions or improvements.

