
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




