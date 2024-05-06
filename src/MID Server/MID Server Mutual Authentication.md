## MID Server Mutual Authentication
### Installing a MID Server
* Please follow along my YouTube Video [01 MID Server Mutual Authentication - Standard MID Server Installation](https://www.youtube.com/watch?v=XKi2RhYa4JU&list=PL5DgOfLBA3RYE5p9BAnEyzpfZZx3R_O_g)

### MID Server Mutual Authentication
* Please follow along my YouTube Video [04 ServiceNow Kafka Integration: MID Server Replication - Setup](https://youtu.be/N87f2OrY1Ho?list=PL5DgOfLBA3Rbllyw8mfqba52m3k_U4EnX)

* Reference: [MID Server Mutual Authentication KB1116112](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1116112)
    - KB1116112 **Troubleshooting** section lists the pre-requisite requiremants for the:
        - Load Balancer: To validate, visit the URL: <instance_url>/adcv2/server
        - Load balancer Mutual Authentication setup: <instance_url>/adcv2/supports_tls
        - Mutual authentication plugin: <instance_url>/$allappsmgmt.do?sysparm_redirect=true&sysparm_search=com.glide.auth.mutual
* Reference: [Create the root pair](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html)
    - to create your own certificate chain, please follow
        - Create the root pair
        - Create the intermediate pair
        - The configuration files are available [HERE](https://jamielinux.com/docs/openssl-certificate-authority/appendix/index.html)
* Reference: [MID Server Mutual Authentication Lab setup guide](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1121648)
    - Use the **Create Cert and Key** section to create the MID server client Certificate, Key, and bundle to be installed on the MID server itself

