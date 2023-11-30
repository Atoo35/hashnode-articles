---
title: "Kafka on Kubernetes"
seoTitle: "Deploy Kafka on Kubernetes ultimate guide"
datePublished: Thu Nov 30 2023 19:39:50 GMT+0000 (Coordinated Universal Time)
cuid: clpllo3bz00050albh8a98s4v
slug: kafka-on-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701198754605/cab34c0b-bcde-4dd9-b905-9c1ca7304285.png
tags: docker, kubernetes, kafka, minikube

---

Hey everyone!! It's been very long since I wrote any blogs on here. So to make up for it I will be writing a few on deploying stuff on Kubernetes using [minikube](https://minikube.sigs.k8s.io/docs/) to keep it local. Today we will learn about how to deploy a Kafka cluster and create a topic to enable messaging queues for other services

Here is the link to [**GitHub Repository**](https://github.com/Atoo35/kafka). You can go ahead and get the code directly but I would recommend reading through the article once.

# Prerequisites

* Basic knowledge of [Kubernetes](https://kubernetes.io/) and [Kafka](https://kafka.apache.org/)
    
* Minikube installed on the system
    
* [Kubectl](https://kubernetes.io/docs/reference/kubectl/) and [Docker](https://www.docker.com/)
    
* NodeJS (since I would be writing small script to test the kafka queue)
    

# Let's Build it!!

### Part 1: Deploying Kafka and creating a topic.

1. Of course start your minikube environment ( lol ) using the below command:
    
    ```bash
    minikube start
    ```
    
2. Create a file name it anything you want and paste the below yml configuration. All it does is just create a namespace for all your kafka related deployments to be bundled together:
    
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: kafka
      labels:
        name: kafka
    ```
    
3. Create another file, name it zookeeper and paste the below code:
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: zookeeper-deployment
      namespace: kafka
      labels:
        app: zookeeper
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: zookeeper
      template:
        metadata:
          labels:
            app: zookeeper
        spec:
          containers:
            - name: zookeeper
              image: confluentinc/cp-zookeeper:7.0.1
              ports:
                - containerPort: 2181
              env:
                - name: ZOOKEEPER_CLIENT_PORT
                  value: "2181"
                - name: ZOOKEEPER_TICK_TIME
                  value: "2000"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: zookeeper-service
      namespace: kafka
    spec:
      selector:
        app: zookeeper
      ports:
        - protocol: TCP
          port: 2181
          targetPort: 2181
    ```
    
    In a very basic explanation, the zookeeper keeps track of all the Kafka brokers in a cluster. It helps to keep track of who the current leader is in the partition and topic and if need be perform elections in the event of a broker crash
    
4. The last file you need to create is the actual Kafka broker file and paste the below config:
    
    ```yaml
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: kafka-deployment
      namespace: kafka
      labels:
        app: kafka
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: kafka
      template:
        metadata:
          labels:
            app: kafka
        spec:
          containers:
            - name: broker
              image: confluentinc/cp-kafka:7.0.1
              ports:
                - containerPort: 9092
              env:
                - name: KAFKA_BROKER_ID
                  value: "1"
                - name: KAFKA_ZOOKEEPER_CONNECT
                  value: zookeeper-service:2181
                - name: KAFKA_LISTENER_SECURITY_PROTOCOL_MAP
                  value: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
                - name: KAFKA_ADVERTISED_LISTENERS
                  value: PLAINTEXT://:29092,PLAINTEXT_INTERNAL://kafka-service:9092
                - name: KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR
                  value: "1"
                - name: KAFKA_TRANSACTION_STATE_LOG_MIN_ISR
                  value: "1"
                - name: KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR
                  value: "1"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: kafka-service
      namespace: kafka
    spec:
      selector:
        app: kafka
      ports:
        - protocol: TCP
          port: 9092
          targetPort: 9092
    ```
    
5. Now we need to actually deploy the above configurations on our local minikube kubernetes cluster. The command is simple, just use the below command for each of the files in the order we wrote them:
    
    ```yaml
    kubectl apply -f <filename>
    ```
    
    This command should create the namespace and all the deployments and services needed for Kafka.
    
    Now if you run the command `kubectl get pods -n kafka` you should see zookeeper and kafka broker pods running.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701370119595/e4f6572e-f19e-42e3-96f0-ec859bc2ef53.png align="center")
    
6. Now that we have the instances running, we need to create a topic in the kafka broker for all the publishers to push messages and subscribers to read the messages from. In order to do that, we need to ssh into the kafka-deployment pod.  
    Run the below command replacing the pod name with your pod name to ssh into the pod:
    
    ```bash
    kubectl exec -it <pod-name> -n kafka -- /bin/bash
    ```
    
    Your terminal should look something like this:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701370293276/56185cb2-70d4-49b5-a61d-03fbbf98ddd9.png align="center")
    
    The next step is to create a topic named `test-topic`. Simply paste the below command and hit enter. All we are doing here is creating a broker with a replication factor of 1 and 3 partitions to make it fault-tolerant.
    
    ```bash
    kafka-topics --bootstrap-server localhost:9092 --create --topic test-topic --replication-factor 1 --partitions 3
    ```
    
    Wait for a few seconds to see the `Created topic test-topic` message in the terminal. Type `exit` to exit the ssh terminal and return back to your local terminal.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701370464836/fe440fae-bd6d-421c-bd1f-f61bba1eacba.png align="center")
    
    At this point, we have successfully deployed a Kafka broker on Kubernetes and have created a topic to send and receive messages to and from, so congratulate yourself!!!!
    

### Part 2: Creating Publishers and Subscribers

Now it's time to subscribe to the messages we will later push to the broker. It is pretty simple to write a NodeJS script and deploy it on Kubernetes in the same namespace.

1. Create a new folder create the file index.js inside the folder and paste the below code. Do run `npm init -y` command and `npm install kafkajs`:
    
    ```javascript
    const { Kafka } = require('kafkajs')
    
    
    // create consumer
    const kafka = new Kafka({
        clientId: 'my-app',
        brokers: ['kafka-service:9092', 'kafka-service:9092']
    })
    
    const consumer = kafka.consumer({ groupId: 'my-app' })
    
    const run = async () => {
    
        await consumer.connect()
        await consumer.subscribe({ topic: 'test-topic', fromBeginning: true })
    
        await consumer.run({
            eachMessage: async ({ message }) => {
                console.log(`received message: ${message.value}`)
            }
        })
    }
    
    run().catch(console.error)
    ```
    
    Nothing fancy, simply connecting to `kafka-service` we created on kubernetes and sending a simple hello world kinda message.
    
    > Note: We can't run this locally since we have kafka deployed on kubernetes and will have to deploy our app onto the same namespace to run it. There are ways to forward the port and configure kafka to accept external messages but it is out of scope for this guide.
    
2. Now to deploy this app onto kubernetes, create a `.dockerignore` file and paste the below code:
    
    ```plaintext
    .env
    node_modules/
    ```
    
3. Create a `Dockerfile` and paste the below code:
    
    ```dockerfile
    FROM node:18-alpine
    
    WORKDIR /app
    
    COPY package*.json ./
    
    RUN npm install
    
    COPY . ./
    
    CMD ["node","index.js"]
    ```
    
4. Finally, create a `deployment.yml` file and paste the below code:
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kafka-test-consumer-deployment
      namespace: kafka
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: kafka-test-consumer
      template:
        metadata:
          labels:
            app: kafka-test-consumer
        spec:
          containers:
            - name: kafka-test-consumer
              image: test-kafka-consumer
              imagePullPolicy: Never
              ports:
                - containerPort: 8081
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: kafka-test-consumer-service
      namespace: kafka
    spec:
      selector:
        app: kafka-test-consumer-service
      ports:
        - name: kafka-test-service-port
          protocol: TCP
          port: 8081
          targetPort: 8081
    ```
    
    We need to build the docker image first, and even before that we need to run the below command for minikube to not fetch images from some docker registry but rather fetch locally
    
    ```yaml
    eval $(minikube docker-env)
    ```
    
5. Build the docker image by running the below command
    
    ```bash
    docker build -t test-kafka-consumer .
    ```
    
6. Once the image is created, run the `kubectl apply -f deployment.yml` command to deploy the consumer in the same namespace as the broker. Run `kubectl get pods -n kafka` to make sure the consumer is working fine
    

Here are the steps to create a producer and deploy on minikube:

1. Create a new folder create the file index.js inside the folder and paste the below code. Do run `npm init -y` command and `npm install kafkajs`
    
    ```javascript
    const { Kafka } = require('kafkajs')
    
    const kafka = new Kafka({
        clientId: 'my-app',
        brokers: ['kafka-service:9092', 'kafka-service:9092']
    })
    
    const producer = kafka.producer()
    
    const run = async () => {
    
        await producer.connect()
        await producer.send({
            topic: 'test-topic',
            messages: [
                { value: 'Hello KafkaJS user!' },
            ],
        })
    
        console.log('Message sent!')
    }
    
    run().catch(console.error)
    ```
    
    Nothing fancy, simply connecting to `kafka-service` we created on kubernetes and sending a simple hello world kinda message.
    
    > Note: We can't run this locally since we have kafka deployed on kubernetes and will have to deploy our app onto the same namespace to run it. There are ways to forward the port and configure kafka to accept external messages but it is out of scope for this guide.
    
2. Now to deploy this app onto kubernetes, create a `.dockerignore` file and paste the below code:
    
    ```plaintext
    .env
    node_modules/
    ```
    
3. Create a `Dockerfile` and paste the below code:
    
    ```dockerfile
    FROM node:18-alpine
    
    WORKDIR /app
    
    COPY package*.json ./
    
    RUN npm install
    
    COPY . ./
    
    CMD ["node","index.js"]
    ```
    
4. Finally, create a `deployment.yml` file and paste the below code:
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kafka-test-service-deployment
      namespace: kafka
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: kafka-test-service
      template:
        metadata:
          labels:
            app: kafka-test-service
        spec:
          containers:
            - name: kafka-test-service
              image: test-kafka
              imagePullPolicy: Never
              ports:
                - containerPort: 8080
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: kafka-test-service
      namespace: kafka
    spec:
      selector:
        app: kafka-test-service
      ports:
        - name: kafka-test-service-port
          protocol: TCP
          port: 8080
          targetPort: 8080
    ```
    
    We need to build the docker image first, and even before that we need to run the below command for minikube to not fetch images from some docker registry but rather fetch locally
    
    ```yaml
    eval $(minikube docker-env)
    ```
    
5. Build the docker image by running the below command
    
    ```bash
    docker build -t test-kafka .
    ```
    
6. Once the image is created, run the `kubectl apply -f deployment.yml` command to deploy the consumer in the same namespace as the broker. Run `kubectl get pods -n kafka` to make sure the producer is working fine
    

If both consumer and producer pods are running we can actually go ahead and see the consumer pod logs to see that we have indeed received the message we sent. To do that use the below command to replace the pod-name for your consumer pod:

```bash
kubectl logs <pod-name> -n kafka
```

You should be able to see the message at the last line and the output should look something like this

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701372986387/a28621fb-f6df-4d3b-b121-2bdac1129890.png align="center")

Voila!! That's it, you have successfully deployed and tested Kafka on Kubernetes on your local machine!!  
Enjoy!!!

### **Support**

If you liked my article, consider supporting me with a coffee ☕️ or some crypto ( ₿, ⟠, etc)

Here is my public address `0x7935468Da117590bA75d8EfD180cC5594aeC1582`

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/default-yellow.png align="left")](https://www.buymeacoffee.com/atoo)

### Let's Connect

[**Github**](https://github.com/Atoo35)

[**LinkedIn**](https://www.linkedin.com/in/atoo35)

[**Twitter**](https://twitter.com/atharva_35)

### Feedback

Let me know if I have missed something or provided the wrong info. It helps me keep genuine content and learn.