## Stream Connect With MID server Message Replication
### Securing the Kafka Server with SSL
1. Make a diretory to which we will create a self-signed CA, keystore, and truststore
    ```
    mkdir ~/demo
    cd ~/demo
    ```
2. Create a self-signed certificate authority
    ```
    openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650
    ```
3. Get the private IP of the Kafka Linux server and the server name
4. Create the keystore
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


