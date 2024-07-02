# Amplify Integration - Guide de connexion Hubspot

Ce guide décrit le processus de création d'une connexion Hubspot pour Amplify Integration

Nous ferons ce qui suit:

* Créer une App Hubspot Connectée et configurer les paramètres OAuth
* Créer une connexion Amplify Integration en utilisant ces paramètres
* Tester notre nouvelle connexion 

## App connectée Hubspot Hubspot

* Si vous n'avez pas de compte Hubspot, créez en un à l'adresse suivante: **[https://www.hubspot.com/](https://www.hubspot.com/)**. Passez tous les écrans de bienvenue jusqu'à atteindre la page d'accueil de Hubspot
  ![hubspot00](../assets/../assets/hubspot-connection/hubspot00.png)
* Si vous n'avez pas d'App de développeur Hubspot, créez en une à l'adresse suivante: [**https://developers.hubspot.com/**](https://developers.hubspot.com/) , choisissez l'option App Developer et passez tous les écrans de bienvenue jusqu'à atteindre la page d'accueil de développeur Hubspot
  ![hubspot01](../assets/hubspot-connection/hubspot01.png)
* Sur votre compte développeur, cliquez sur Manage Apps puis cliquez sur Create App pour créer une nouvelle App
  ![hubspot02](../assets/hubspot-connection/hubspot02.png)
* Donnez un nom à votre App (par ex: AIP App), un logo et une description (facultatif)
* Cliquez sur la fenêtre Auth pour voir votre Client ID et votre Client Secret. Vous en aurez besoin pour paramétrer votre Connexion Hubspot dans Amplify Integration
  ![hubspot03](../assets/hubspot-connection/hubspot03.png)
* Faites défiler vers le bas jusqu'aux URLs de redirection puis entrez l'URL de redirection suivant, puis appuyez sur Save:
  `<<Nom de domaine de l'interface de Designe d'Amplify Integration>>/design/oauth2/callback`
  ![hubspot04](../assets/hubspot-connection/hubspot04.png)
* Faites défiler vers le bas jusqu'à la section Scopes et sélectionnez les scopes souhaités. Par exemple, vous pouvez sélectionner les 4 Scopes CRM en lien avec les clients:
  ![hubspot05](../assets/hubspot-connection/hubspot05.png)
* Nous aurons besoin de la liste des Scopes lorsque nous créerons la connexion Hubspot sur Amplify Integration Hubspot. Pour cet exemple, ce sont les suivants:
`crm.schemas.contacts.write crm.objects.contacts.write crm.schemas.contacts.read crm.objects.contacts.read oauth`
## Connexion Hubspot sur Amplify Integration

* Créez une nouvelle connexion, sélectionnez Hubspot et donnez-lui un nom et une description
  ![hubspot06](../assets/hubspot-connection/hubspot06.png)
  ![hubspot07](../assets/hubspot-connection/hubspot07.png)
* Entrez votre Client ID, Client Secret et la liste des Scopes provenant de l'étape précédente puis cliquez sur Update
  ![hubspot08](../assets/hubspot-connection/hubspot08.png)
* Cliquez sur Generate Token pour vous authentifier, renseignez vos informations de connexion, sélectionnez votre compte non-développeur puis cliquez sur choose Account
  ![hubspot09](../assets/hubspot-connection/hubspot09.png)
  ![hubspot10](../assets/hubspot-connection/hubspot10.png)
  ![hubspot11](../assets/hubspot-connection/hubspot11.png)
* Faites défiler vers le bas et cliquez sur Connect App
  ![hubspot12](../assets/hubspot-connection/hubspot12.png)
  ![hubspot13](../assets/hubspot-connection/hubspot13.png)
* Cliquez sur Test
  ![hubspot14](../assets/hubspot-connection/hubspot14.png)

Votre connexion peut désormais être utilisée dans une intégration. Mais avant cela, testons-la

## Test de la Connexion

* Créez une intégration et ajoutez un évènement déclencheur Scheduler et réglez le à n'importe quelle valeur étant donné que nous n'allons pas l'activer
  ![hubspot15](../assets/hubspot-connection/hubspot15.png)
  ![hubspot16](../assets/hubspot-connection/hubspot16.png)
* Cliquez sur le bouton `+` pour ajouter un composant Hubspot Get All à votre intégration
  ![hubspot17](../assets/hubspot-connection/hubspot17.png)
  ![hubspot18](../assets/hubspot-connection/hubspot18.png)
* Étendez le panneau inférieur et sélectionnez le connecteur Hubspot créé précedemment 
  ![hubspot19](../assets/hubspot-connection/hubspot19.png)
* Nous avons maintenant besoin de créer un plug, pour cela cliquez sur Add à côté de Plugs. Donnez un nom puis cliquez sur Create
  ![hubspot20](../assets/hubspot-connection/hubspot20.png)
  ![hubspot21](../assets/hubspot-connection/hubspot21.png)
* Cliquez sur Configure, selectionnez le connecteur, choisissez Get All pour les Actions, contacts pour les Objects, sélectionnez les champs désirés (firstname, lastname, ...) cliquez sur Generate puis sur Save
  ![hubspot22](../assets/hubspot-connection/hubspot22.png)
* Retournez à l'intégration, sélectionnez le plug puis cliquez sur Save
  ![hubspot23](../assets/hubspot-connection/hubspot23.png)
* Cliquez sur Test pour essayer l'intégration
  ![hubspot24](../assets/hubspot-connection/hubspot24.png)
* Cliquez sur Hubspot GetAll, les contacts Hubspot apparaissent
  ![hubspot25](../assets/hubspot-connection/hubspot25.png)
