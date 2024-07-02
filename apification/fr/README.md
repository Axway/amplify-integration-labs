# APIsation de factures

## Introduction

Dans cet exercice, nous allons créer une intégration qui nous permettra d'exposer avce une API des factures présentes dans une base de données. L'intégration permettra également d'orchestrer et d'agréger les données des factures avec un service de conversion de devises afin de convertir le montant de la facture dans la devise souhaitée et de calculer le montant total des factures.

L'API que nous construirons sera comme telle:

`GET /invoices?status=Overdue&currencycode=EUR`

Avec ce type de réponse:

```json
{   
    "success": true,
    "invoices": [
        {
            "invnum": "IN4001",
            "invdate": "2022-11-05",
            "businessname": "ACME Corp",
            "billtoname": "Cesar Bowman",
            "vat": "15%",
            "status": "Overdue",
            "currency": "EUR",
            "totalamt": 462.47
        },
        {
            "invnum": "IN4003",
            "invdate": "2023-01-04",
            "businessname": "Crypto Corp",
            "billtoname": "Jane Doe",
            "vat": "15%",
            "status": "Overdue",
            "currency": "EUR",
            "totalamt": 601.21
        }
    ],
    "currency": "EUR",
    "status": "Overdue",
    "grandTotal": 1063.68
}
```
Un exemple est illustré ci-dessous:

![demo](../images/intro-demo.gif)

L'intégration est décrite ci-dessous:

* Exposer un endpoint API GET qui permet de récupérer les factures en fonction de leur statut et de la devise souhaitée
* Interroger une base de données PostgreSql
* Passer en revue les factures
  * Faire un appel API REST à un service de conversion de devises pour convertir le montant total de chaque facture dans la devise souhaitée
  * Faire le total des montants de chaque facture
* Renvoyer un tableau (array) optimisé de factures avec le montant total converti ainsi qu'un montant total général pour toutes les factures

Le flux de données (data-flow est illustré ci-dessous):

![flow](../images/intro-flow.png)

Dans cet exercice, nous apprendrons à:

* Configurer une connexion serveur HTTP/S 
* Utiliser un composant HTTP/S Server Get pour définir un endpoint API GET avec des paramètres de recherche (query parameters)
* Configurer une connexion PostgreSql Database 
* Utiliser un composant PostgreSql Database Select et le branchement (plug) associé pour interroger une table avec une clause Where
* Parcourir un array (de factures)
* Configurer une connexion HTTP/S Client (à un convertisseur de devises API REST)
* Utiliser le composant Map pour:
  * Mapper les données entre elles
  * Utiliser des fonctions de Map pour régler la précision décimale et ajouter à un Json array 
* Définir la réponse pour l'API que nous exposons

L'intégration finale est illustrée ci-dessous:

![integration](../images/lab4-integration.png)

## Pré-requis

* Accès à Amplify Integration
  > Si vous n'avez pas de compte, veuillez contacter **[amplify-integration-training@axway.com](mailto:amplify-integration-training@axway.com?subject=Amplify%20Integration%20-%20Training%20Environment%20Access%20Request&body=Hi%2C%0D%0A%0D%0ACould%20you%20provide%20me%20with%20access%20to%20an%20environment%20where%20I%20can%20practice%20the%20Amplify%20Integration%20e-Learning%20labs%20%3F%0D%0A%0D%0ABest%20Regards.%0D%0A)** par mail avec en objet `Amplify Integration Training Environment Access Request`
* Une base de données Postgres pour stocker nos enregistrements de facturation. Dans cet exercice nous avons utilisé [**Neon**](https://neon.tech)
* Un accès gratuit à [**API Layer Exchange Rates Data API**](https://apilayer.com/marketplace/exchangerates_data-api).
  > Assurez-vous d'être bien inscrit et testez l'API dans POSTMAN afin de vous familiariser avec les appels d'API et la visualisation de réponses

## Étape 1

Créons une base de données Postgres qui contiendra nos factures

* Créer un compte [**Neon**](https://neon.tech). 
* Créer un projet et noter les détails de connexion de l'URL Postgres "postgres://_`username`_:_`password`_@_`server`_/_`databaseName`_" pour plus tard
* Sélectionner le projet et accéder à l'éditeur SQL pour effectuer ces 3 requêtes de base de données:
  * Créer une table de factures:

  ```sql
  CREATE TABLE Invoice (
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    invid           SERIAL PRIMARY KEY,
    invnum          VARCHAR(100) NOT NULL,
    invdate         DATE NOT NULL DEFAULT CURRENT_DATE,
    duedate         DATE NOT NULL DEFAULT CURRENT_DATE,
    businessname    VARCHAR(100) NOT NULL,
    businessaddress VARCHAR(100) NOT NULL,
    businessphone   VARCHAR(100) NOT NULL,
    billtoname      VARCHAR(100) NOT NULL,
    billtoaddress   VARCHAR(100) NOT NULL,
    vat             VARCHAR(100) NOT NULL,
    totalamt        MONEY NOT NULL,
    currency        VARCHAR(100) NOT NULL,
    status          VARCHAR(100) NOT NULL,
    notes           VARCHAR(100)
  );
  ```

  * Insérer plusieurs exemples de factures (Vous pouvez modifier les données comme vous le souhaitez)

  ```sql
  INSERT INTO Invoice
    ( invnum,businessName,businessAddress,businessPhone,
      billToName,billToAddress,
      totalAmt,currency,vat,invdate,duedate,status
    )
  VALUES
    ( 'IN4001','ACME Corp','3734 Jacobs Street,Pittsburgh, PA, 15201 , USA','412-297-3188',
      'Cesar Bowman','3734 Jacobs Street,Pittsburgh, PA, 15201 , USA',
      500.00,'USD','15%',(select current_date - 90),(select current_date - 60),'Overdue'
    ),
    ( 'IN4002','Hillside Inc','144 Main St,NY, NY, 10021 , USA','212-444-1122',
      'John Smith','144 Main St,NY, BY, 10021 , USA',
      700.00,'USD','15%',(select current_date - 30),(select current_date - 1),'Paid'
    ),
    ( 'IN4003','Crypto Corp','19 Summer Ave,Miami, FL, 88088 , USA','212-444-1122',
      'Jane Doe','19 Summer Ave,Miami, FL, 88088 , USA',
      650.00,'USD','15%',(select current_date - 30),(select current_date -1),'Overdue'
    ),
    ( 'IN4004','Jamee Corp','111 French St,New Orelans, LA, 79890 , USA','212-444-1122',
      'John Smith','144 Main St,NY, BY, 10021 , USA',
      900.00,'USD','15%',(select current_date - 1),(select current_date + 30),'Sent'
    );
  ```

  * Vérifier que ces factures soient bien dans la base de données:

  ```sql
  Select * from invoice
  ```

  ![sql editor](../images/lab1-sql-editor.png)

La base de données est maintenant prête

## Étape 2

Dans cette étape, nous allons créer notre intégration et définir un endpoint API REST en utilisant le composant HTTP/S Server et la connexion associée. Ensuite nous interrogerons notre base de données pour récupérer les factures avec un statut particulier

* Créer une nouvelle intégration (par ex: GetInvoicesByStatus)
* Cliquer sur le bouton Event et ajouter un composant HTTP/S Server et choisir GET pour la méthode
  * Une connexion HTTP/S sera nécessaire. Pour cela, cliquer sur Add à côté de Connection et donner à la connexion un nom et une description
  * Sélectionner HTTPS comme protocole et laisser None pour l'Authentication pour le moment puis cliquer sur Update
  ![HTTPS server connection](../images/lab2-https-server-connection.png)
  * Fermer l'onglet de la connexion et retourner au composant HTTP/S Server de l'intégration, rafraîchir la liste de connexion puis sélectionner la connexion tout juste créée
  * Entrer `invoices` pour le resource path (chemin d'accès à la ressource) et entrer deux Query Parameters: `status` et  `currencycode` puis apuyer sur Save. 
    > À noter que le resource path doit être unique. Étant donné que vous travaillez probablement dans un environnement partagé, vous pouvez préfixer le ressource path avec vos initiales pour le rendre unique (par exemple, lb_invoices) \
  ![HTTPS Server component](../images/lab2-https-server-component.png) 
    > À noter que nous avons toujours besoin de connecter la réponse au composant HTTP/S Server mais nous ferons cela juste après avoir défini la variable réponse
* Cliquer sur le bouton `+` pour ajouter un composant Database Select puis agrandir le panneau inférieur
  * Nous devons créer une connexion Database pour notre Database Postgres. Pour cela cliquer sur Add à côté du sélecteur de connexion et donner à la connexion un nom et une description (par ex: Neon Postgres DB)
    * Sélectionner PostgreSQL comme Database Type et choisir la version utilisée lors de la création de votre DatabaseSelect (la version par défaut est 15.x)
    * Mettre à jour l'URL de connexion jdbc:postgresql://_`server`_/_`databaseName`_ avec `host` et `database name` que vous avez noté à l'étape précèdente après la création de la database (le port PostgreSQL par défaut 5432 n'est pas requis dans l'URL)
    * Entrer le nom d'utilisateur et le mot de passe que vous avez noté après la création de votre Database
    * Cliquer sur Update puis sur Test \
    ![database connection](../images/lab2-database-connection.png)
      > Notez que si vous obtenez des erreurs de délai de connexion, vous pouvez développer la section Advanced et régler le `Connection Wait Timeout` à 1000. N'oubliez pas de cliquer sur update.
    * Fermer l'onglet de la connexion et retourner au composant Database Select dans votre intégration
  * Cliquer sur Refresh dans le panneau de la Connexion et sélectionner la connexion de Database tout juste créée
  * Nous avons besoin d'un plug pour sélectionner les factures par statut. Pour cela cliquer sur Add à côté du sélecteur de plug et donner au plug un nom et une description (par ex: GetInvoicesByStatus) puis cliquer sur le bouton Configure
    * Sélectionner le connecteur de database tout juste créée et sélectionner `Select` pour Actions et `public` pour schemas
    * Cocher la case à côté de Invoice et sélectionner tous les champs
    * Cliquer sur Where et sélectionner le champs `invoice.status` et l'opérateur `=` puis cliquer sur Generate et ensuite sur Save
    ![database plug configuration](../images/lab2-database-plug-configuration.png)
    ![database plug](../images/lab2-database-plug_.png)
    * Fermer l'onglet du plug et retourner au composant Database Select de l'intégration. Cliquer sur Refresh dans le selecteur de plug et choisir le plug tout juste créé
  * Dérouler `HTTPSServerGetOutput` dans le panneau de gauche et afficher`queryParams->status`. Tirer une ligne de `status` à `GetInvoicesByStatusInput->where->invoice_status` dans ACTION PROPERTIES sur le panneau central
  * Cliquer sur Save \
  ![database component](../images/lab2-database-component_.png)
* Nous allons déclarer une variable de réponse que nous utiliserons dans l'intégration. Voici la réponse renvoyée au client.
  * Faire un clic droit sur n'importe quelle variable du panneau de droite, sélectionner Extract et coller le texte JSON suivant qui décrit la réponse API souhaitée et cliquer sur Copy Node
    ```json
    {
        "grandTotal": 1049.51,
        "status": "Overdue",
        "currency": "EUR",
        "success": true,
        "invoices": [
            {
                "currency": "EUR",
                "totalamt": 456.31,
                "invnum": "IN4001",
                "invdate": "2023-01-12",
                "businessname": "ACME Corp",
                "billtoname": "Cesar Bowman",
                "vat": "15%"
            },
            {
                "currency": "EUR",
                "totalamt": 593.17,
                "invnum": "IN4003",
                "invdate": "2023-03-13",
                "businessname": "Crypto Corp",
                "billtoname": "Jane Doe",
                "vat": "15%"
            }
        ]
    }
    ```
  ![extract json](../images/lab2-extract-json.png)
  * Refaire un clic droit et sélectionner Paste. Nommer la variable `response` et lui donner une descrpition
  ![database component](../images/lab2-database-component2.png)

* Maintenant que nous avons déclaré notre variable de réponse API, retournons au composant HTTP/S Server et effectuons le mapping de notre réponse
  * Cliquer sur le composant HTTP/S Server au début de l'intégration et cliquer sur Response\
  ![https server component response](../images/lab2-https-server-component-response.png)
  * Cliquer sur Map Response et agrandir le panneau inférieur
  * Tirer une ligne de la variable `response` du panneau de gauche, vers `HttpResponseInput->body` sous ACTION PROPERTIES dans le panneau central et cliquer sur Save
  ![https server component response map](../images/lab2-https-server-component-response-map_.png)

L'intégration doit ressembler à ceci: \
![integration](../images/lab2-integration.png)

* Activer l'intégration et faire un appel API depuis un navigateur, Postman ou curl comme suit:

   > Passer la souris sur l'icône du lien pour voir l'URL nécessaire à l'appel API et copier le lien.
  
  ![alt text](/apification/images/image.png)
  
  > Veiller à mettre à jour le chemin de ressource "/invoices" pour qu'il corresponde à ce qui a été défini. Par exemple, dans notre cas d'utilisation, le statut est "Overdue" et le code de la devise est "EUR". Le chemin de la ressource doit donc être "/invoices?status=Overdue&currencycode=EUR".

  > Effectuez un appel l'API depuis votre navigateur, Postman ou curl avec la commande suivante:   
  
  ```bash
  curl --location --request GET "<YOUR URL>"
  ```
  
  > Note : La réponse sera vide pour l'instant, ignorez donc le message d'erreur "empty response" de votre navigateur ou client

* Consulter la transaction dans le Monitor et cliquer sur l'étape Database Select puis dérouler `GetInvoicesByStatusOutput->resultSet`. Les factures ont bien été récupérées
![transaction monitoring](../images/lab2-transaction-monitoring_.png)

## Étape 3

Dans cette étape, nous allons parcourir les factures, parser chacune d'entre elles dans un format JSON et effectuer une conversion du montant de la facture dans la devise souhaitée, qui sera transmise à l'appel API sous la forme d'un paramètre de requête

* Désactiver l'intégration
* Cliquer sur le bouton `+` et ajouter un composant For-each, l'agrandir et cliquer sur Config
* Sélectionner `GetInvoicesByStatusOutput->response->resultSet` en déroulant GetInvoicesByStatusOutput puis response, afin de boucler sur chaque élément du resultSet. Cliquer sur Save
![foreach configuration](../images/lab3-foreach-configuration_.png)
* Convertissons le montant total des factures dans la devises désirée en utilisant l'API de conversion de devises APILayer 
  * Ajouter un composant HTTP/S Client Get dans la boucle et agrandir le panneau inférieur 
  * Cliquer sur Add à côté du sélecteur Connection et donner un nom et une description à la connexion (e.g. Exchange Rates Data API) puis suivre ces étapes:
    * Sélectionner HTTPS pour le Protocol 
    * Sélectionner HTTP/2 pour la version 
    * Entrer `api.apilayer.com/exchangerates_data` pour l'Url
    * Entrer `/symbols` pour le  Safe Resource Path
    * Ajouter `apikey` en header name et entrer la clé API APILayer 
    * Cliquer sur Update puis sur Test
      ![https client connection](../images/lab3-https-client-connection.png)
* Retourner au composant HTTP/S Client Get, cliquer sur raffraîchir dans le sélecteur de connexion et choisir la connexion tout juste créée 
* Dans le panneau central, sous ACTION PROPERTIES, dérouler `HTTPSGetInput` puis :
  * Faire un clic droit sur basePath et entrer `/convert` comme value 
  * Faire un clic droit sur `queryParams` et ajouter 3 variables de type String dans queryparams (`amount`, `from` et `to` )  
  * Tirer une ligne de `GetInvoicesByStatusOutput->response->resultSet->invoice_totalamt` sur le panneau de gauche vers `HTTPSGetInput->queryParams->amount` sur le panneau central pour définir le montant à convertir par l'API d'APILayer 
  * Tirer une ligne de `GetInvoicesByStatusOutput->response->resultSet->invoice_currency` sur le panneau de gauche vers `HTTPSGetInput->queryParams->from` sur le panneau central pour définir le code devise source à utiliser par l'API d'APILayer
  * Tirer une ligne de `/HTTPSServerGetOutput->queryParams->currencycode` sur le panneau de gauche vers `HTTPSGetInput->queryParams->to` sur le panneau central et configurer pour définir le code devise cible à utiliser par l'API APILayer
  * Faire un clic droit sur n'importe quelle variable dans le panneau de droite, sélectionner Extract et coller le texte JSON suivant qui décrit la réponse API du convertisseur de devises. Cliquer ensuite sur Copy Node

    ```json
    {
        "success": true,
        "query": {
            "from": "USD",
            "to": "EUR",
            "amount": 100
        },
        "info": {
            "timestamp": 1683656343,
            "rate": 0.91192
        },
        "date": "2023-05-09",
        "result": 91.192
    }
    ```

  ![extract json](../images/lab4-extract-json.png)
  * Refaire un clic droit sur le panneau de droite et sélectionner Paste puis nommmer la variable `currencyConvertResponse`
  * Tirer une ligne depuis ACTION PROPERTIERS `HTTPSGetOutput->response` vers `currencyConvertResponse` 
  * Cliquer sur Save
  ![https client component](../images/lab3-https-client-component_.png)

* L'intégration doit ressembler à ceci:
  ![integration](../images/lab3-integration_.png)

* Activer l'intégration en faisant un appel d'API depuis un navigateur, Postman, ou curl comme suit:
  ```bash
  curl --location --request GET 'https://<dataplane-hostname>:9443/invoices?status=Overdue&currencycode=EUR'
  ```

* Retrouver l'intégration dans le Monitor et cliquer dessus. Un numéro doit apparaître dans le For-each indiquant le nombre de factures
![transaction monitoring](../images/lab3-transaction-monitoring.png)
* Cliquer sur le signe `+` à côté du composant For-each puis cliquer sur une des itérations
* Cliquer sur HTTP/S Client Get puis dérouler la variable HTTPSGetOutput pour voir la réponse API de la conversion de devise
![transaction monitoring response details](../images/lab3-transaction-monitoring-response-details.png)

## Étape 4

Dans cette étape, nous allons mettre en correspondance (mapping) notre facture et le montant converti dans la devise souhaitée avec l'array de factures de la réponse, et calculer le motant total

* Désactiver l'intégration pour continuer le design
* Nous devons d'abord créer notre facture en utilisant le résultat de la conversion pour mettre à jour la devise et le montant, et fixer la précision décimale à deux chiffres après la virgule.
  * Ajouter un composant Map dans la boucle for-Each après la conversion de devise, puis agrandir le panneau inférieur
  * Ajouter une variable temporaire pour faire le mapping de la facture actuelle, en faisant ce qui suit:
    * Faire un clic droit sur n'importe quelle variable du panneau de droite et sélectionner Extract puis coller le texte JSON suivant qui représente le résultat désiré pour notre facture. Cliquer ensuite sur Copy Node

    ```json
    {
      "invnum": "IN4001",
      "invdate": "2023-01-26",
      "businessname": "ACME Corp",
      "billtoname": "Cesar Bowman",
      "vat": "15%",
      "totalamt": 500.00,
      "currency": "USD",
      "status": "Paid"
    }
    ```
    * Faire un Clic droit sur n'importe quelle variable du panneau de droite et sélectionner Paste. Donner un nom à la variable `invoiceResponse`
  * Cliquer dessus et dérouler cette variable
  * Dérouler `currencyConvertResponse` du panneau de gauche
    * Ajouter une fonction de mapping en utilisant le bouton '+fx', sélectionner DecimalPrecision dans la catégorie Math
    * Tirer une ligne de  `currencyConvertResponse->result` vers `decimal`
    * Régler (Set value) la `precision` à 2
    * Tirer une ligne de `output` à `invoiceResponse->totalamt`
      ![map1](../images/lab4-map1.png)
    * Cliquer sur la fonction DecimalPrecision pour réduire sa taille et continuer le mapping
    * Tirer une ligne de `currencyConvertResponse->query->to` vers `invoiceResponse->currency` sur le panneau de droite 
  * Dérouler `GetInvoicesByStatusOutput->response_resultSet` dans le panneau de gauche et tirer des lignes de :
    * `invoice_invnum` vers `invoiceResponse->invnum` dans le panneau de droite
    * `invoice_invdate` vers `invoiceResponse->invdate` dans le panneau de droite
    * `invoice_businessname` vers `invoiceResponse->businessname` dans le panneau de droite
    * `invoice_billtoname` vers `invoiceResponse->billtoname` dans le panneau de droite
    * `invoice_vat` vers `invoiceResponse->vat` dans le panneau de droite
    * `invoice_status` vers `invoiceResponse->status` dans le panneau de droite
  * Cliquer sur Save
  ![map1](../images/lab4-map1_.png)
* Ajoutons ensuite la facture convertie à la liste des réponses et calculons le total global des réponses.
  * Ajouter un autre composant Map après le précédent et développer le panneau inférieur.
  * Ajouter une fonction AppendList de la catégorie List.
    * Tirer une ligne de `response->invoices[]` sur la gauche, vers `docList`
    * Tirer une ligne de `invoiceResponse` à `docIn`
    * Tirer une ligne de `docList` à `response->invoices[]` sur la droite
    ![map2](../images/lab4-map2-AppendList.png)
  * Ajouter une fonction AddFloats
    * Tirer une ligne de `response->grandTotal` à `num1`
    * Tirer une ligne de `InvoiceResponse->totalamt` à `num2`
    * Tirer une ligne de `output` à `response->grandTotal`
    ![map2](../images/lab4-map2-addfloats.png)
  * Compléter le champ de réponse
    * Tirer une ligne de `HTTPSServerGetOutput->queryParams->currencycode` sur la gauche, vers `response->currency` sur la droite
    * Tirer une ligne de `HTTPSServerGetOutput->queryParams->status` sur la gauche, vers  `response->status` sur la droite
    * Fixer la valeur (Set value) de `response->success` à `true`
    ![map2 addFloats](../images/lab4-map2.png)
  * Cliquer sur Save

L'intégration doit ressembler à ceci:
![integration](../images/lab4-integration.png)

* Activer l'intégration en faisant un appel API depuis un navigateur, Postman ou depuis Curl avec le même URL que précédemment comme suit:
  
  > Passer la souris sur l'icône du lien pour voir l'URL nécessaire à l'appel API et copier le lien.
  
  ![alt text](/apification/images/image.png)
  
  > Veiller à mettre à jour le chemin de ressource "/invoices" pour qu'il corresponde à ce qui a été défini. Par exemple, dans notre cas d'utilisation, le statut est "Overdue" et le code de la devise est "EUR". Le chemin de la ressource doit donc être "/invoices?status=Overdue&currencycode=EUR".

  > Effectuez un appel l'API depuis votre navigateur, Postman ou curl avec la commande suivante:   
  
  ```bash
  curl --location --request GET "<YOUR URL>"
  ```

Le résultat devrait ressembler à:

  ```json
  {
      "grandTotal": 1072.19,
      "invoices": [
          {
              "invnum": "IN4001",
              "invdate": "2023-07-26",
              "businessname": "ACME Corp",
              "billtoname": "Cesar Bowman",
              "vat": "15%",
              "status": "Overdue",
              "currency": "EUR",
              "totalamt": 466.17
          },
          {
              "invnum": "IN4003",
              "invdate": "2023-09-24",
              "businessname": "Crypto Corp",
              "billtoname": "Jane Doe",
              "vat": "15%",
              "status": "Overdue",
              "currency": "EUR",
              "totalamt": 606.02
          }
      ],
      "currency": "EUR",
      "status": "Overdue",
      "success": true
  }
  ```

## Étape 5 - Relevez le défi!

Ajouter une Authentification de base à votre API puis la tester de nouveau
