


# Your First S/4 Cloud Extension
This guide will help you through the following process:  
  1. Creating your first **Simple S/4 extension**
  2. Consuming OData from the **Business Partner** S/4 Cloud service - via a Fiori application
  3. Testing and editing your application
  4. Running your application both locally and remotely
  
## Overview
<!-- TOC depthFrom:2 depthTo:2 -->
- [Create New S4-Fiori Extension Project](#create-new-s4-fiori-extension-project)
- [Bind Application to S/4 OData Service](#bind-application-to-s4-odata-service)
- [Customize the view.xml Code](#customize-the-view.xml-code)
- [Run Application with Local Approuter](#run-application-with-local-approuter)
- [Test Application with Preview](#test-application-with-preview)
- [Add Additional UI Module](#add-additional-ui-module)
- [Build Application](#build-application)
- [Deploy Application to Cloud Foundry](#deploy-application-to-cloud-foundry)
<!-- /TOC -->

## Create New S4-Fiori Extension Project
*This step will guide you through generating an S4-Fiori Extension project using a Yeoman Generator.*

### Create a Dev Space in Stable
   1. Open the Stable WING landscape  
[https://webide-stable.cfapps.sap.hana.ondemand.com/index.html?extensions=true](https://webide-stable.cfapps.sap.hana.ondemand.com/index.html?extensions=true)
   2. Select 'Create Dev Space'
   3. Name your Dev Space and select the **SAP Fiori** extension pack 
   4. Select your Dev Space to start developing your application
   5. Select 'Open Workspace' and select the projects folder

### Develop an S4 Fiori Extension Project Using YeoMan
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
 
![](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/important.png?raw=true"|=150x150)| This process can be replaced by cloning the already generated repository via the terminal |
---|------------------------------------------------------------------------
```sh
 > git clone https://github.wdf.sap.corp/i057517/mta_bp.git
```
Upon completion, you will get the following folder structure

File / Folder | Purpose
---------|----------
`bp_app/` | Contains your Fiori web application 
`mta_bp_appRouter/` | Contains the app-router serving the Fiori app
`mta_bp_ui_deployer/` | Contains the application that deployes the UIs to the HTML5 repository (not used locally)
`mta.yaml` | Defines your application resources and dependencies
`xs-security.json` | Defines your application security scopes and roles

## Bind Application to S/4 OData Service   
*This step will guide you through the process of binding your application to OData provided by an external service.*   

### Prerequisites
   - The "**S4Cloud_Business_Partner**" destination has been configured in you CF sub-account  
   (generally, this would have been done by your sub-account administrator on the sub-account level).   
   *If you still don't have such destination or an equivalent one, ask your sub-account administrator to create one before continuing with this tutorial.*

### Bind Application to S/4 HANA OData Cloud Service 
   1. From the command pallet (F1) run the "**Add OData Service**" command
  ![Alt text](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/add_odata_service.png?raw=true "Bind Services") 
   2. Select the UI module that you would like to bind to the OData service (**"Select an HTML5 module"**)
   3. Select the destination you would like to use ("**Select S4Cloud_Business_Partner**")
   4. Verify the following files were updated:  
        - manifest.json file:
          - "**dataSources**" was added to the 'sap.app' section including the uri to the selected destination
          - A default "**models**" parameter was added to the model section  
        - xs-app.json file:
          - An additional route has been added with the selected destination route (as the first route entry in the "**routes**" section).
  
## Customize the view.xml Code
*This step will guide you through the process of editing your application using the 'Layout Editor'.
We will add a list that will display data pulled from the OData service and bound to the _"A_BusinessPartner"_ entity set.*  

   1. Navigate to the "view/View1.view.xml" file
   2. Open the XML code with the 'Layout Editor', Right click on the view file "_Open With --> Layout Editor_"
   3. Add a list using the editor
   4. Bind the added list to the _"A_BusinessPartner"_ entity set

## Code completion
*This step will guide you through the process of using the code completion tools*

  1. Create an .eslintrc file in the UI module folder (e.g.: bp_app), paste this content and save the file:
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
	  /*
	     Strange hack to ensure ts-server will watch changes in the
	     `node_modules/@openui5/ts-types` dir when running in VSCode.
	     See: https://github.com/microsoft/TypeScript/issues/32285.
	  */
	  /// <reference types="@openui5/ts-types" /> #
	  ```

4. In the Terminal, run cd mta_bp/bp_app/ && npm i
5. Open mta_bp/bp_app/webapp/Component.js file and click on "sap/ui/core/UIComponent".
6. There is a lamp under sap.ui.define, click on it and choose "Fix this @sap/ui5-jsdocs/no-jsdoc problem". 

   ![Alt text](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/coco_lamp.png?raw=true "Code completion") 
   
   As a result following jsdoc is added:
	```
	/**
	* @param {typeof sap.ui.core.UIComponent} UIComponent
	* @param {typeof sap.ui.Device} Device
	*/
	```
  7. Save the file.
  8. In init function add following code. You are expected to get a list of suggestions for **component1**:  
    ![Alt text](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/coco_suggestions.png?raw=true "Code completion") 

## Run Application with Local Approuter
This step will guide you through the process of running you application localy using the approuter and localy bound services.

![](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/china.png?raw=true")|For China testing, you should bind a 'Destination' service instance and an 'Xsuaa' service to your UI application. See [Bind services](#bind-services)|
---|--- 

  1. Open the "**MTA Explorer**" view and select the UI module that you would like to run. 
  2. Select the 'run' inline command next to the UI module you would like to run (green triangle)

Your module doesn't require additional services or you have already bound your services | Your module requires additional services
---|---
If you are prompted to bind services select **"Proceed without defining"**. You will then be prompted with a request to run an 'NPM install' on your approuter module | You will be prompted to bind your required services prior to running your application. See [Bind services](#bind-services).  

The **"Debug"** view will open and under the "THREADS" section you will see a running node application. In the "Debug Console" you will be able to find the run application Url.   
If you performed the run from the "MTA Explorer", the application should open automatically in a new browser tab. 
    
In the new browser tab, you should be able to see your running application.   
If your application was bound to an 'Xsuaa' service, you should get a login screen for the authentication.  
After being authenticated you should be able to see the bound backend data from the S4 destination service (S4Cloud_Business_Partner).  

## Bind services  
This step enables you to bind a local service to your application.  

  1. Login to CF via "**View --> Find Command ...**" option (F1). Enter "cf login" and select the "**CF: Login to Cloud Foundary**" command - and fill in your credentials and space details
  2. In the "**MTA Explorer**" right click the desired UI module and select "**Bind Services to a Locally run MTA Module**"    
 ![Alt text](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/bind_service.png?raw=true "Bind Services") 
  3. Select the service you would like to bind to your application (for instance: uaa_mta_simple). 
![Alt text](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/select_service.png?raw=true "Select Service from MTA to bind") 
 4. Bind the selected service to a real instance from your Cloud Foundry account  
  ![Alt text](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/select_instance.png?raw=true "Select CF Service Insctance")


![](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/china.png?raw=true")|For China testing, select the "s4_cloud_uaa" 'Xsuaa' service, repeat steps 2-4 and select the "s4_cloud_dest" 'Destination' service|
---|--- 
    
Once the binding process is finished you should have a ".env" file in your UI module folder (e.g.: bp_app) that contains the VCAP services for the local binding.

![](https://github.com/i336151/S4Fiori/blob/master/Docs/Images/important.png?raw=true")| The 'Xsuaa' service that you have selected should contain a '**tenant-mode**' parameter that is set to '**dedicated**. It also must have the Stable landscape whitelisted as explained in the following link: [https://wiki.wdf.sap.corp/wiki/display/webapptoolkit/WING+-+Update+Xsuaa+service+instance](https://wiki.wdf.sap.corp/wiki/display/webapptoolkit/WING+-+Update+Xsuaa+service+instance)|
---|--- 
  5. Return to [Run Application with Local Approuter](#run-application-with-local-approuter) 
       
## Test Application with Preview
Local preview allows you to see your UI's without running your application.
  1. Create a _localService_ folder under the _webapp_ directory in your UI module with the following content 
     - create the required files and use the _"copy"_ & _"paste"_ option):
        * [metadata.xml](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/metadata.xml)
	   (In case you used "Add Odata Service" option, this file was downloaded for you)
         * [mockserver.js](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/mockserver.js)
     
  2. Add the following files to the _test_ folder under the _webapp_ directory with following content (create the required files and use _"copy"_ & _"paste"_ option):
     * [initMockServer.js](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/initMockServer.js)
     * [mockServer.html](https://github.wdf.sap.corp/devx-wing/wing-tutorials/blob/master/Tutorials/S4_Fiori_Extension_Data/mockServer.html)

  3. Open manifest.json and validate that mainService.uri has a leading slash: "uri": "/S4Cloud_Business_Partner/"
  4. Navigate to the _"test/mockServer.html"_ file and click on Preview (Menu _Open With --> Preview_)

## Add Additional UI module
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

Upon completion, a new UI module is created under the selected MTA project. The mta.yaml file was updated with a new UI module (under the module section) and the new UI module was added to the UI deployer module (e.g.: **mta_bp_ui_deployer**) as a new required module under the requires section.  

   
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
Make sure you are logged into a CF org/space: 
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

