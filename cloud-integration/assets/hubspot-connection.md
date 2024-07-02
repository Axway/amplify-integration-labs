# Amplify Integration - Hubspot Connection Guide

This guide describes how to create an Amplify Integration Hubspot Connection.

We will do the following:

* Create a Hubspot Connected App and configure the OAuth settings
* Create an Amplify Integration Hubspot Connection using these settings
* Test our new Amplify Integration Hubspot Connection

## Hubspot Connected App

* If you don't have a Hubspot account, create one at **[https://www.hubspot.com/](https://www.hubspot.com/)**. Skip all welcome wizards until you reach the Hubspot homepage
  ![hubspot00](hubspot-connection/hubspot00.png)
* If you don't have a Hubspot App Developer account, create one at [**https://developers.hubspot.com/**](https://developers.hubspot.com/) , choose App Developer option and skip all welcome wizzards if any until you reach the Hubspot Developer homepage
  ![hubspot01](hubspot-connection/hubspot01.png)
* In your developer account, click on Manage Apps and click the Create App button to create a new app
  ![hubspot02](hubspot-connection/hubspot02.png)
* Give your app a name (e.g. AIP App) and optional Logo and Description
* Click on the Auth tab to see your Client ID and Client Secret. You will need these when you setup the Amplify Integration Hubspot Connection
  ![hubspot03](hubspot-connection/hubspot03.png)
* Scroll down to the Redirect URLs and enter the following Redirect URL and press Save:
  `<<YOUR AIP BASE URL>>/design/oauth2/callback`
  ![hubspot04](hubspot-connection/hubspot04.png)
* Scroll down to the Scopes section and select your desired scopes. For example, I selected the contact CRM scopes below:
  ![hubspot05](hubspot-connection/hubspot05.png)
* We will need the list of selected scopes when we create the Amplify Integration Hubspot Connection. For my settings, they are as follows:
`crm.schemas.contacts.write crm.objects.contacts.write crm.schemas.contacts.read crm.objects.contacts.read oauth`

## Amplify Integration Hubspot Connection

* Create a new Connection and select Hubspot and give it a name and description
  ![hubspot06](hubspot-connection/hubspot06.png)
  ![hubspot07](hubspot-connection/hubspot07.png)
* Enter your Hubspot App Client ID, Client Secret and Scope list from above and click on Update
  ![hubspot08](hubspot-connection/hubspot08.png)
* Click on the Generate Token button to authenticate, enter your credentials, select the non developer account and click on Choose Account
  ![hubspot09](hubspot-connection/hubspot09.png)
  ![hubspot10](hubspot-connection/hubspot10.png)
  ![hubspot11](hubspot-connection/hubspot11.png)
* Scroll down and click on the Connect App button
  ![hubspot12](hubspot-connection/hubspot12.png)
  ![hubspot13](hubspot-connection/hubspot13.png)
* Click the Test button
  ![hubspot14](hubspot-connection/hubspot14.png)

Now your Connection can be used in an integration but first let's test it out.

## Test the Connection

* Create an integration and add a Scheduler trigger event and set to any value since we won't be turning it on
  ![hubspot15](hubspot-connection/hubspot15.png)
  ![hubspot16](hubspot-connection/hubspot16.png)
* Click the `+` button and add a Hubspot Get All component to the integration
  ![hubspot17](hubspot-connection/hubspot17.png)
  ![hubspot18](hubspot-connection/hubspot18.png)
* Expand the bottom panel and select the Hubspot Connector you created above
  ![hubspot19](hubspot-connection/hubspot19.png)
* Now we need to create a Plug so click on the Add button next to Plugs. Provide a name and click on the Create button
  ![hubspot20](hubspot-connection/hubspot20.png)
  ![hubspot21](hubspot-connection/hubspot21.png)
* Click on Configure, select the connector, Get All for the Actions, contacts for the Objects, select the desired fields (firstname, lastname, ...) and press Generate and then press Save
  ![hubspot22](hubspot-connection/hubspot22.png)
* Back in the integration, select the plug and then click the save button
  ![hubspot23](hubspot-connection/hubspot23.png)
* Click the Test button to try the integration
  ![hubspot24](hubspot-connection/hubspot24.png)
* Click on the Hubspot GetAll step and see the Hubspot contacts
  ![hubspot25](hubspot-connection/hubspot25.png)
