# Gide d'intégration du broker RabbitMQ

## Création d'une instance RabbitMQ sur CloudAMQ

1. Naviguer sur <https://www.cloudamqp.com/> et cliquer sur **Get Started**\
   ![get started](../assets/rabbitmq-instructions/cloudamq01.jpg)

2. Créer un compte avec votre email ou bien en utilisant votre compte Github/Google
   ![signup](../assets/rabbitmq-instructions/cloudamq02.jpg)

3. Une fois votre compte créé, lors de votre première connexion vous devez créer une Team:
   - Renseigner un nom, 
   - Accepter les **tems of services**, 
   - Décliner la mention GDPR,
   - Cliquer sur **Create team**
   ![create a team](../assets/rabbitmq-instructions/cloudamq04.jpg)

4. Créer une nouvelle instance en cliquant sur **+ Create New Instance**
   ![create a new instance](../assets/rabbitmq-instructions/cloudamq05.jpg)

5. Renseigner un nom d'instance, choisissez le plan `Little Lemur (Free)` puis cliquer sur le bouton **Select Region**
    ![instance setup](../assets/rabbitmq-instructions/cloudamq06.jpg)

6. Sélectionner une **region** (*par exemple AWS*) ainsi qu'un **data center** où votre instance RabbitMQ sera déployée puis cliquer sur le bouton **Review**
   ![region and datacenter](../assets/rabbitmq-instructions/cloudamq07.jpg)

7. Vérifier l'exactitude des informations de votre instance puis cliquer sur le bouton **Create Instance**
   ![instance creation](../assets/rabbitmq-instructions/cloudamq08.jpg)

8. Une fois votre instance créée, vous devriez être en capacité de la voir et d'y accéder
   ![instance access](../assets/rabbitmq-instructions/cloudamq09.jpg)

9. Cliquer sur le nom de votre instance afin d'afficher aux détails comme son URL ou bien ses identifiants
    ![instance details](../assets/rabbitmq-instructions/cloudamq10.jpg)

10. Les informations suivantes seront nécessaire pour la configuration d'un composant RabbitMQ dans Amplify Integration: 
- Dans la section **General**
  - Cluster
- Dans la section **AMQP details**
  - User & Vhost
  - Password
  - Port (non TLS)

  ## Création d'une queue et d'une règle de routage
  Afin de publier un message dans RabbitMQ, il est nécessaire de créer une queue ainsi qu'une règle de routage. Plus d'information [ici](https://www.cloudamqp.com/blog/part4-rabbitmq-for-beginners-exchanges-routing-keys-bindings.html).

  1. Cliquer sur le bouton RabbitMQ Manager à partir de la page principale de votre instance
   ![rabbitmq manager ](../assets/rabbitmq-instructions/cloudamq12.jpg)

   2. Cliquer sur l'onglet `Queues and Streams` et ajoute rune nouvelle queue
   renseigner un **Name** et cliquer sur le bouton **Add queue**
    ![rabbitmq queue1 ](../assets/rabbitmq-instructions/cloudamq13.jpg)
    ![rabbitmq queue2 ](../assets/rabbitmq-instructions/cloudamq14.jpg)

   3. Une fois la queue créée, cliquer sur le nom de la queue, puis cliquer sur la section `Bindings`
      - Renseigner **amq.topic** dans `From exchange`
      - Renseigner **invoiceKey** dans `Routing key`
      - Cliquer sur le bouton `Bind` pour sauvegarder
![rabbitmq binding ](../assets/rabbitmq-instructions/cloudamq15.jpg)