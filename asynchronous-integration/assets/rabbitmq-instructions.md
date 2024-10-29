# RabbitMQ Invoice API integration guide

## Create a RabbitMQ instance on CloudAMQ

1. Start from <https://www.cloudamqp.com/> by clicking on **Get Started**\
   ![get started](rabbitmq-instructions/cloudamq01.jpg)

2. Create an account with your email or choose Github/Google sign-up
   ![signup](rabbitmq-instructions/cloudamq02.jpg)

3. Once your account is created and you first login, you have to create a team :
   - Put a team name, 
   - Accept tems of services, 
   - Decline GDPR
   - Click on **Create team**
   ![create a team](rabbitmq-instructions/cloudamq04.jpg)

4. Then, create a new instance by clicking on **+ Create New Instance**
   ![create a new instance](rabbitmq-instructions/cloudamq05.jpg)

5. Put an instance name, choose `Little Lemur (Free)` plan and click **Select Region** button
    ![instance setup](rabbitmq-instructions/cloudamq06.jpg)

6. Select a **region** (*example AWS*) and a **data center** where your RabbitMQ instance will be deployed and click on **Review** button
   ![region and datacenter](rabbitmq-instructions/cloudamq07.jpg)

7. Review your instance information and click on **Create Instance** button
   ![instance creation](rabbitmq-instructions/cloudamq08.jpg)

8. Once your instance is created, you should be able to see it and access it
   ![instance access](rabbitmq-instructions/cloudamq09.jpg)

9. Click on your instance name to display details like URL and credentials
    ![instance details](rabbitmq-instructions/cloudamq10.jpg)

10. In order to configure an Amplify Fusion Connection, copy the following information: 
- From **General**
  - Cluster
- From **AMQP details**
  - User & Vhost
  - Password
  - Port (non TLS)

  ## Create a RabbitMQ queue and a routing rule
    To publish a message in RabbitMQ, you need to create a queue and a routing rule. You can read about CloudAMQP RabbitMQ Exchanges and a Routing Keys [here](https://www.cloudamqp.com/blog/part4-rabbitmq-for-beginners-exchanges-routing-keys-bindings.html).

  1. Click the RabbitMQ Manager button in your CloudAMQP cluster Overview page
   ![rabbitmq manager ](rabbitmq-instructions/cloudamq12.jpg)

   2. Click on the `Queues and Streams` tab and add a new queue.
   Enter a **Name** and click on **Add queue** button
    ![rabbitmq queue1 ](rabbitmq-instructions/cloudamq13.jpg)
    ![rabbitmq queue2 ](rabbitmq-instructions/cloudamq14.jpg)

   3. One your queue is created, click on your queue name, then go to `Bindings` section.
      - Set **amq.topic** in `From exchange`
      - Set **invoiceKey** in `Routing key`
      - Click on `Bind` button to save.
![rabbitmq binding ](rabbitmq-instructions/cloudamq15.jpg)