# Stream Connect
### Install Kafka on a Linux-based VM
* Follow https://www.linuxtechi.com/how-to-install-apache-kafka-on-ubuntu/
* My YouTube Video [01 ServiceNow Kafka Integration: Installing Kafka on a VM](https://youtu.be/1Zp1GigqH9w?list=PL5DgOfLBA3Rbllyw8mfqba52m3k_U4EnX)
### Setup for direct instance connection through Hermes
* Please follow along my YouTube Video [02 ServiceNow Kafka Integration: StreamConnect Produce and Consume](https://youtu.be/TDtJB00XwW8?list=PL5DgOfLBA3Rbllyw8mfqba52m3k_U4EnX)
* Pre-requisites
    - Make sure that your instance can be accessed via the adcv2 load balancer. If it is not, you will need to create a case to migrate your instance from F5 to ADCV2. Check using the following URL: **https://\<InstanceName>.service-now.com/adcv2/server**
    - **com.glide.kmf.global** plugin must be installed
    - Check **https://\<InstanceName>.service-now.com/now/key-management-framework/health**
    ![KMF Health](images/kmfHealth.png)
    - Check Hermes status **\<InstanceName>.service-now.com/xmlstats.do?include=services_status**
    - Verify that the user is assigned **sn_kmf.cryptographic_manager** role. If the user cannot see the role or if the user cannot access the **sys_kmf_certificate.list** then it is likely that you will need to create a case to enable Hermes service on the instance ![Role](images/role.jpg)

### Create a truststore and a keystore to connect 
reference:
- https://docs.servicenow.com/bundle/washingtondc-servicenow-platform/page/administer/hermes-messaging-service/task/create-hermes-topic.html

1. Create a directory to store the files. ex: `mkdir hermes`
2.  sys_kmf_certificate.list open the root certificate copy the PEM certificate and paste it into **ipki-ca.pem** file
3. Create <InstanceName>.cnf file with the following content:
    ```
    prompt=no
    encrypt_key=no
    distinguished_name=req_dn
    default_md=sha256
    [req_dn]
    CN=<Root Cert's Subject Common Name>
    O=<Root Cert's Subject organization>
    GN=Customer-Kafka
    SN=<Instance ID from the stats page>
    ```
4. Generate a PEM-encoded Certificate Request (CSR) by running the following command: 
    ``` 
    openssl req -new -newkey rsa:2048 -nodes -config <instance_name>.cnf -keyout <instance_name>.key -out <instance_name>.pem 
    ```
5. Back at the instance certificate table, click request certificate
    ![Request](images/request.jpg)
6. After uploading the certifiate request, open the new signed certificate record, copy the PEM Certificate text and paste it into the **customer-cert.pem** file. The directory should contain the following files:
    ![Files](images/files.jpg)
7. Create a keystore using the following command:
    ```
    openssl pkcs12 -export -out keystore.p12 -inkey <instance_name>.key -in customer-cert.pem -certfile ipki-ca.pem 
    ```
8. To generate a truststore we need to download an Entrust Certificate authority. In a Web browser, visit the https://www.entrust.com/resources/tools/root-certificate-downloads and download the chain certificate, selecting (Non EV SSL) CA - L1K.
    ![Entrust CA](images/entrust.jpg)
    ```
    wget https://web.entrust.com/root-certificates/entrust_l1k.cer
    ```

9. Create the truststore with the following command:
    ```
    keytool -keystore truststore.p12 -alias entrust -import -file entrust_l1k.cer
    ```
10. With the trustsore and the keystore files we can now exchange messages with the instance.
### Produce and consume messages
References: 
- https://docs.servicenow.com/bundle/washingtondc-servicenow-platform/page/administer/hermes-messaging-service/task/produce-messages-hermes.html
- https://docs.servicenow.com/bundle/washingtondc-servicenow-platform/page/administer/hermes-messaging-service/task/consume-messages-hermes.html
1. Create a new directory and copy the keystore and the truststore files to it
    ```
    mkdir demo
    cp *.p12 /demo
    ```
2. Create a configuration file in the new demo directory. In the docs they create separate file for producer.properies and for consumer.properies, bot the two files are identical.
    * properties.cfg
        ```
        security.protocol=SSL
        ssl.truststore.password=<truststore password>
        ssl.truststore.location=<full path>/demo/truststore.p12
        ssl.truststore.type=PKCS12
        ssl.keystore.password=<keystore password>
        ssl.keystore.location=<full path>/demo/keystore.p12
        ssl.keystore.type=PKCS12
        ssl.key.password=<keystore password>
        ```
3. Navigate to the Hermes Diagnostic page and copy the producer and consumer bootstrap
    ![Bootsrap](images/bootstrap.png)
4. In the instance, **sys_kafka_topic.list** add a topic. For example: demo-topic. 
    - Note: We could create the topic from the command line, but it is easier to create it from the UI, and get the topic prefix correctly when listing it in the next step.
5. <u>**List topics**</u>: In Kafka Linux issue the following command from the same location as the producer.properties (demo directory):
    ```
    /usr/local/kafka/bin/kafka-topics.sh --command-config properties.cfg --bootstrap-server <InstanceName>.service-now.com:4000,<InstanceName>.service-now.com:4001,<InstanceName>.service-now.com:4002,<InstanceName>.service-now.com:4003 --list
    ```
    - Note the full path to kafka-topics.sh script and the producer bootstrap from step 3
    - Note that the demo-topic that was created in step 1 has a prefix on snc.<InstanceName>, which is the default namespace
6. <u>**Produce Messages**</u>: In Kafka Linux issue the following command from the same location as the producer.properties (demo directory):
    ```
    /usr/local/kafka/bin/kafka-console-producer.sh --producer.config properties.cfg --topic snc.<InstanceName>.demo-topic  --bootstrap-server <InstanceName>.service-now.com:4000,<InstanceName>.service-now.com:4001,<InstanceName>.service-now.com:4002,<InstanceName>.service-now.com:4003
    ```
    - Note the full path to kafka-console-producer.sh script and the producer bootstrap from step 3
    - Note the topic name include the prefix as listed in step 5
    - Note that the demo-topic that was created in step 1 has a prefix on snc.\<InstanceName>, which is the default namespace
    - On the instance, inspect the messages using the Topic Inspector
    ![Topic Inspector](images/topicInspector.png)
7. <u>**Consume Messages**</u>: In Kafka Linux issue the following commands from different terminal windows (demo directory):
    ```
    /usr/local/kafka/bin/kafka-console-consumer.sh --consumer.config properties.cfg --topic snc.<InstanceName>.demo-topic  --bootstrap-server <InstanceName>.service-now.com:4100,<InstanceName>.service-now.com:4101,<InstanceName>.service-now.com:4102,<InstanceName>.service-now.com:4103 --group snc.<InstanceName>.g1 --from-beginning 
    ```
    
    ```
    /usr/local/kafka/bin/kafka-console-consumer.sh --consumer.config properties.cfg --topic snc.<InstanceName>.demo-topic  --bootstrap-server <InstanceName>.service-now.com:4200,<InstanceName>.service-now.com:4201,<InstanceName>.service-now.com:4202,<InstanceName>.service-now.com:4203 --group snc.<InstanceName>.g1 --from-beginning
    ```
    - Note the full path to kafka-console-consumer.sh script and the consumer bootstrap from step 3
    - Note the topic name include the prefix as listed in step 5
    - Note that the demo-topic that was created in step 1 has a prefix on snc.\<InstanceName>, which is the default namespace
    - The consumer group snc.<InstanceName>.g1 will consume the massge only once
    - Both terminal windows are required because we do not know which consumer bootsrap is active








