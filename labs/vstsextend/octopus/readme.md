---
title: Automating Deployments with VSTS and Octopus Deploy
layout: page
sidebar: vsts2
permalink: /labs/vstsextend/octopus/
folder: /labs/vstsextend/octopus/
---
Last updated : {{ "now" | date: "%b %d,%Y" }}

## Overview

Last updated : {{ "now" | date: "%b %d, %Y" }}.

[Octopus Deploy](https://Octopus.com) is an automated deployment server that makes it easy to automate deployment of ASP.NET web applications, Java applications, NodeJS application and custom scripts to multiple environments.

Azure DevOps includes a first-class, powerful release management capability that simplifies deployment of any application to any platform. But teams who prefer or already have chosen Octopus deploy, can use the **[Octopus Deploy Integration](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks)** extension available on **Extensions for Azure DevOps** that provides Build and Release tasks to integrate Octopus Deploy with Azure DevOps and Azure DevOps Server.

This lab shows how we can integrate Azure DevOps/Azure DevOps Server Build with Octopus to automate build and deployment of an ASP.NET Core application to an Azure App Service.

## Prerequisites for the lab

1. **Microsoft Azure Account**: You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-us/free/){:target="_blank"}

    * If you are an active Visual Studio Subscriber, you are entitled for a $50-$150 credit per month. You can refer to this [link](https://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/){:target="_blank"} to find out more information about this including how to activate and start using your monthly Azure credit.

    * If you are not a Visual Studio Subscriber, you can sign up for the FREE [Visual Studio Dev Essentials](https://www.visualstudio.com/dev-essentials/){:target="_blank"} program to create an **Azure free account** (includes 1 year of free services, $200 for 1st month).

1. You will need an **Azure DevOps Organization**. If you do not have one, you can sign up for free [here](https://www.visualstudio.com/products/visual-studio-team-services-vs){:target="_blank"}


## Setting up the Environment

Octopus Deploy has two components:

* **Octopus Server** - a centralized web front-end that orchestrates deployments , and
* **Tentacle** - agent that needs to be on every target endpoint.

We will spin up a Octopus server on Azure. Click the **Deploy to Azure** button below to provision a Octopus Server.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/octopus.octopusdeployoctopus-deploy){:target="_blank"}

1. Click on **Create** button.

   ![click_create](images/icrosoftAzure.png)

1. Click on **Create new** provide the **Resource group** name and click **OK**.
   ![create_RG](images/MicrosoftAzure.png)

1. Provide Octopus server **Domain Name**, VM Admin **username and password**, and SQL admin **username and password**. Note down the Domain Name (DNS) as this is used later to connect to Octopus server.

   ![server_details](images/MicrosoftAzure1.png)

1. Provide Octopus Admin **username and password**. This is used to login to Octopus server. Also provide your **name, organization name** and **email address** to activate trial license for octopus server. Click OK.

   ![octopus_details](images/MicrosoftAzure2.png)

1. Click **OK** in the Summary section.

   ![summary](images/MicrosoftAzure3.png)

1. Click **Create** in the Buy section.

   ![click_create2](images/MicrosoftAzure4.png)

1. It takes approximately 15 minutes for the deployment to complete. Once the deployment is successful, connect to Octopus server using DNS, and login with Octopus Admin username and password.

   ![Octopus_login](images/OctopusDeploy15_1.png)

1. You will see the Octopus deploy web portal.

    ![Octopus Dashboard](images/OctopusDeploy15_2.png)

## Generating Service Principal Names details from Azure

We need to generate SPN details to get **Tenant ID**, **Application ID** and **Application Password/Key** which will be used in later part of the lab.

1. Login to your Azure account and click on **Cloud Shell**.

    ![webapp](images/CloudShellMicrosoftAzure.png)

1. Select **Bash** or **Powershell** to run the command which will generate the SPN details.
 
    ![webapp](images/DashboardMicrosoftAzure2.png)

1. Create Storage account as Azure Cloud Shell requires an Azure file share to persist files, If you already have one select it or create new. Select the subscription and click on **Create storage**. 
    ![webapp](images/DashboardMicrosoftAzure3.png)

1. Once the storage account is provisioned, run the below command to get the SPN details.

    `az ad sp create-for-rbac --name (provide a short name) --password (provide a password)`
    
    ![webapp](images/DashboardMicrosoftAzure5.png)

1. Note down the **Tenant ID**, **Appliction ID** and the **Password**. 

    ![webapp](images/DashboardMicrosoftAzure6.png)


## Setting up the Deployment Target

In this lab, we will use Azure WebApp as the deployment target.

Click the **[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.WebSite)** button below to provision Azure WebApp.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.WebSite){:target="_blank"}

1. Provide **Web App name** and **Resource Group**. You can either create new Resource group or use existing one, and click **Create**

    ![webapp](images/MicrosoftAzure6.png)

## Generate Octopus API Key

API key is used to authenticate Azure DevOps with Octopus server. Using API key lets you keep your username and password secret.

1. From the Octopus Deploy portal, select **Profile** under *User* menu.

   ![User Profile](images/OctopusDeploy15_3.png)

1. Select **My API Key** and click **New API Key** to create one. We will use the API Key to connect Octopus Deploy with Azure DevOps

   ![Request New API Key](images/APIKey.png)

1. Specify a **purpose**, for e.g., **Azure DevOps Integration** and click **Generate New**.

   ![Generate New API Key](images/OctopusDeploy7.png)

1. Copy the API Key to clipboard and save this somewhere as you may use it for future requests.

   ![Generated API Key](images/OctopusDeploy8.png)

## Setting up the Azure DevOps team project

1. Use [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net/?Name=octopus&TemplateId=77370){:target="_blank"} to provision the project on our Azure DevOps Organization.

   > **Azure DevOps Demo Generator** helps you create team projects on your Azure DevOps Organization with sample content that include source code, work items, iterations, service endpoints, build and release definitions based on the template you choose during the configuration.

   ![VSTS Demo Generator](images/DemoGenerator9.png)

1. Select the Organization, provide a name for your project. Provide the Octopus URL (VM's DNS URL) that was created previously, API Key and click on **Create Project**.
Once the project is provisioned, click the URL to navigate to the project.

   ![VSTS Demo Generator](images/DevOpsDemoGenerator10.png)

   {% include note.html content= "This URL will automatically select Octopus template in the demo generator. If you want to try other projects, use this URL instead - [https://azuredevopsdemogenerator.azurewebsites.net/](https://azuredevopsdemogenerator.azurewebsites.net/){:target=\"_blank\"}" %}

## Exercise 1: Configure Deployment Target in Octopus Server

Let us create a deployment environment in Octopus server and link to Azure using Management Certificate. Environments are deployment targets consisting of machines or services used by Octopus Deploy to deploy software. With Octopus Deploy,  we  can deploy software to Windows servers, Linux servers, Microsoft Azure, or even an offline package drop.

Grouping  our deployment targets by environment lets you define your deployment processes and have Octopus deploy the right versions of our software to the right environments at the right time.

In this lab, we are using Azure App Service as the deployment target.

1. From the Octopus portal, select **Create environments** to go into the **Infrastructure** page

   ![CreateEnvironment](images/CreateEnvironment.png)

1. Once inside, click **Add Environment**.

   ![AddEnvironment](images/OctopusDeploy15_4.png)

1. Provide the environment name and click **Save**.

   ![DevEnvironment](images/OctopusDeploy15_5.png)

1. Octopus Deploy provides first-class support for deploying Azure Cloud Services and Azure Web Applications. To deploy software to Azure,   we must add  our  Azure subscription to Octopus Deploy, and then use the built-in step templates to deploy to the cloud. Once the environment is created, click on **Accounts** select **Azure Subscription** form **ADD ACCOUNT** dropdown.


   ![Add Account](images/OctopusDeploy15_6.png)

1. Octopus Deploy authenticates with Azure in one of two methods:

    * To deploy to Azure Resource Manager (ARM), Octopus requires [**Azure Service Principal Account**](https://octopus.com/docs/infrastructure/azure/creating-an-azure-account/creating-an-azure-service-principal-account){:target="_blank"}

    * [**Azure Management Certificate**](https://octopus.com/docs/infrastructure/azure/creating-an-azure-account/creating-an-azure-management-certificate-account){:target="_blank"} is used by Octopus to deploy to Cloud Services and Azure Web Apps.

    Enter the following details -

    * **Name**: Provide an account name
    * **Subscription ID**: Your [Azure Subscription ID](https://blogs.msdn.microsoft.com/mschray/2016/03/18/getting-your-azure-subscription-guid-new-portal/){:target="_blank"}
    * **Authentication Method**: Choose **Use a Service Principal**
    * **Tenant ID**, **Application ID**, **Application Password/Key**: Created earlier in the lab


   ![Create Account](images/AccountOctopusDeploy.png)

   ![Create Account](images/accountOctopusDeploy2.png)

1. Click **SAVE AND TEST** and notice that your account is verified.

   ![Download Certificate](images/accountverifyOctopusDeploy.png)

1. To add deployment target, go to **Deployment Targets**, click on **ADD DEPLOYMENT TARGET**, select **Azure Web App** and click **NEXT**.

   ![O8](images/AddDeploymentTargetsOctopusDeploy.png)

   ![O8](images/NewDeploymentTarget.png)

1. In create deployment target page provide **Display Name**, choose **Environment** from the dropdown, provide **Target Roles**, select the **Account** which was created earlier and select **Azure Web App** from the drpdown as shown below.

   ![O9](images/Createdeploymenttarget.png)

   ![O9](images/Createdeploymenttarget2.png)

## Exercise 2: Create Project in Octopus

Let us create a Project in Octopus to deploy the package to **Azure App Service**. A [**Project**](https://octopus.com/docs/deployment-process/projects){:target="_blank"} is a collection of deployment steps and configuration variables that define how your software is deployed.

1. Go to Octopus dashboard and click on **Create a project**.

   ![Project](images/Project.png)

1. Click on **ADD PROJECT**, provide the project name, description and click on **SAVE**.

   ![AddProject](images/OctopusDeploy15_11.png)

   ![PUProject](images/Projects.png)

1. Once the project is created, click **Define your deployment process**. The [deployment process](https://octopus.com/docs/deploying-applications/deployment-process){:target="_blank"} is like a recipe for deploying your software.

   ![DefineProcess](images/project2.png)

1. Click on **ADD STEP** to see a list of built-in step templates, custom step templates, and community contributed step templates.

   ![AddStep](images/Project3.png)

1. **Search** for **Azure Web App** template and click **Add**.

   ![AddWebAppStep](images/Project4.png)

1. Populate the step template with required details -

   * **Step Name** : A short, unique name for the template.
   * **On Behalf Of** : Provide on behalf of target roles created in previous exercise step **8**. 
   * **Package ID** : Type-in as Asp.netcore (if you are providing different package ID, update it in Package Application task of the build definition)

   ![PkgID](images/project5.png)

   ![Azure](images/project6.png)

1. Clicking **Save** should define the project creation and its deployment process.

## Exercise 3: Triggering CI-CD

In this exercise, we will package ASP.NET Core application and push the package to Octopus Server. We will use build tasks of **Octopus Deploy Integration** extension which was installed during Team Project provisioning.

| Tasks| Usage|
|-------| ------|
|![dotnetcore](images/dotnetcore.png) **Restore**| dotnet command-line tool restores all the package dependencies like **ASP.NET Core Identity, ASP.NET Core session** etc. required to build this project|
|![dotnetcore](images/dotnetcore.png) **Build**| We will use dotnet command-line tool to build the project and its dependencies into a set of binaries|
|![dotnetcore](images/dotnetcore.png) **Publish**| We will use this task to create a package with published content for the web deployment|
|![octopuspackage](images/octopuspackage.png) **Package Application** | We will package the PHP source code into a zip file with the version number|
|![pushpackage](images/pushpackage.png) **Push packages to Octopus**| The copied package will be pushed to Octopus server from VSTS artifacts directory|
|![createoctopus](images/createoctopus.png) **Create Octopus Release**|Automates the creation of release in Octopus server. A release captures all the project and package details to be deployed over and over in a safe and repeatable way|
|![releaseoctopus](images/releaseoctopus.png) **Deploy Octopus Release**| Automates the deployment of release in Octopus server. A deployment is the execution of the steps to deploy a release to an environment. An individual release can be deployed numerous times to different environments|

1. Go to **Builds** under **Pipelines** tab, select **Octopus** build definition and click on **Edit**.

   ![BuildDefinition](images/Builds11.png)


1. In **Push Packages to Octopus** task, update **Octopus Deploy Server** field with the created endpoint value.


   ![QBuild](images/BuildAzureDevOpsServices1.png)

1. In **Create Octopus Release** task, update **Octopus Deploy Server** field with the created endpoint value and **Project** fields.

    ![Update1](images/BuildAzureDevOpsServices2.png)

1. In **Deploy Octopus Release** task, update **Octopus Deploy Server** field with the created endpoint value, choose the appropriate values from the drop down for fields - **Project**  and **Deploy to Environments** and **Save** the build definition.

    ![Update](images/BuildAzureDevOpsServices3.png)

1. Navigate to **Repos** hub on the Azure DevOps portal.

   ![Code Hub](images/OctopusRepos1.png)

1. The repository contains an **ASP.NET Core** application source code provisioned by Azure DevOps Demo Generator.

   > The team project already has a Continuous Integration (CI) build configured that gets automatically initiated when the source code modifications are committed to the repository.

1. To edit source code, open file **Index.cshtml** by navigating to below path in the master branch and click on **Edit** :

   `Octopus/src/PartsUnlimitedWebsite/Views/Home/Index.cshtml`

   ![Source code path](images/IndexcshtmlRepos2.png)

1. Make some small changes to the code. For this example, change discount percentage of `50%` to `70%` on `line 30` and then click on **Commit** to save and commit the changes.

   ![Code Edit](images/IndexcshtmlRepos3.png)

1. Go to **Build** tab, you will see in-progress build.

    ![BuildProgress](images/BuildPipelines.png)

1. Once the build completes, go to Octopus portal project dashboard. You will see the release completion in Octopus.

    ![CD-Octopus](images/OctopusDeploy24.png)

1. Go to Azure Web App from your **[Azure Portal](https://portal.azure.com){:target="_blank"}** and click on **Browse**.

   ![Browse](images/MicrosoftAzure24.png)

1. You will see an ASP.NET Core application up and running.

   ![Changes](images/PartsUnlimited.png)
