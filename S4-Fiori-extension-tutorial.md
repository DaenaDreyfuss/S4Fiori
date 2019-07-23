# Your First S/4 Cloud Extension

This guide will help you through the following process:  
  1. Creating your first **Simple S/4 extension**
  2. Consuming OData from the **Business Partner** S/4 Cloud service - via a Fiori application
  3. Running the application on your developer workspace

## Overview
<!-- TOC depthFrom:2 depthTo:2 -->
- [Create New S4-Fiori Extension Project](#create-new-s4-fiori-extension-project)
- [Bind Application to S/4 OData Service](#bind-application-to-s4-odata-service)
- [Customize view.xml Code](#customize-view.xml-code)
- [Run Application with Local Approuter](#run-application-with-local-approuter)
- [Test Application with Preview](#test-application-with-preview)
- [Add Additional UI Module](#add-additional-ui-module)
- [Build Application](#build-application)
- [Deploy Application to Cloud Foundry](#deploy-application-to-cloud-foundry)
<!-- /TOC -->

### Create New S4-Fiori Extension Project

#### Create a Dev Space in Stable
   1. Open the Stable WING landscape  
[https://webide-stable.cfapps.sap.hana.ondemand.com/index.html?extensions=true](https://webide-stable.cfapps.sap.hana.ondemand.com/index.html?extensions=true)
   2. Select 'Create Dev Space'
   3. Name your Dev Space and select the **SAP Fiori** extension pack 
   4. Select your Dev Space to start developing your application
   5. Select 'Open Workspace' and select the projects folder

#### Develop an S4 Fiori Extension Project Using YeoMan
  1. Open the **Terminal** via the menu bar (Terminal-->New Terminal)
  2. In the terminal navigate to the projects folder
  3. Run the **[yo](command:yo)** to start the Yeoman generator
  4. Select the **Fiori Project** Yeoman generator
        - Specify a project name? (**mta_bp**) 
        - Do you want to create a UAA Service and bind it to the approuter module? (**Yes**)
        - Do you want to create a Destination Service and bind it to the approuter module? (**Yes**)
        - What is the HTML5 module name? (**bp_app**)
        - What is the namespace? (**ns**)
        - What is the UI5 version? (**1.64**)
	- Which template do you want to use? (**SAPUI5 Application**)
        - What is the view name? (**View1**)
 
Upon completion, you will get the following folder structure

File / Folder | Purpose
---------|----------
`bp_app/` | Contains your Fiori web application 
`mta_bp_appRouter/` | Contains the app-router serving the Fiori app
`mta_bp_ui_deployer/` | Contains the application that deployes the UIs to the HTML5 repository (not used locally)
`mta.yaml` | Defines your application resources and dependencies
`xs-security.json` | Defines your application security scopes and roles

## Bind Application to S/4 OData Service   
### Prerequisites
   - The "**S4Cloud_Business_Partner**" destination has been configured in you CF sub-account  
   (generally, this would have been done by your sub-account administrator on the sub-account level).   
   **If you still don't have such destination or an equivalent one, ask your sub-account administrator to create one before continuing with this tutorial.**

### Bind Application to S/4 HANA OData Cloud Service 
   1. From the command pallet (F1) run the "**Add OData Service**" command
  ![Alt text](S4_Fiori_Extension_Data/add_odata_service.png?raw=true "Bind Services") 
   2. Select the UI module that you would like to bind to the OData service (**"Select an HTML5 module"**)
   3. Select the destination you would like to use ("**Select S4Cloud_Business_Partner**")
   4. Verify the following changes files were updated:
	- manifest.json file:
		- "**dataSources**" was added to the 'sap.app' section including the uri to the selected destination
		- A default "**models**" parameter was added to the model section  
	- xs-app.json file:
    		- An additional route has been added with the selected destination route (as the first route entry in the "**routes**" section).
  
## Customize view.xml Code
Add a list that will display data pulled from the OData service and bound to the _"A_BusinessPartner"_ entity set.  
Navigate to the "view/View1.view.xml" file, open the XML code editor, and add the following code:
~~~
<mvc:View xmlns:mvc="sap.ui.core.mvc" xmlns="sap.m" controllerName="ns.bp_app.controller.View1" displayBlock="true">
	<Shell id="shell">
		<App id="app">
			<pages>
				<Page id="page" title="{i18n>title}">
					<content>
					    <List noDataText="Drop list items here" id="list0" items="{/A_BusinessPartner}">
					        <items>
					            <StandardListItem type="Navigation" title="List Item 1" description="{FirstName}" icon="sap-icon://picture" id="item0"/>
					        </items>
					    </List>
					</content>
				</Page>
			</pages>
		</App>
	</Shell>
</mvc:View>
~~~

Alternatively, you can open the "_Open With --> Layout Editor_", add a list using the editor, and bind the added list to the _"A_BusinessPartner"_ entity set. 

**Code completion**

1. Create .eslintrc file in the UI module folder (e.g.: bp_app), paste this content and save the file:
	```
	{
	    "plugins": [
		"@sap/ui5-jsdocs"
	    ],
	    "extends": ["plugin:@sap/ui5-jsdocs/recommended", "eslint:recommended"]
	}
	```

2. Open mta_bp/bp_app/package.json, add following **devdependencies** and save the file:
	```
	  "eslint": "5.16.0",
	  "@sap/eslint-plugin-ui5-jsdocs": "2.0.1"
	```

3. Open mta_bp/bp_app/webapp/Component.js file, add below code to the begining of the file and save the file:  
	```
	/**
	* Strange hack to ensure ts-server will watch changes in the
	* `node_modules/@openui5/ts-types` dir when running in VSCode.
	* - See: https://github.com/microsoft/TypeScript/issues/32285.
	*/
	/// <reference types="@openui5/ts-types" /> #
	```

4. In the Terminal, run cd mta_bp/bp_app/ && npm i
5. Open mta_bp/bp_app/webapp/Component.js file and click on "sap/ui/core/UIComponent".
6. There is a lamp under sap.ui.define, click on it and choose "Fix this @sap/ui5-jsdocs/no-jsdoc problem". 

   ![Alt text](S4_Fiori_Extension_Data/coco_lamp.png?raw=true "Code completion") 
   
   As a result following jsdoc is added:
	```
	/**
	* @param {typeof sap.ui.core.UIComponent} UIComponent
	* @param {typeof sap.ui.Device} Device
	*/
	```
7. Save the file.
8. In init function add following code. You are expected to get a list of suggestions for **component1**:  
    ![Alt text](S4_Fiori_Extension_Data/coco_suggestions.png?raw=true "Code completion") 

## Run Application with Local Approuter
**For China testing, you should bind a destination service instance and an uaa service to your UI application. See [Bind services](#bind-services)**
1. Open the "**MTA Explorer**" view and select the UI module that you would like to run. 
2. If your UI module **don't require** any resource from the mta.yaml (like: uaa, destination), click on "**Run MTA Module**". This will run an 'npm install' of your approuter module (if the node modules can't be found in the approuter module). 
**Otherwise**, do the service binding before you run the application. See [Bind services](#bind-services).  

## Bind services  
If you would like to run your application bound to a UAA or destination service (a.k.a local binding), follow these instructions:  
1. Login to CF via "**View --> Find Command ...**" option (F1). Enter "cf login" and select the "**CF: Login to Cloud Foundary**" option. Provide details according to instructions.   
**For China testing, do the CF login via terminal:**   
      ~~~
      cf login -a https://api.cf.canaryac.vlab-sapcloudplatformdev.cn
      ~~~
2. In the "**MTA Explorer**" select the desired UI module, right click on it and select "**Bind Services to a Locally run MTA Module**" option.    
 ![Alt text](S4_Fiori_Extension_Data/bind_service.png?raw=true "Bind Services") 
  
3. Select an MTA service that you would like to bind to your application (for instance: uaa_mta_simple). 
 
    ![Alt text](S4_Fiori_Extension_Data/select_service.png?raw=true "Select Service from MTA to bind") 
 
 4. Bind the selected service to a real instance from the CF account that you logged in  
    ![Alt text](S4_Fiori_Extension_Data/select_instance.png?raw=true "Select CF Service Insctance")

    **For China testing, select the "s4_cloud_uaa" uaa service**  
    
    **Note:**  
        Do the same local binding steps 2-4, for every service from the mta.yaml that relevant for your run flow scenario. **For China testing, select the "s4_cloud_dest" destination service**
    
    Once the binding process is finished you should have an ".env" file in your UI module folder (e.g.: bp_app) with a VCAP service for the local binding.
        
    **Note: (For China testing, you can skip this part and navigate to bullet #5)** 
        
       - The service that you have selected should contain a '**tenant-mode**' parameter that is set to '**dedicated**'.
       - If the selected **UAA** service does not yet exposed in the **Stable** landscape as a whitesource, you must do so manually:  
            1. Navigate to the **xs-security.json** file and update it with following data:  
                - Update the "**xsappname**" parameter with the equivalent value found in the .env file (located under your UI module).
                - Add an additional parameter **oauth2-configuration** (after "role-templates") as seen below (if you are not working on the "Stable" landscape, update the respective redirect-uris with your landscape host):
                    ```sh
                    "oauth2-configuration": {
                          "redirect-uris": [
                          "https://*.stable-aws-01.dev1.sapwebide.net.sap/**"
                          ]
                      }
                    ```    
            2. In order to update the selected **UAA** service with this new white source list, run the following command in your terminal:
                ```sh
                cf update-service <service instance name> -c <xs-security.json file path>
                e.g.: cf update-service uaa2 -c mta_bp/xs-security.json
            ```

5. In the "**MTA Explorer**" perform the "**Run MTA Module**". This will run your application. You will promote several questions that need in order to run the application:
     - "Module <your module name> is missing one or more required services bindings" - since we already did the local binding manually, you can click on **"Proceed without defining"**. 
     - "Seems like you need to run 'npm install" - click on **"Run npm install now"**.
     
     The **"Debug"** view will open and under the "THREADS" section you will see a running node application. In the "Debug Console" you will find the run application Url. If you performed the run from the "MTA Explorer", the application should be opened in a new browser tab directly. If you performed the run from the **"Debug"** view tab, you should copy the application Url and launch it in a new browser tab.  
    See example:  
    ![Alt text](S4_Fiori_Extension_Data/mta_explorer.png?raw=true "Run MTA Module")
    
    In the new browser tab, you should be able to see your running application.   
    If your application was bound to a UAA service, you should get a login in screen for the authentication.  
    Once authenticated, you should be able to see the bound backend data from the S4 destination service (S4Cloud_Business_Partner).  

## Test Application with Preview
If you would like to see your UI's without running your application, you can test your application using a local preview. This is  useful when developing the UI aspects of your application. 

For this follow the following steps: 

1. Create a _localService_ folder under the _webapp_ directory in your UI module with the following content 
    - create the required files and use the _"copy"_ & _"paste"_ option):
         * [metadata.xml](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/metadata.xml)
	   (In case you used "Add Odata Service" option, this file was downloaded for you)
         * [mockserver.js](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/mockserver.js)
     
    - Alternatively, you can open a terminal, navigate to ~/projects and enter the following commands:
        ```bash
        cd s4CloudExtensionExampleProject/bp_app/webapp/localService/
        wget https://github.wdf.sap.corp/raw/devx-wing/wing-tutorials/master/Tutorials/S4_Fiori_Extension_Data/metadata.xml
        wget https://github.wdf.sap.corp/raw/devx-wing/wing-tutorials/master/Tutorials/S4_Fiori_Extension_Data/mockserver.js
        ```

2. Add the following files to the _test_ folder under the _webapp_ directory with following content (create the required files and use _"copy"_ & _"paste"_ option):
     * [initMockServer.js](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/initMockServer.js)
     * [mockServer.html](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/mockServer.html)

3. Open manifest.json and validate that mainService.uri has a leading slash: "uri": "/S4Cloud_Business_Partner/"
4. Navigate to the _"test/mockServer.html"_ file and click on Preview (Menu _Open With --> Preview_)


[Run application with local approuter](#run-application-with-local-approuter)

## Add Additional UI module
To create a new UI module you will need to follow following steps:
1. Open the **Terminal** via the menu bar (Terminal-->New Terminal)  
2. In the terminal navigate to the mta project in which you would to create a new UI module
3. Run the **[yo](command:yo)** to start the Yeoman generator
    ```sh
    > cd ~/projects/mta_bp
    > yo
    ```
4. Select the **Fiori Module** Yeoman generator
    - Specify a path to the folder containing the "mta.yaml" file of the MTA project that you would like to update: (/home/user/projects/mta_bp) 
    - Specify a module name: (HTML5Module)
    - What is the namespace? (ns)
    - Which template do you want to use? (SAPUI5 Application)
    - What is the UI5 version? (1.64)
    - What is the view name? (View1)

Upon completion, a new UI module is created under the selected mta project. The mta.yaml file was updated with a new UI module (under the module section) and the new UI module was added to the UI deployer module (e.g.: **mta_bp_ui_deployer**) as a new required module under the requires section.  

   
## Build Application
Navigate to the project root folder
  ```
cd mta_bp
  ```
Init the mbt tool in order to generate the Makefile.mta file
```
mbt build -p=cf
```  

## Deploy Application to Cloud Foundry
Make sure you are logged in to a CF org/space: 
 ```
cf login -a <api-endpoint> -o <org> -s <space>
cd mta_archives 
cf deploy <your_created_mtar_file>.mtar
  ```
Once the deployment process is finished, run the deployed application via the approuter link.

To get running applications in the CF, execute the following command: 
 ```
cf a
  ```
Create the runnable url:
1. Find your approuter application in the list (make sure its status is **"started"**). 
2. Copy the respective url link as appears in the approuter application. 
3. Locate the 'sap.app.id' (from the manifest.json file located in your HTML5 module webapp folder) and add it to the copied link (any "." in the id should be removed)
4. Append the application executable html file to the copied link. 
5. Open a new web tab and paste the composed link. See example:  
    `https://devx2-tokyo-approuter.cfapps.sap.hana.ondemand.com/nsbp_app/index.html`

