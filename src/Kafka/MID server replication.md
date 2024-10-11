## Stream Connect With MID server Message Replication
### Securing the Kafka Server with SSL
* Please follow along my YouTube Video [03 ServiceNow Kafka Integration: MID Server Replication - Securing Kafka](https://youtu.be/Dg9sQgld1_Y?list=PL5DgOfLBA3Rbllyw8mfqba52m3k_U4EnX)

1. Make a directory to which we will create a self-signed CA, keystore, and truststore
    ```
    mkdir ~/demo
    cd ~/demo
    ```
2. Create a self-signed certificate authority
    ```
    openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650
    ```
2a. Create truststore
    ```
    keytool -keystore kafka.server-r.truststore.jks -alias ca-cert -import -file ca-cert
    ```
3. Get the private IP of the Kafka Linux server and the server name
4. Create the keystore (You will need to modify this command with your server name and private IP Address)
    ```
    keytool -keystore kafka.server-r.keystore.jks -alias server-r -validity 3650 -genkey -keyalg RSA -ext SAN=dns:kafka,IP:10.0.0.7
    ```
5. Create certificate signing request (CSR)
    ```
    keytool -keystore kafka.server-r.keystore.jks -alias server-r -certreq -file ca-request-server-r.csr
    ```
6. Sign the request using CA keys
    ```
    openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-server-r.csr -out ca-signed-server-r.crt -days 3650 -CAcreateserial
    ```
7. Import the root certificate to the new keystore
    ```
    keytool -keystore kafka.server-r.keystore.jks  -alias ca-cert -import -file ca-cert
    ```
8. Import the signed cert to the new keystore
    ```
    keytool -keystore kafka.server-r.keystore.jks -alias server-r -import -file ca-signed-server-r.crt
    ```
9. Make the following directory
    ```
    mkdir /usr/local/kafka/config/ssl
    ```
10. Copy the keystore and the truststore to the new ssl directory
    ```
    cp *.jks /usr/local/kafka/config/ssl
    ```
11. Modify the server.properties file to point to the keystore and the trust store and add the relevant SSL listeners. A **sample** file is available [here](server.properties) (You will need to modify it)
12. Make a new directory to test the SSL connection
    ```
    mkdir ~/demo/testssl
    cd ~/demo/testssl
    ``` 
13. Create the client-ssl.properties file to point to the keystore and the trust store and add the relevant SSL listeners. A **sample** file is available [here](client-ssl.properties) (You will need to modify it)
14. Start Zookeeper and Kafka. verify the status.
    ```
    sudo systemctl start zookeeper
    sudo systemctl start kafka
    sudo systemctl status zookeeper
    sudo systemctl status kafka
    ```
15. Use SSL to produce massages:
    ```
    /usr/local/kafka/bin/kafka-console-producer.sh --broker-list kafka:9093 --topic test --producer.config client-ssl.properties
    ```
16. Use SSL to consume the produced massages:
    ```
    /usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka:9093 --topic test --consumer.config client-ssl.properties --from-beginning
    ```
### Using the MID server to replicate messages
* Please follow along my YouTube Video [04 ServiceNow Kafka Integration: MID Server Replication - Setup](https://youtu.be/N87f2OrY1Ho?list=PL5DgOfLBA3Rbllyw8mfqba52m3k_U4EnX)

* Reference: [Stream Connect Message Replication](https://docs.servicenow.com/bundle/washingtondc-integrate-applications/page/administer/integrationhub/concept/stream-connect-message-replication.html)

**Pre-requisites:**
- A MID server that has access to the Kafka Server (In my demo video both the MID and the Kafka server are on the same Azure Virtual Network)
- The same pre-requisites as described in the following YouTube Video [02 ServiceNow Kafka Integration: StreamConnect Produce and Consume](https://youtu.be/TDtJB00XwW8?list=PL5DgOfLBA3Rbllyw8mfqba52m3k_U4EnX)
- Nice to have:
    - INSTALL plugin ServiceNow IntegrationHub Flow Trigger - Kafka (com.glide.hub.flow_trigger.kafka) 
    - Update workflow studio plugin sn_workflow_studio

1. Verify that the MID host can connect to the Kafka server
    ```
    ping kafka
    ```
2. On the instance: Navigate to Connection & Credential Alias and create a new Kafka Server Alias. Select the Kafka connection type and save the record
3. Create a new connection in the Alias record "Kafka Server Connection". 
4. Create a new Kafka SSL credential record for the connection
5. On the Kafka server navigation to the location of the keystore and the truststore, use base64 to encode the files.
    ```
    cat kafka.server-r.keystore.jks |base64 -w 0
    cat kafka.server-r.truststore.jks |base64 -w 0
    ```
6. Copy the encoded text into the corresponding credential record fields, with the correct passwords.
7. Once the credential was saved on the connection record, enter the Kafka server's SSL bootsrap connection ```kafka:9093```
8. On the Kafka server, create a source topic (make sure that the server is running):
    ```
    /usr/local/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic KafkaServerSource
    ```
9. On the instance topics, create the destination Hermes topic ```Hermes Destination```
10. Create a new message replication record with the connection alias created in step 3.
11. Create a new Kafka Topic Replication record with the source topic (Step 8) and the destination topic (Step 9) and the direction ``To ServiceNow"
12. Observe that the replication is initiated and running without errors
13. Produce messages on the SSL port targeting the KafkaServerSource topic
    ```
    cd ~/ssl/demo
    /usr/local/kafka/bin/kafka-console-producer.sh --broker-list kafka:9093 --topic KafkaServerSource --producer.config client-ssl.properties
    ```
14. Observe that the messages were replicated by the MID server by inspecting the ```Hermes Destination``` topic
15. Open the Kafka server connection (Kafka connection record that was created in Step 3), Observe the Consumer group id field
16. On the Kafka server, the following command can be used to list the consumer group IDs
    ```
    /usr/local/kafka/bin/kafka-consumer-groups.sh --command-config client-ssl.properties --bootstrap-server kafka:9093 --list
    ```



