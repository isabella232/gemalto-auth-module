![Gemalto Logo](/images/logo-gemalto-340x120.gif)

# Gemalto Authentication Solution
Document version: 1.0 (July 2018)

Gemalto’s Digital Banking solutions for eBanking and eCommerce help banks take advantage of the digital transformation by ensuring customer trust and regulatory compliance. By offering frictionless and convenient strong authentication combined with risk management, Gemalto’s eBanking solutions protect online banking customers from external attacks and guarantee an ideal balance between a user-friendly, and secure online experience.

These solutions are now intergrated into ForgeRock Identity Platform.

![ForgeRock Gemalto integration](/images/gto_overview.png)

You will find below the steps to integrate Access Management with the Gemalto’s Digital Banking solutions.

## Pre-requisites

### Gemalto solution
For this integration, the Gemalto federation interface needs to running. It is the component that exposes Gemalto solutions via SAMLv2, OAuth 2, and OpenID Connect.
A new OpenID Connect client needs to be created. The following inputs will be needed for this guide:
* **OpenID Discovery URL** - example: ```"https://gemalto.mybank.com/auth/realms/dbanking/.well-known/openid-configuration"```
* **Client ID** - entered when creating the client in Gemalto systems - example: ```"forgerock"``` 
* **Client Secret** - a randomely generated secret used during the OpenID Connect exchange: example: ```"2ggse304-wg34-9nwx-193j-3jud85jdwpzm"```

### ForgeRock Access Management
This guide is targeting Access Management (AM) version 6.0+. It can be deployed using this set of [instructions](https://backstage.forgerock.com/docs/am/6/quick-start-guide/).

ForgeRock 5.5+ can also be used with the same type of integration.

Access Management must be able to connect to Gemalto's solution.

## Add Gemalto solution as an OpenID Connect provider
In this section, ForgeRock wizard **"Configure Social Authentication"** is used. This will automatically create 2 component:
* An **"Authentication Module"** named "GemaltoSocialAuthentication"
* An **"Authentication Chain"** named "GemaltoSocialAuthenticationService"

Note: It is also possible to create the module and chain manually if needed.

**Steps**
1. Login to the Access Management console and select the realm.
1. In the **"Realm Overview"**, select **"Configure Social Authentication"**
1. Select **"Configure Other Authentication"**
1. Fill the form 
    + Provider details
        + OpenID Discovery URL ```"<See pre-requisites>"```.
        + Provider Name ```"Gemalto"```
        + Image URL/Path: https://www.gemalto.com/_catalogs/masterpage/Gemalto/assets/logo-gemalto-340x120.gif
    + Client Details
        + Client ID ```"<See pre-requisites>"```
        + Client Secret ```"<See pre-requisites>"```
        + Confirm Client Secret ```"<See pre-requisites>"```
        + Redirect URL: the default one should be correct
1. Click **"Create"** (at the top right)

![ForgeRock Wizard](/images/forgerock-wizard.png)


### Configure pre-created Authentication Module
The Authentication Module that was automatically created requires some changes to function properly. Here are the steps to finish its configuration.
1. Open the pre-created module via the left menu: **"Authentication > Modules"**. It will be at the bottom of the module list **"GemaltoSocialAuthentication"**
1. Click on it to view its properties. We will update the "Core" and "Account Provisioning" tabs.
1. In the **Core tab**, modify the following. Keep other entries as-is.
    + Scope ```"openid profile email"```
    + Subject Property ```"preferred_username"```
    + OAuth 2.0 Provider Logout Service: 
        + Copy the URL from "Access Token Endpoint URL" and paste it in this field. Replace **"token"** by **"logout"** at the end of the URL. Example: 
            + Access Token Endpoint URL: https://gemalto.mybank.com/auth/realms/dbanking/protocol/openid-connect/token
            + OAuth 2.0 Provider Logout Service: https://gemalto.mybank.com/auth/realms/dbanking/protocol/openid-connect/logout
    + Logout Options ```"logout"```
1. Click **"Save Changes"**    

![Sample of Module Core tab](/images/module-core.png)

5. In the **Account Provisioning tab**, modify the following. Keep other entries as is.
    + Create account if it does not exist ```"checked"```
    + Account Mapper Configuration: Remove existing entry, add ```"preferred_username=uid"```
    + Attribute Mapper Configuration: Modify entry to look like this:
        + ```"given_name=givenName preferred_username=uid family_name=sn name=cn email=mail"``` (only the uid entry is modified)
1. Click **"Save Changes"**    

![Sample of Module Account Provisioning tab](/images/module-accountprovisionning.png)

### Configure pre-created Authentication Chain
The Authentication Chain that was automatically created requires some changes for the logout feature to work properly. Here are the steps to finish its configuration
1. Open the pre-created chain via the left menu: **"Authentication > Chains"**. It will be at the bottom of the module list **"GemaltoSocialAuthenticationService"**.
1. Click on it to view its properties.
1. Open the **Settings tab**.
1. In the **"Post Authentication Processing Class"**, add the following "CLASS NAME": ```"org.forgerock.openam.authentication.modules.oauth2.OAuth2PostAuthnPlugin"```

![Sample of Chain Setting tab](/images/chain-settings.png)

## Testing your configuration
Access Management provides a quick way to test your setup at this stage.
1. Trigger the chain using a browser by using its service name
Example: https://openam.mybank.com/openam/XUI/#login&service=gemaltoSocialAuthenticationService

2. This should automatically redirect you to the Gemalto login page via an OpenID Connect redirection

![Demo login page](/images/demo-loginpage.png)

3. Login to the Gemalto authentication solution using one of the configured authentication method.

You should be redirected to the Access Management user portal

![ForgeRock User Portal](/images/demo-userpage.png)

4. Logout of the User Portal using the top right menu.

## Disclaimer
Gemalto hereby disclaims all warranties and conditions with regard to the information contained herein, including all implied warranties of merchantability, fitness for a particular purpose, title and non-infringement. In no event shall Gemalto be liable, whether in contract, tort or otherwise, for any indirect, special or consequential damages or any damages whatsoever including but not limited to damages resulting from loss of use, data, profits, revenues, or customers, arising out of or in connection with the use or performance of information contained in this document.
