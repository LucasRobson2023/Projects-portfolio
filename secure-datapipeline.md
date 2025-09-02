
<img width="920" height="673" alt="Screenshot 2025-08-27 at 15 38 16" src="https://github.com/user-attachments/assets/574042d4-96c9-423d-99b9-c448b520df2e" />


<img width="843" height="500" alt="Screenshot 2025-08-27 at 15 38 35" src="https://github.com/user-attachments/assets/bdf97a65-9a02-40b0-acfd-974e09c1cfe8" />



<img width="940" height="453" alt="Screenshot 2025-08-27 at 15 38 57" src="https://github.com/user-attachments/assets/d2bda006-9b48-4cc7-a3ac-de51f3347f77" />



<img width="755" height="396" alt="Screenshot 2025-08-27 at 15 39 30" src="https://github.com/user-attachments/assets/fdc1e9c0-83f2-4f78-a507-85156c954c83" />


<img width="846" height="391" alt="Screenshot 2025-08-27 at 15 39 45" src="https://github.com/user-attachments/assets/151ffbe9-6eb1-4055-88ea-e61e9e143d4b" />


I started off with creating a Kafka service using confluent's quickstart, which provided me with the following services:
-cp-server
-schema-registry
-connect
-prometheus
-alertmanager
-control-center
-ksqldb-server

I then wanted to add Spark for the data to be consumed and transformed. I created a Master and worker, ensuring each one was on a different port, such as the Worker which usually is on 8081, which I mapped to 8084. I then connected all of the services together to be on the same network, "kafka-spark-net".

I also setup a service of my own to initalise topics automatically, after the creation of the Kafka service.

<img width="880" height="279" alt="Screenshot 2025-08-28 at 20 41 51" src="https://github.com/user-attachments/assets/17ea5850-3f2b-401d-af08-db9dffae2350" />

For Spark, I was having some issues with permissions when trying to exectute tasks. So to fix this I created folders within the services of the Master and Worker:
  - HOME=/tmp
  - IVY_HOME=/tmp/.ivy2
  - SPARK_JARS_IVY=/tmp/.ivy2

For the Master service I also attached a volume containing the Spark application which would run the consumption of the Kafka data:
- ./spark-apps:/opt/bitnami/spark-apps

I also created a Producer script which takes network traffic on my machine and feed packet by packet to the Kafka Topic.
<img width="817" height="506" alt="Screenshot 2025-08-28 at 20 47 21" src="https://github.com/user-attachments/assets/9324c670-bf5b-44f2-a78f-283384316b25" />

This fed the data into the test-topic, as seen in the control center:

<img width="1222" height="700" alt="Screenshot 2025-08-28 at 20 49 33" src="https://github.com/user-attachments/assets/aa87adc6-bac5-453e-8c49-0ff701231601" />

And as seen in the screenshot below, the Spark service consumes the data and currently just outputs the data.

<img width="1049" height="453" alt="Screenshot 2025-08-28 at 20 50 32" src="https://github.com/user-attachments/assets/efb17bf8-172a-4b2f-a5f7-f7a91ba0f6aa" />


One thing I wanted to fix before continuing is how the Spark app is run. Currently, I manually force the service to run the application, ideally this is done automatically. Along with this, I have to provide the packages in the command, which specifies the packages needed to be used and downloaded at runtime - which is inefficent.

Now when I run the services kakfa topics are created automatically along with the spark application, meaning its always ready for data to be produced and consumed. Currently, when I run the data does come through it is just outputted out to the console within the spark job container. What I want to create is a proper streaming service with the data being processed and stored.

My idea to get a flow of data was to take in my personal network traffic using tshark. This would be stored within a kafka topic, consumed by my spark application and the stored in a different kafka topic. 

As seen here this is the Producer script which feeds the data into Kafka, and the second image shows the data within the topic.

<img width="1470" height="826" alt="Screenshot 2025-09-02 at 10 18 52" src="https://github.com/user-attachments/assets/28de9af4-e40e-449c-b216-e7862f87a291" />

Once this data is stored within the Kafka topic, it can now be processed by the spark consumer. As seen above, the packet is JSON, with nested JSON within it. The type of data varies based on the internet traffic which is captured, some of the packets include UDP, TCP, ARP, ICMP and others. To capture this I will create predefined topics within Kafka to allow storage based on the type of packet. I could create the topics dynamically, however, this comes with security risks as it could enable DoS, by requiring too much storage for each topic and partition. I could also just have one table with all of the data, but with a column defining the packet - but I prefer the strict seperation as it allows for more efficent querying. 

<img width="1470" height="831" alt="Screenshot 2025-09-02 at 10 17 15" src="https://github.com/user-attachments/assets/d34bdbbc-7499-4fcf-88bf-50f982413c84" />

Withing the spark application I created a flexible schema, with the layers column including a dictionary with the information on the packet (determining if its UDP, TCP etc.). The bellow code is responsible for routing the packet to the proper topic based on the top layer of the JSON. 

<img width="575" height="138" alt="Screenshot 2025-09-02 at 10 19 33" src="https://github.com/user-attachments/assets/86b1f08b-6195-4f21-badd-e305e6bcae29" />

This then lets us route it within the correct topic, and as seen bellow, it is successful. 
<img width="1452" height="837" alt="Screenshot 2025-09-02 at 10 16 59" src="https://github.com/user-attachments/assets/927a63fd-b7a5-48df-a16a-03e154ac59e8" />

<img width="1470" height="795" alt="Screenshot 2025-09-02 at 10 18 17" src="https://github.com/user-attachments/assets/f9759573-a390-4796-87f4-246be80f7ea7" />



While testing I found that I managed to overflow the queue which is used as tshark produces more messages than kafka can process.

<img width="699" height="296" alt="Screenshot 2025-09-02 at 10 12 44" src="https://github.com/user-attachments/assets/5dcffba9-b9f9-40ab-b5cf-2e715e6c7e80" />

To fix this I implemented 3 solutions. Firstly retry on BufferError, instead of crashing, it waits until space is available. Secondly, I set producer.poll(0.1) in the retry loop, which gives Kafka time to send queued messages by 0.1 seconds. Lastly I increased the queue.buffering.max.messages when I expect a very high throughput.

<img width="1198" height="236" alt="Screenshot 2025-09-02 at 10 35 15" src="https://github.com/user-attachments/assets/1c868108-6abc-43e4-beca-fb13810de4b8" />



