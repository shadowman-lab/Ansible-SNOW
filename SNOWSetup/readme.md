# Connecting ServiceNow and Ansible Automation Platform (AAP)

## Sections
[ServiceNow/AAP Integration Instructions using Rest Messages](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#servicenowaap-integration-instructions-using-rest-messages)

[ServiceNow/AAP Integration Instructions using Ansible Spoke](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#servicenowaap-integration-instructions-using-ansible-spoke)

[ServiceNow/AAP Integration Instructions using Event-Driven Ansible Notification Service](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#servicenowaap-integration-instructions-using-event-driven-ansible-notification-service)

[ServiceNow Basic Auth Connection Configuration](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#servicenow-basic-auth-connection-configuration)

[Have AAP reach out to ServiceNow](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#have-aap-reach-out-to-servicenow)

[Have AAP use ServiceNow as an inventory source](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#have-aap-use-servicenow-as-an-inventory-source)

## Connect ServiceNow to Ansible AAP

## Notes
- These instructions assume that there is no MID-Server for ServiceNow, and that the ServiceNow instance and AAP can talk to each other directly over the public internet.
- This has been tested with:
  - Ansible Tower 3.6, 3.7, 3.8, AAP 2.0, 2.1, 2.2, 2.3, 2.4, 2.5
  - ServiceNow Orlando, Paris, Quebec, Vancouver

- While the mid-server is an outbound connection from on-prem to the customer’s ServiceNow Instance, it subscribes to the “ECC Queue” allowing for bidirectional communication between an on-prem AAP and ServiceNow. Because it is polling, there can be a delay between the initiation of an action and the mid-server processing the request.

- If you create outbound REST messages in ServiceNow you can choose to have that executed directly from the ServiceNow instance or via a mid-server.

- If you have a subscription to ServiceNow’s Standard IntegrationHub pack, there is a spoke for integrating with AAP so you don't have to write you own API requests.

## ServiceNow/AAP Integration Instructions using Rest Messages

## Notes
- ServiceNow MID Servers do not support OAuth, you must use basic authentication. Skip steps 1-3, 6-11. In step 12, click Authentication Type of Basic. Click the Magnifying Glass next to the Basic Auth Profile and create a new profile with a valid AAP username and password. Skip step 13. In step 14, under HTTP Request, ensure you select the desired MID Server next to Use MID Server.

### Preparing AAP

#### 1)
In AAP 2.4 and older, navigate to **Applications** on the left side of the screen and then click the **Blue Add Button** on the right, which will present you with a Create Application dialog screen. In AAP 2.5 and newer, navigate to **Access Management -> OAuth Applications** on the left side of the screen and then click the **Blue Create OAuth application Button** on the top, which will present you with a Create Application dialog screen. Fill in the following fields:
| Parameter | Value |
|-----|-----|
| Name  | Descriptive name of the application that will contact AAP  |
|  Organization |  `Default` |
|  Authorization Grant Type |  `Authorization code` |
|  Redirect URIs |  `https://<snow_instance_id>.service-now.com/oauth_redirect.do` |
|  Client Type |  `Confidential` |

<img src="images/create_application.png" alt="AAP Create Application" title="AAP Create Application" width="1000" />

#### 2)
Click the blue **Save** button in AAP 2.4 and older or the blue **Create OAuth application** in AAP 2.5 and newer, at which point a window will pop up, presenting you with the Client ID and Client Secret needed for ServiceNow to make API calls into AAP. This will only be presented **ONCE**, so capture these values for later use.

<img src="images/application_secrets.png" alt="AAP Application Secrets" title="AAP Application Secrets" width="500" />

#### 3)
Next, in AAP 2.4 or older navigate to **Settings** on the left side of the screen and then **Miscellaneous Authentication settings**. After you click Edit at the bottom, you’ll want to toggle the **Allow External Users to Create Oauth2 Tokens** option to ***on***. Click the blue **Save** button to commit the change.

In AAP 2.5 or newer navigate to **Settings -> Platform gateway** on the left side of the screen. After you click Edit platform gateway settings at the top right, you’ll want to set enabled for the **Allow External Users to Create Oauth2 Tokens** option. Click the blue **Save platform gateway settings** button to commit the change.

## Note
- This is only needed if using a non-local user within automation controller for the integration.

<img src="images/tower_settings.png" alt="AAP Settings" title="AAP Settings" width="1000" />



#### 4)
The Orlando release of the ServiceNow developer instance does not allow for the self-signed certificate provided by AAP. We need to equip our AAP instance with a certificate from a trusted Certificate Authority. The easiest way to accomplish this is to SSH into AAP and run the Certbot ACME client in order to generate a certificate from LetsEncrypt (instructions can be found [here](https://letsencrypt.org/getting-started/)). It is important to place the contents of the root certificate + the intermediate certificate + the certificate you generate (found at `/etc/letsencrypt/live/<tower domain>/cert.pem`) at the location AAP places its self-signed certificate, `/etc/tower/tower.cert`. The LetsEncrypt intermediate certificate can be found [here](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt). You must also replace the contents of `/etc/tower/tower.key` with the contents of `/etc/letsencrypt/live/<tower domain>/privkey.pem`.

Be sure to restart the nginx service on your AAP server after updating the certificate and key.

### Preparing ServiceNow

#### 5)
Moving over to ServiceNow, Navigate to **System Definition-->Certificates**. This will take you to a screen of all the certificates Service Now uses. Click on the **blue New button**, and fill in these details:
| Parameter | Value |
|-----|-----|
| Name  | Descriptive name of the certificate  |
|  Format |  `PEM` |
|  Type |  `Trust Store Cert` |
|  PEM Certificate |  The certificate to authenticate against AAP with. Use the certificate you just generated on your AAP server, located at `/etc/tower/tower.cert.` Copy the contents of this file into the field in ServiceNow. |

<img src="images/tower_cert.png" alt="AAP Certificate" title="AAP Certificate" width="1000" />

Click the **Submit** button at the bottom.

#### 6)
In ServiceNow, Navigate to **System OAuth-->Application Registry**. This will take you to a screen of all the Applications ServiceNow communicates with. Click on the **blue New button**, and you will be asked What kind of Oauth application you want to set up. Select **Connect to a third party Oauth Provider**.

<img src="images/snow_app_reg.png" alt="SNOW Application Registry" title="SNOW Application Registry" width="800" />

#### 7)
On the new application screen, fill in these details:

<img src="images/snow_app_reg_deets.png" alt="SNOW Application Registry Details" title="SNOW Application Registry Details" width="800" />

| Parameter | 2.4 and older Value | 2.5 and newer Value |
|-----|-----|-----|
| Name  | Descriptive Application Name  | Descriptive Application Name  |
|  Client ID |  The Client ID you got from AAP | The Client ID you got from AAP |
|  Client Secret |  The Client Secret you got from AAP | The Client Secret you got from AAP |
|  Default Grant Type |  `Authorization Code` | `Authorization Code` |
|  Authorization URL |  `https://<aap_url>/api/o/authorize/` | `https://<unified_ui_url>/o/authorize/` |
|  Token URL |  `https://<aap_url>/api/o/token/` | `https://<unified_ui_url>/o/token/` |
|  Redirect URL |  `https://<snow_instance_id>.service-now.com/oauth_redirect.do` |  `https://<snow_instance_id>.service-now.com/oauth_redirect.do` |
|  Send Credentials |  `As Basic Authorization Header` |  `As Basic Authorization Header` |

Click the **Submit** button at the bottom.

#### 8)
You should be taken out to the list of all Application Registries. Click back into the Application you just created. At the bottom, there should be two tabs: Click on the tab **Oauth Entity Scopes**. Under here, there is a section called **Insert a new row…**. Double click here, and fill in the field to say Writing Scope. Click on the **green check mark** to confirm this change. Then, right-click inside the grey area at the top where it says Application Registries and click Save in the menu that pops up.

<img src="images/write_scope.png" alt="SNOW Write Scope" title="SNOW Write Scope" width="800" />

#### 9)
The writing scope should now be Clickable. Click on it, and in the dialog window that you are taken to, type **write** in the Oauth scope box. Click the **Update** button at the bottom.

<img src="images/write_scope_deets.png" alt="SNOW Write Scope" title="SNOW Write Scope" width="800" />

#### 10)
Back in the Application Settings page, scroll back to the bottom and click the **Oauth Entity Profiles** tab. There should be an entity profile populated - click into it.

<img src="images/oauth_entity.png" alt="Oauth Entity" title="Oauth Entity" width="500" />

#### 11)
You will be taken to the Oauth Entity Profile Window. At the bottom, Type Writing Scope into the Oauth Entity Scope field. Click the green check mark and **update**.

<img src="images/oauth_entity_deets.png" alt="Oauth Entity" title="Oauth Entity" width="800" />

#### 12)
Navigate to **System Web Services-->Outbound-->REST Messages**. Click the blue **New** button. In the resulting dialog window, fill in the following fields:

<img src="images/rest_message.png" alt="REST Message" title="REST Message" width="800" />

| Parameter | 2.4 and older Value | 2.5 and newer Value |
|-----|-----|-----|
| Name  | `Provision Cloud Webservers with Users`  |`Provision Cloud Webservers with Users`  |
|  Endpoint |  The url endpoint of the AAP action you wish to do. This can be taken from the browsable API at `https://<aap_url>/api/v2/` |  The url endpoint of the AAP action you wish to do. This can be taken from the browsable API at `https://<unified_ui_url>/api/controller/v2/` |
|  Authentication Type |  `Oauth 2.0` |  `Oauth 2.0` |
|  Oauth Profile |  Select the Oauth profile you created | Select the Oauth profile you created |

Right-click inside the grey area at the top; click **Save**.

#### 13)
Click the **Get Oauth Token** button on the REST Message screen. This will generate a pop-up window asking to authorize ServiceNow against your AAP instance/cluster. Click Authorize. ServiceNow will now have an Oauth2 token to authenticate against your AAP server.

Note: The ServiceNow user MUST be able to access the ServiceNow API (check if the user you are logged into ServiceNow with has API access)

Note: If you wish to have AAP use a specific user when reaching out from ServiceNow (such as a dedicated servicenow user) ensure you are logged in as that user when you click Authorize. You can utilize a System Administrator or a Normal User as this user. If you are using a Normal User, ensure they have execute access on any Job Templates or Workflow Job Templates you intend to run.

<img src="images/snow_authorize.png" alt="SNOW Authorize" title="SNOW Authorize" width="500" />

#### 14)
Under the HTTP Methods section at the bottom, click the blue New button. At the new dialog window that appears, fill in the following fields:

<img src="images/snow_http_method.png" alt="SNOW HTTP Method" title="SNOW HTTP" width="1000" />

- **HTTP Method**: `POST`
- **Name**: Descriptive HTTP Method Name
- **Endpoint**: The url endpoint of the AAP action you wish to do. This can be taken from the browsable AAP 2.4 and older API at `https://<aap_url>/api/v2/` or `https://<unified_ui_url>/api/controller/v2/` in AAP 2.5 and newer for example `https://<unified_ui_url>/api/controller/v2/job_templates/41/launch`
- **HTTP Headers**: ***(under the HTTP Request tab)***
  - The only HTTP Header that should be required is `Content-Type: application/json`
- **HTTP Query Parameters**: ***(under the HTTP Request tab)***
  - If you wish to pass user-provided variables into your Job/Workflow Template, you can add them into the content field. The variable value needs to be in the format `${varname}`. An example for this demonstration:

```
{
	"extra_vars":
	{
		"num_instances": ${num_instances},
		"instance_size": "${instance_size}",
		"cloud_provider": "${cloud_provider}",
		"from_snow": true
	}
}
```
**NOTE** None of the variables are required, this is simply an example. They should match your Job Template or Workflow requirements

**NOTE** `from_snow` is hard coded to be true; we do not want the user to change this value as this request is in fact coming from ServiceNow.

**NOTE** For the extra variables to be received by controller, one of the following must be true for the Job Template:
They correspond to variables in an enabled survey.
Prompt on launch for Varibles (or ask_variables_on_launch in the API) is set to True.

#### 15)
As we user-provided parameters in the Content field, click on Auto-generate variables in order to generate variables for test runs. Populate the Test value column with some default values that you would like to test your call with (see below for an example). You can then kick off a RESTful call to AAP using these parameters with the **Test** link.

<img src="images/snow_test_vars.png" alt="SNOW Test Vars" title="SNOW Test Vars" width="700" />

### Testing connectivity between ServiceNow and AAP

#### 16)
Clicking the **Test** link will take you to a results screen, which should indicate that the Restful call was sent successfully to AAP. In this example, ServiceNow kicks off an AAP job Template, and the response includes the Job ID in AAP:  2764

<img src="images/test_job_snow.png" alt="SNOW Test Job" title="SNOW Test Job" width="800" />

You can confirm that this Job Template was in fact started by going back to AAP and clicking the **Jobs** section on the left side of the screen; a Job with the same ID should be in the list

### Creating a ServiceNow Catalog Item to Launch an AAP Job Template

#### 17)
Now that you are able to make outbound RESTful calls from ServiceNow to AAP, it’s time to create a catalog item for users to select in ServiceNow in a production self-service fashion. While in the HTTP Method options, click the **Preview Script Usage** link:

<img src="images/api_script.png" alt="AAP API Script" title="AAP API Script" width="800" />

Copy the resulting script the appears, and paste it into a text editor to reference later.

#### 18)
In ServiceNow, navigate to **Workflow-->Workflow Editor**. This will open a new tab with a list of all existing ServiceNow workflows. Click on the blue **New Workflow** button:

<img src="images/workflow_editor.png" alt="Workflow Editor" title="Workflow Editor" width="800" />

#### 19)
In the New Workflow dialog box that appears, fill in the following options:

<img src="images/new_workflow.png" alt="Workflow" title="Workflow" width="800" />

| Parameter | Value |
|-----|-----|
| Name | `Provision Cloud Webservers with Users` |
| Table | `Requested Item [sc_req_item]` |

Everything else can be left alone. Click the **Submit** button.

#### 20)
The resulting Workflow Editor will have only a Begin and End box. Click on the line (it will turn blue to indicate it has been selected), then press **delete** on your keyboard to get rid of it.

<img src="images/workflow_diagram_initial.png" alt="Workflow Diagram Initial" title="Workflow Diagram Initial" width="600" />

#### 21)
On the right side of the Workflow Editor Screen, select the Core tab and, under **Core Activities-->Utilities**, drag the Run Script option into the Workflow Editor. In the new dialog box that appears, type in a descriptive name (the name of the RESTful call will suffice, `Provision Webservers with Users`), and paste in the script you captured from before. For each variable, replace the default values with ***current.variables.[variablename]*** (see example below). Click **Submit** to save the Script.

If you want to utilize the Service Now request number and pass it to Ansible for closed loop automation, add in **r.setStringParameterNoEscape('ticket_number', current.request.getDisplayValue());** Replace 'ticket_number' with the variable name utilized in the Rest Message.

<img src="images/workflow_script.png" alt="Workflow Script" title="Workflow Script" width="1000" />

#### 22)
Draw a connection from **Begin**, to the newly created Run Script Box, and another from the **Run Script** box to **End**. Afterward, click on the three horizontal lines to the left of the Workflow name, and select the **Publish** option. You are now ready to associate this workflow with a catalog item.

<img src="images/workflow_final.png" alt="Workflow Final" title="Workflow Final" width="600" />

#### 23)
Navigate to **Service Catalog-->Catalog Definitions->Maintain Items**. Click the blue **New** button on the resulting item list. In the resulting dialog box, fill in the following fields:

<img src="images/catalog_item.png" alt="Catalog Item" title="Catalog Item" width="1000" />

| Parameter | Value |
|-----|-----|
| Name | `Provision Cloud Webservers with Users` |
| Catalog | The catalog that this item should be a part of |
| Category | Required if you wish users to be able to search for this item |


In the Process Engine tab, populate the Workflow field with the Workflow you just created. Click the Submit Button. You’ve now created a new catalog item!

#### 24)
Navigate back to the Catalog Item settings, and at the bottom, click the **New** button under the variables tab. In the window that results, populate the question you want to present to the user, and the variable name. You can also put a default value under the Default Value Tab. If you select a variable type of `Multiple Choice`, after submitting the changes you can add options under the ***Question Choices*** section at the bottom of the Variable settings page.

<img src="images/catalog_vars.png" alt="Catalog Vars" title="Catalog Vars" width="1000" />

Here are the fields required for each variable in this demo:
##### cloud_provider
| Parameter | Value |
|-----|-----|
| Type | `Multiple Choice` |
| Question | `Which Cloud provider to provision into?` |
| Name | `cloud_provider` |
| Default value | `aws` |

###### variable options
| Text | Value | Order |
|-----|-----|-----|
| `Amazon Web Services` | `aws` | `100` |
| Google Cloud Platform | `gcp` | `200` |

##### num_instances
| Parameter | Value |
|-----|-----|
| Type | `Single Line Text` |
| Question | `How many instances should be spun up? (Any value from 1 through 10)` |
| Name | `num_instances` |
| Default value | `3` |

##### instance_size
| Parameter | Value |
|-----|-----|
| Type | `Multiple Choice` |
| Question | `What size instance should be selected?` |
| Name | `instance_size` |
| Default value | `small` |

###### variable options
| Text | Value | Order |
|-----|-----|-----|
| `small` | `small` | `100` |
| `medium` | `medium` |  `200` |
| `large` | `large` |  `300` |

#### 25)
Lastly, to run this catalog item, navigate to **Self-Service-->Homepage** and search for the catalog item you just created. Once found, click the **order now** button. You can see the results page pop up in ServiceNow, and you can confirm that the Job is being run in AAP.

<img src="images/catalog_order.png" alt="Catalog Item" title="Catalog Item" width="1000" />

Congratulations! After completing these steps, you can now use a ServiceNow Catalog Item to launch a Workflow Template in AAP. This is ideal for allowing end users to use a front end they are familiar with in order to perform this, and other automated tasks of varying complexities. This goes a long way toward reducing the time to value for the enterprise as a whole, rather than just the teams responsible for writing the playbooks being used.

## ServiceNow/AAP Integration Instructions using Ansible Spoke

This walkthrough assumes you have an Integration Hub Standard/Professional subscription and Ansible spoke activated. It also assumes you have the ability to reach your automation controller from ServiceNow (a mid-server can be utilized but only basic Auth will work). For this example, I will be utilizing an already existing Ansible Automation Platform (AAP) workflow that patches all of my Red Hat Enterprise Linux Servers and updates a ServiceNow Catalog Request. I will also be using Ansible Automation Platform 2.2 but this integration will work in Ansible Automation Platform 1.2 and any version of 2.x as well. Ansible spoke leverages the ServiceNow Flow Designer which can be easier to use when leveraging variables and building out the API Rest message.

## Notes
- ServiceNow MID Servers do not support OAuth, you must use basic authentication. Skip steps 1-3 and replace steps 6 and 7 with [ServiceNow Basic Auth Connection Configuration](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#servicenow-basic-auth-connection-configuration)

### Preparing AAP

#### 1)
In AAP 2.4 and older, navigate to **Applications** on the left side of the screen. Click the **Blue Add Button** on the right, which will present you with a Create Application dialog screen. Fill in the following fields:
| Parameter | Value |
|-----|-----|
| Name  | Descriptive name of the application that will contact AAP  |
|  Organization |  `Default` |
|  Authorization Grant Type |  `Authorization code` |
|  Redirect URIs |  `https://<snow_instance_id>.service-now.com/api/sn_ansible_spoke/ansible_oauth_redirect` |
|  Client Type |  `Confidential` |

<img src="images/create_application.png" alt="AAP Create Application" title="AAP Create Application" width="1000" />

In AAP 2.5 and newer, navigate to **Access Management -> OAuth Applications** on the left side of the screen and then click the **Blue Create OAuth application Button** on the top, which will present you with a Create Application dialog screen. Fill in the following fields:
| Parameter | Value |
|-----|-----|
| Name  | Descriptive name of the application that will contact AAP  |
|  Organization |  `Default` |
|  Authorization Grant Type |  `Authorization code` |
|  Redirect URIs |  `https://<snow_instance_id>.service-now.com/oauth_redirect.do` |
|  Client Type |  `Confidential` |

#### 2)
Click the blue **Save** button, at which point a window will pop up, presenting you with the Client ID and Client Secret needed for ServiceNow to make API calls into AAP. This will only be presented **ONCE**, so capture these values for later use.

<img src="images/application_secrets.png" alt="AAP Application Secrets" title="AAP Application Secrets" width="500" />

#### 3)
Next, navigate to **Settings** on the left side of the screen and then **Miscellaneous Authentication settings**. After you click Edit at the bottom, you’ll want to toggle the **Allow External Users to Create Oauth2 Tokens** option to ***on***. Click the blue **Save** button to commit the change.

## Note
- This is only needed if using a non-local user within automation controller for the integration.

<img src="images/tower_settings.png" alt="AAP Settings" title="AAP Settings" width="1000" />

#### 4)
The Orlando release of the ServiceNow developer instance does not allow for the self-signed certificate provided by AAP. We need to equip our AAP instance with a certificate from a trusted Certificate Authority. The easiest way to accomplish this is to SSH into AAP and run the Certbot ACME client in order to generate a certificate from LetsEncrypt (instructions can be found [here](https://letsencrypt.org/getting-started/)). It is important to place the contents of the root certificate + the intermediate certificate + the certificate you generate (found at `/etc/letsencrypt/live/<tower domain>/cert.pem`) at the location AAP places its self-signed certificate, `/etc/tower/tower.cert`. The LetsEncrypt intermediate certificate can be found [here](https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt). You must also replace the contents of `/etc/tower/tower.key` with the contents of `/etc/letsencrypt/live/<tower domain>/privkey.pem`.

Be sure to restart the nginx service on your AAP server after updating the certificate and key.


### Preparing ServiceNow

#### 5)
Moving over to ServiceNow, Navigate to **System Definition-->Certificates**. This will take you to a screen of all the certificates Service Now uses. Click on the **blue New button**, and fill in these details:
| Parameter | Value |
|-----|-----|
| Name  | Descriptive name of the certificate  |
|  Format |  `PEM` |
|  Type |  `Trust Store Cert` |
|  PEM Certificate |  The certificate to authenticate against AAP with. Use the certificate you just generated on your AAP server, located at `/etc/tower/tower.cert.` Copy the contents of this file into the field in ServiceNow. |

<img src="images/tower_cert.png" alt="AAP Certificate" title="AAP Certificate" width="1000" />

Click the **Submit** (or **Update** if you had a previous AAP certificate) button at the bottom.

### Set Up Ansible Spoke

If using a MID server, skip steps 6 and 7 and perform [ServiceNow Basic Auth Connection Configuration](https://github.com/shadowman-lab/Ansible-SNOW/tree/main/SNOWSetup#servicenow-basic-auth-connection-configuration)

#### 6)
Navigate to **Connections & Credentials-->Connection & Credential Aliases**. Click the existing "AnsibleTowerAlias" alias. In the resulting dialog window, ensure the following fields are filled in:

| Parameter | Value |
|-----|-----|
| Name  | `AnsibleTowerAlias`  |
|  Application |  Ansible Spoke |
|  Type |  Connection and Credential |
|  Connection Type |  HTTP |
|  Default Retry Policy |  Default HTTP Retry Policy |
|  Configuration Template |  Ansible |

<img src="images/alias.png" alt="Connection & Credential Aliases" title="Connection & Credential Aliases" width="800" />

#### 7)
Under Related Links select "Create New Connection & Credential" and enter in the following information:

| Parameter | 2.4 and older Value | 2.5 and newer Value |
|-----|-----|-----|
| Connection Name  | `<provider-name> Spoke Connection` | `<provider-name> Spoke Connection` |
|  Connection URL  |  `https://<aap_url>` |  `https://<unified_ui_url>` |
|  Credential Name |  `<provider-name> Spoke Credentials` |`<provider-name> Spoke Credentials` |
|  Application Registry Name |  `<provider-name>` | `<provider-name>` |
|  OAuth Client ID | The Client ID you got from AAP | The Client ID you got from AAP |
|  OAuth Client Secret | The Client Secret you got from AAP | The Client Secret you got from AAP |
|  OAuth Entity Profile Name | Ansible Entity Profile | Ansible Entity Profile |
|  OAuth Entity Scope | `write` | `write` |
|  Authorization URL |  `https://<aap_url>/api/o/authorize/` | `https://<unified_ui_url>/o/authorize/` |
|  Token URL |  `https://<aap_url>/api/o/token/` | `https://<unified_ui_url>/o/token/` |
|  OAuth Redirect URL |  `https://<snow_instance_id>.service-now.com/api/sn_ansible_spoke/ansible_oauth_redirect` | `https://<snow_instance_id>.service-now.com/oauth_redirect.do` |

<img src="images/connection_credential.png" alt="Connection & Credential" title="Connection & Credential" width="800" />

Select **Create and Get OAuth Token** to complete the Ansible spoke set up.  This will generate a window asking to authorize ServiceNow against your AAP instance/cluster. Click **Authorize**.

## NOTE In AAP 2.5 this will fail with an HTTP Error 401 - Unauthorized Error because of an API Scipt auto applied by ServiceNow. To fix this:

1) Go to **System OAuth-->Application Registry**. Select the Application Registry you just created. Delete the OAuth API Script field. Click **Update** at the top.

2) Navigate to **All > Connections & Credentials > Credentials**. Select your newly created credential. Under Related Links select **Get Oauth Token**. This will generate a window asking to authorize ServiceNow against your AAP instance/cluster. Click **Authorize**. This should now successfully generate a token

3) You will also need to adjust the API endpoint for AAP 2.5 for Jobs to launch properly. Navigate to **All > Connections & Credentials > Connections** and select your newly created connection. In the Attributes section, Version change **v2** to be **controller/v2** and click Update.

Note: The ServiceNow user MUST be able to access the ServiceNow API (check if the user you are logged into ServiceNow with has API access)

Note: If you wish to have AAP use a specific user when reaching out from ServiceNow (such as a dedicated servicenow user) ensure you are logged in as that user when you click Authorize. You can utilize a System Administrator or a Normal User as this user. If you are using a Normal User, ensure they have execute access on any Job Templates or Workflow Job Templates you intend to run. **Authorize**.

### Create a Catalog Item for Users

#### 8)
Navigate to **Service Catalog-->Catalog Definitions->Maintain Items**. Click the blue **New** button on the resulting item list. In the resulting dialog box, fill in the following fields:

| Parameter | Value |
|-----|-----|
| Name | `AAP Ansible Spoke Patch` |
| Catalog | The catalog that this item should be a part of |
| Category | Required if you wish users to be able to search for this item |

<img src="images/cat_item.png" alt="Catalog Item" title="Catalog Item" width="1000" />

#### 9)
Navigate back to the Catalog Item settings, and at the bottom, click the **New** button under the variables tab. In the window that results, populate the question you want to present to the user, and the variable name. You can also put a default value under the Default Value Tab. If you select a variable type of `Multiple Choice`, after submitting the changes you can add options under the ***Question Choices*** section at the bottom of the Variable settings page.

Here are the fields required for each variable in this demo:

##### exclude
| Parameter | Value |
|-----|-----|
| Type | `Single Line Text` |
| Question | `What packages would you like to exclude?` |
| Name | `exclude` |

<img src="images/spoke_cat_vars.png" alt="Catalog Vars" title="Catalog Vars" width="1000" />

### Update Spoke Actions For Workflow Job Templates

#### 10)
Navigate to **Process Automation-->Flow Designer**

<img src="images/flow_designer.png" alt="Flow Designer" title="Flow Designer" width="800" />

This will open up a new tab. Click Actions and in the Name section type "Launch Job Template" and hit enter. Select the Launch Job Template Action.

#### 11)
Once it loads, click the 3 dots in the top right and select "Copy Action"

<img src="images/copy_action.png" alt="Copy Action" title="Copy Action" width="800" />

Enter in a name and select Ansible Spoke as the Application in the box that opens.

| Parameter | Value |
|-----|-----|
| New action name | `Launch Workflow Job Template` |
| Application | `Ansible Spoke` |

In this new action, Create a new label "Workflow Job Template ID" set to Mandatory and delete "Job Template ID"

<img src="images/action_inputs.png" alt="Action Inputs" title="Action Inputs" width="800" />

Select Pre Processing on the left-hand side

<img src="images/input_vars.png" alt="Input Vars" title="Input Vars" width="800" />

In the Input Variables section change "job_template_id" to "workflow_job_template_id" and for value drag in Workflow Job Template ID from the right side under input variables

Update the script to utilize this new id:
```
(function execute(inputs, outputs) {
    inputs = new AnsibleUtils().trimStringInputs(inputs);
    outputs.workflow_job_template_id = inputs.workflow_job_template_id;
    if (inputs.data) {
        outputs.payload = new AnsibleUtils().validateJson(inputs.data);
    }
})(inputs, outputs);
```

<img src="images/input_script.png" alt="Input Script" title="Input Scipt" width="800" />

In the Output Variables section create a new label "workflow_job_template_id" set to Mandatory and delete "job_template_id"

<img src="images/output_vars.png" alt="Output Vars" title="Output Vars" width="800" />

Select Launch Workflow on the left-hand side

Scroll down to the Resource Path section and change the url portion from "job_templates" to "workflow_job_templates" and ensure you drag over from Pre Processing the workflow_job_template_id so the Resource path looks like "api/step->Launch Job Template->Version/workflow_job_templates/step->Pre Processing->workflow_job_template_id"

<img src="images/launch_workflow.png" alt="Launch Workflow" title="Launch Workflow" width="800" />

Click Save and then Publish

### Create Flow For A Catalog Item Request

#### 12)
Go back to the Flow Designer home, click home in the top left. Click on the blue **New**, and then Flow.

| Parameter | Value |
|-----|-----|
| Flow Name  | AAP Patching |
|  Application |  Global |

Click on the blue **Submit** Button

<img src="images/new_flow.png" alt="New Flow" title="New Flow" width="800" />

#### 13)
In the New Tab that appears, click Add a trigger, select Service Catalog, Select Advanced Options and select "Run flow in background (default)"

Click on the blue **Done** Button

<img src="images/trigger.png" alt="Trigger" title="Trigger" width="800" />

#### 14)
Under Actions, Select "Add an Action, Flow Logic or Subflow" then select Action. Select "Get Catalog Variables". Drag "Requested Item Record" from the right side into the "Submitted Request [Requested Item] box"

In "Template Catalog Items and Variable Sets [Catalog Items and Variable Sets]" Select your Catalog Item from before "AAP Ansible Spoke Patch" Select the variables you want to pass to AAP "exclude" and click the right arrow.

<img src="images/get_vars.png" alt="Get Vars" title="Get Vars" width="800" />

Click on the blue **Done** Button

#### 15)
Under Actions, Select "Add an Action, Flow Logic or Subflow" then select Action. Under the Ansible Spoke select "Launch Job Template" or "Launch Workflow Job Template" depending on which type you want to run.

Enter in the template ID number from AAP and any extra arguments the job requires (this includes limits, survey items, etc). For the arguments, you can drag the variables from step one or from the trigger into the appropriate position.
```
{
	"extra_vars":
	{
		"exclude": "DRAGGED FROM STEP1",
		"ticket_number": "DRAGGED FROM STEP1"
	}
}
```
Click on the blue **Done** Button

<img src="images/launch_job.png" alt="Launch Job" title="Launch Job" width="800" />

Click Save and then click Activate on the top bar

**NOTE** For the extra variables to be received by controller, one of the following must be true for the Job Template:
They correspond to variables in an enabled survey.
Prompt on launch for Varibles (or ask_variables_on_launch in the API) is set to True.

### Update Catalog With Newly Created Flow

#### 16)
Navigate back to **Service Catalog-->Catalog Definitions->Maintain Items** and select the item you created earlier. Click on **Process Engine** and then populate the Flow field with the Flow you just created.

Right-click inside the grey area at the top; click **Save**.

<img src="images/process_engine.png" alt="Process Engine" title="Process Engine" width="800" />

#### 17)
Lastly, to run this catalog item, navigate to **Self-Service-->Service Catalog** and search for the catalog item you just created. Once found, click the **Order Now** button. You can see the results page pop up in ServiceNow, and you can confirm that the Job is being run in AAP.

<img src="images/spoke_catalog.png" alt="Catalog Item" title="Catalog Item" width="1000" />

Congratulations! After completing these steps, you can now use a ServiceNow Catalog Item to launch a Template in AAP using Ansible Spoke. This is ideal for allowing end users to use a front end they are familiar with in order to perform this, and other automated tasks of varying complexities. This goes a long way toward reducing the time to value for the enterprise as a whole, rather than just the teams responsible for writing the playbooks being used.

## ServiceNow/AAP Integration Instructions using Event-Driven Ansible Notification Service

This walkthrough assumes you are able to install applications within your ServiceNow instance. It also assumes you have the ability to reach your Event-Driven Ansible Controller from ServiceNow (a mid-server can be utilized).

### Preparing ServiceNow

#### 1)
In ServiceNow, navigate to the **All** menu and select **System Applications -> All Available Applications -> All** which will navigate to the Application Manager. Search for **Event-Driven Ansible** and then click on the Store Application for Event-Driven Ansible Notification Service.

<img src="images/application_manager.png" alt="Application Manager" title="Application Manager" width="1000" />

On the Event-Driven Ansible Notification Service click install, select your version, Install Now, and then Click Install and let the process complete.

<img src="images/eda_notification_service.png" alt="Event-Driven Ansible Notification Service" title="Event-Driven Ansible Notification Service" width="1000" />

#### 2)
After the install is complete, you need to add the EDA role to the ServiceNow user who will be performing the configuration. Navigate to the **All** menu and select **System Security -> Users and Groups -> Users**. Select a user from the list or click New to create a new user. If needed, fill in the required and possibly other fields. If necessary, set a password. To assign roles, switch to the Roles tab and click **Edit**. In the Edit Member page search the collection for the **x_rhtpp_eda.admin** role and move it to the right to appear in the roles list. In addition, assign the **itil** and **itil_admin** roles to grant the user write access to the problem, problem task, configuration item, change request, and incident tables. This user can not have the **admin** role or you will be unable to save changes to the Event-Driven Ansible Notification Properties. Click **Save** to apply the assigned roles.

<img src="images/eda_user_role.png" alt="Event-Driven Ansible Role" title="Event-Driven Ansible Role" width="1000" />

### Preparing AAP

## Notes
This assumes you have already set up the token for automation controller within Event-Driven Ansible controller if using AAP 2.4. If using AAP 2.5 or newer step 4 will include creating a credential to run Templates.

#### 3)
Now we will create a basic rulebook in order to display the information sent by ServiceNow. Push this rulebook to a Git repository (ensure it is in a folder called rulebooks from the root of the project). The rulebook is where you will decide what events to monitor for and what actions to take (such as calling an existing Job Template or Workflow Job Template in automation controller). This example rulebook will listen on port 5003 and debug any notifications that appear to help us see what information is sent from ServiceNow.
```
- name: Listen for events on a webhook from ServiceNow
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5003

  rules:
    - name: Output ServiceNow Information
      condition: event.meta is defined
      action:
        debug:
```
A more detailed rulebook example with an https webhook source and a token for source verification including calling Workflow Job Template
```
---
- name: Listen for events on a webhook from ServiceNow
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5003
        certfile: /certs/shadowman_cert.cer
        keyfile: /certs/shadowman_private.key
        token: "{{ eda_auth_token }}"

  rules:
    - name: Respond to Node Exporter Down Incident
      condition: event.payload.short_description  == "Prometheus Node Exporter is down"
      action:
        run_workflow_template:
          name: "Automated Response Node Exporter Down"
          organization: "Security"
          job_args:
            extra_vars:
              vm_name: "{{ event.payload.u_vm_name }}"
              ticket_number: "{{ event.payload.number }}"
```

#### 4)
On AAP 2.4 After your rulebook has been pushed to Git, we will login to Event-Driven Ansible controller and go to Projects. Either sync an existing Project if you already have one or go to **+ Create Project** and provide a name and your SCM URL and click **Create Project**. Ensure the Project has succesfully synced.

Create a Rulebook Activation by going to **Rulebook Activations** and clicking **+ Create rulebook activation**. Give it a name, select your existing Project, the rulebook you previously created, and your Decision environment (the default Decision Environment will work). Click **Create rulebook activation**.

On the new page, click **History** and ensure the rulebook is successfully running. The rulebook must be running in order for the test in the next section to work (and also for EDA to receive events from ServiceNow)

<img src="images/eda_controller.png" alt="Event-Driven Ansible Controller" title="Event-Driven Ansible Controller" width="1000" />

On AAP 2.5 and newer, login to the Unified UI. Go to **Automation Decisions -> Projects**. Either sync an existing Project if you already have one or go to **+ Create Project** and provide a name and your SCM URL and click **Create Project**. Ensure the Project has succesfully synced.

Now we will create a Credential for the Event Stream to use. Go to **Automation Decisions -> Infrastructure -> Credentials**. Select **Create credential**. Enter a name, select an organization and select **ServiceNow Event Stream** as the Credential Type. Then enter in a token (this can be a randomly generated token from something like Bitwarden). Click **Create credential at the bottom**

Now we will create a Credential for the Automation Platform so Job and Workflow Templates can be launched. Go to **Automation Decisions -> Infrastructure -> Credentials**. Select **Create credential**. Enter a name, select an organization and select **Red Hat Ansible Automation Plaform** as the Credential Type. Enter in host `https://<unified_ui_URL>/api/controller/`, Username that exists in the platform and Password. Click **Create credential at the bottom**

Now go to **Automation Decisions -> Event Streams**. Click **Create event stream** at the top. Enter in a name, select an Organization, select the event stream type of **ServiceNow Event Stream** select the Credential you created in the previous step. Then click **Create event stream**

Now we will create the Rulebook Activation and attach it to the Event Stream. Go to **Automation Decisions -> Rulebook Activations** Select **Create rulebook activation**. Enter a name and Organization. Select the Project you previously created and then your ServiceNow rulebook. Select the Gear icon next to Event Streams which will open up a new dialog box. Select your Rulebook Source (which will be the header of your rulebook) and then your Event Stream (your previously created ServiceNow Event Stream) and click **Save**. Click the magnifying glass next to Credential and select the Red Hat Ansible Automation Platform credential you previously created. Select your Decision environment (the default Decision Environment will work). Click **Create rulebook activation**.

### Back to ServiceNow

#### 5)
Now we will configure the Event-Driven Ansible Notification as the ServiceNow user we just assigned permissions. Navigate to the **All** menu and select **Event-Driven Ansible Notifications -> Properties**. In the Webhook Configurations section fill in a Webhook URL and a
Webhook authorization token if desired. The webhook URL should the FQDN of your Event-Driven Ansible Controller server plus the port your rulebook webhook will be listening on, for example on AAP 2.4 **http://eda.shadowman.dev:5003/endpoint** with AAP2.5 you'll navigate to the Event Stream you created at **Automation Decisions -> Event Streams**, select it, and then copy the URL that appears. For example `https://<unified_ui_URL>:443/eda-event-streams/api/eda/v1/external_event_stream/574d0d17-f0b2-4bgf-93ad-g8186a03eede/post/`. If you are using a 443 endpoint you can remove the port, so for example `https://<unified_ui_URL>/eda-event-streams/api/eda/v1/external_event_stream/574d0d17-f0b2-4bgf-93ad-g8186a03eede/post/`

No MID Server is needed if the Webhook URL is accessible directly from the running ServiceNow instance. Otherwise, fill in a proper MID Server name. Your Event-Driven Ansible Controller server listening to the webhook requests must be reachable by the MID Server. To validate the MID Server use the **All** menu and select **MID Servers -> Servers**. The selected MID Server must appear on the server list and its Status field must be Up and its Validated field must be Yes.

Click the **Test Connectivity** button to test the connection from the running servicenow instance to the configured webhook. If successful, it will display “Webhook Connection OK”.

Select what tables to monitor and when to send events to the webhook in the Tables to monitor section.

Please note that all fields in a selected table will be included in the payload of the event sent to Event-Driven Ansible via the webhook. There is no filtering of what fields are excluded.
Finally click the **OK** button to persist all the settings. These settings can be modified anytime by visiting the same properties page.

<img src="images/eda_configuration.png" alt="Event-Driven Ansible Configuration" title="Event-Driven Ansible Configuration" width="1000" />

#### 6)
To test the configuration and see the output provided by ServiceNow, you can easily create a test Incident. For this test, navigate to the **All** menu and select **Event-Driven Ansible Notifications -> Properties**. Check **When Created** for the Incident table if you haven't already. Click **OK** to confirm the changes. Create a new Incident, navigate to the **All** menu and select **Incident -> Create New**. Fill in the incident information (at a minimum you need Caller and Short description) and click Submit.

Navigate to Event-Driven Ansible Controller and select **Rule Audit** on AAP 2.4 or **Automation Decisions -> Rule Audit** on AAP 2.5. You should see a new Rule that has been triggered. Select the name. Go to **Events** and click on **ansible.eda.webhook** to see the full json payload that was received by EDA. This is what you can use to create the conditions for your rulebook in the future. You can now utilize the Event-Driven Ansible Notification Service.

<img src="images/eda_json.png" alt="Event-Driven Ansible Controller JSON" title="Event-Driven Ansible Controller JSON" width="1000" />


## ServiceNow Basic Auth Connection Configuration

#### 1) Navigate to All > Connections & Credentials > Credentials.

#### 2) Click New.
The system displays the message What type of Credentials would you like to create?

#### 3) Select Basic Auth Credentials.

#### 4) On the form, fill these values

| Parameter | Value |
|-----|-----|
| Name | `Name to uniquely identify the record. For example, enter Ansible Basic Auth Cred.` |
| User name	 | User name to log in to AAP. Ensure that the Ansible user has the System Administrator role.|
| Password | Password to log in to AAP. |
| Active | Option to actively use the credential record. |
| Order | Order to apply this credential. For example, enter 100. |

#### 5) Click Submit

#### 6) Navigate to All > Connections & Credentials > Connection & Credential Aliases

#### 7) Open the record for Ansible.

#### 8) From the Connections tab, click New.

#### 9) On the form, fill these values

| Parameter | Value |
|-----|-----|
| Name | `Name to uniquely identify the connection record. For example, enter Ansible Connection.` |
| Credential | Credential record you created for AAP. For example, select Ansible Basic Auth Cred. |
| Connection URL | URL of the AAP instance.|

#### 10) In the Advanced MID Server Configuration tab, select the MID Server as per your requirement.

#### 11) Click Submit.

## Have AAP reach out to ServiceNow

## Dependencies:

### Ansible Engine
- ansible version >= 2.9

### Collections

```bash
servicenow.itsm
```

Or if using AAP and want to attach it to a project, create a file at **collections/requirements.yml** and add

```bash
collections:

  - name: servicenow.itsm
```

This collection will be required when running >Ansible 2.10

### Create a custom credential

Creating a custom credential in AAP will allow you to pass in your ServiceNow instance, username, and password as environment variables. This means you won't need to include them in your playbook.

#### 1)
In AAP, navigate to **Credential Types** on the left side of the screen. Click the **Blue Add Button** on the top, which will present you with a New Credential Type dialog screen. Fill in the following fields:
| Parameter | Value |
|-----|-----|
| Name  |  `ServiceNow ITSM Credential`  |
| Description | Description of your credential type |

Input Configuration

```
fields:
  - id: instance
    type: string
    label: Instance
  - id: username
    type: string
    label: Username
  - id: password
    type: string
    label: Password
    secret: true
required:
  - instance
  - username
  - password
```

Injector Configuration


```
env:
  SN_HOST: '{{instance}}'
  SN_PASSWORD: '{{password}}'
  SN_USERNAME: '{{username}}'
```

<img src="images/itsmcredtype.png" alt="AAP Credential Type ITSM" title="AAP Credential Type ITSM" width="1000" />

#### 2) Create your ServiceNow Credential

| Parameter | Value |
|-----|-----|
| Name | `ServiceNow ITSM Credential` |
| Description | Description of your credential |
| Organization |  `Default` |
| Credential Type | `ServiceNow ITSM Credential` |
| Instance | `<snow_instance_id> including .service-now.com` |
| Username | `SNOW Username` |
| Password | `SNOW Password` |

<img src="images/itsmcred.png" alt="AAP Credential ITSM" title="AAP Credential ITSM" width="1000" />

#### 3) Create a Job Template

Create a playbook in your repo utilzing your new Ansible collection referencing the apppropriate modules. When creating your job template, ensure you select the ServiceNow credential that you created.

Congratulations! You can now have AAP reach out to SNOW to query and update records!

# Example playbooks

## Creating an incident in SNOW with custom fields that uses stored facts in Ansible to populate IP address, OS, and VM name
```
- name: Create an incident in ServiceNow
  hosts: "{{ vm_name }}"
  gather_facts: yes
  connection: local

  tasks:

  - name: Create an incident in ServiceNow
    servicenow.itsm.incident:
      state: new
      description: "{{ sn_description | default(omit) }}"
      short_description: "{{ incident_description }}"
      caller: admin
      urgency: "{{ sn_urgency }}"
      impact: "{{ sn_impact }}"
      other:
        u_operating_system: "{{ os | default(omit) }}"
        u_ip_address: "{{ ip_addr | default(omit) }}"
        u_vm_name: "{{ inventory_hostname | default(omit) }}"
    register: new_incident
    delegate_to: localhost

  - debug:
      var: new_incident.record.number
```

## Update a ServiceNow catalog request using the ID passed from SNOW
```
- name: Update a catalog item in ServiceNow
  hosts: localhost
  gather_facts: no

  tasks:

  - name: Retrieve catalog request sysid
    servicenow.itsm.api_info:
      resource: sc_request
      sysparm_query: numberSTARTSWITH{{ ticket_number }}
    register: requestresult
    when: ticket_number != ''

  - name: Update a catalog work notes and state in ServiceNow
    servicenow.itsm.api:
      action: patch
      resource: sc_request
      sys_id: "{{ requestresult.record[0].sys_id }}"
      data:
        request_state: "{{ request_state | default(omit) }}"
        work_notes: "{{ work_notes }}"
    when: ticket_number != ''
```

## Have AAP use ServiceNow as an inventory source

## Dependencies:

### Ansible Engine
- ansible version >= 2.9

### Collections

```bash
servicenow.itsm
```

### Create a custom credential if not already done

Creating a custom credential in AAP will allow you to pass in your ServiceNow instance, username, and password as environment variables. This means you won't need to include them in your playbook.

#### 1)
In AAP, navigate to **Credential Types** on the left side of the screen. Click the **Blue Add Button** on the right, which will present you with a New Credential Type dialog screen. Fill in the following fields:
| Parameter | Value |
|-----|-----|
| Name  |  `ServiceNow ITSM Credential`  |
| Description | Description of your credential type |

Input Configuration

```
fields:
  - id: instance
    type: string
    label: Instance
  - id: username
    type: string
    label: Username
  - id: password
    type: string
    label: Password
    secret: true
required:
  - instance
  - username
  - password
```

Injector Configuration


```
env:
  SN_HOST: '{{instance}}'
  SN_PASSWORD: '{{password}}'
  SN_USERNAME: '{{username}}'
```

<img src="images/itsmcredtype.png" alt="AAP Credential Type" title="AAP Credential Type" width="1000" />

#### 2) Create your ServiceNow Credential

In AAP, navigate to **Credentials** on the left side of the screen. Click the **Blue Add Button** on the right, which will present you with a New Credential dialog screen. Fill in the following fields:


| Parameter | Value |
|-----|-----|
| Name | `ServiceNow ITSM Credential` |
| Description | Description of your credential |
| Organization |  `Default` |
| Credential Type | `ServiceNow ITSM Credential` |
| Instance | `<snow_instance_id> from URL including .service-now.com` |
| Username | `SNOW Username` |
| Password | `SNOW Password` |

<img src="images/itsmcred.png" alt="AAP Credential" title="AAP Credential" width="1000" />

#### 3) Inventory yml file

Create a playbook in your repo utilzing your new Ansible collection. Note, the instance field does require the full domain name at this time. Name the file now.yml or now.yaml. You can adjust the selection order, fields, compose, and keyed_groups per the documenation https://github.com/ansible-collections/servicenow.itsm/blob/main/docs/servicenow.itsm.now_inventory.rst
```
# now.yml
plugin: servicenow.itsm.now
inventory_hostname_source: host_name
columns:
  - host_name
  - name
  - fqdn
  - ip_address
  - os
  - classification
  - sys_id
compose:
  ip_addr: ip_address
  ansible_host: host_name
keyed_groups:
  - key: os
    separator: ""
  - key: classification
    separator: ""
```

#### 4) Create your ServiceNow Inventory

In AAP, navigate to **Inventories** on the left side of the screen. Click the **Blue Add Button** on the right, which will present you with a New Inventory dialog screen. Fill in the following fields:


| Parameter | Value |
|-----|-----|
| Name | `ServiceNow Inventory` |
| Description | Description of your Inventory |
| Organization |  `Default` |

<img src="images/inventory.png" alt="AAP Inventory" title="AAP Inventory" width="1000" />

#### 5) Create your ServiceNow Source

In AAP, after you've saved, click to **Sources** on the top of the screen. Click the **Blue Add Button** on the right, which will present you with a New Inventory dialog screen. Fill in the following fields:


| Parameter | Value |
|-----|-----|
| Name | `ServiceNow Inventory` |
| Description | Description of your Inventory |
| Source |  `Sourced from a Project` |
| Credential |  `ServiceNow ITSM Credential` |
| Project |  `Project that contains your now.yml playbook` |
| Inventory File |  `Type the exact location of the file in the repo i.e. inventories/now.yml` |

<img src="images/inventorysource.png" alt="AAP Inventory Source" title="AAP Inventory Source" width="1000" />

Save and then sync the inventory. You now have a dynamic inventory from ServiceNow

<!-- # Table Of Contents
- [Software Requirements](#requirements)
- [Variables](#variables)
  * [default-vars.yml](#default-variables)
  * [linux_users.yml](#linux-users)
- [Credentials](#credentials)
  * [gmail_creds.yml](#gmail-credentials)
  * [redhat-activation-key.yml](#redhat-activation-key)
  * [snow_creds.yml](#servicenow-credentials)
  * [tower_creds.yml](#tower-credentials)
  * [vault_creds.yml](#hashicorp-vault-credentials)

## Requirements -->
