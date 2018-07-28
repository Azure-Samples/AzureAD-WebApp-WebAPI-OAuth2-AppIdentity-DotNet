---
services: active-directory
platforms: dotnet
author: jmprieur
---

# Calling a web api using an application identity

In contrast to the WebApp-WebAPI-OpenIDConnect-DotNet sample, this sample shows how to build an MVC web application that uses Azure AD for sign-in using OpenID Connect, and then calls a web API under the application's identity (instead of the user's identity) using tokens obtained via OAuth 2.0. This sample uses the OpenID Connect ASP.Net OWIN middleware and ADAL .Net.

This sample is an example of the trusted sub-system model, where the web API trusts the web application to have authenticated the user, and receives no direct evidence from Azure AD that the user was authenticated.  If you want to build a web app that calls a web API using a delegated user identity, please check out the sample named WebApp-WebAPI-OpenIDConnect-DotNet.

For more information about how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](http://go.microsoft.com/fwlink/?LinkId=394414).

> Looking for previous versions of this code sample? Check out the tags on the [releases](../../releases) GitHub page.

## How To Run This Sample

To run this sample you will need:
-  [Visual Studio 2017](https://aka.ms/vsdownload)
- An Internet connection
- An Azure Active Directory (Azure AD) tenant. For more information on how to get an Azure AD tenant, please see [How to get an Azure AD tenant](https://azure.microsoft.com/en-us/documentation/articles/active-directory-howto-tenant/) 
- A user account in your Azure AD tenant. This sample will not work with a Microsoft account, so if you signed in to the Azure portal with a Microsoft account and have never created a user account in your directory before, you need to do that now.

### Step 1:  Clone or download this repository

From your shell or command line:

`git clone https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity.git`

### Step 2:  Register the sample with your Azure Active Directory tenant

There are two projects in this sample. Each needs to be separately registered in your Azure AD tenant. To register these projects, you can:

- either follow the steps in the paragraphs below ([Step 2](#step-2--register-the-sample-with-your-azure-active-directory-tenant) and [Step 3](#step-3--configure-the-sample-to-use-your-azure-ad-tenant))
- or use PowerShell scripts that:
  - **automatically** create for you the Azure AD applications and related objects (passwords, permissions, dependencies)
  - modify the Visual Studio projects' configuration files.

If you want to use this automation, read the instructions in [App Creation Scripts](./AppCreationScripts/AppCreationScripts.md)


#### Register the TodoListService web API

1. Sign in to the [Azure portal](https://portal.azure.com).
2. On the top bar, click on your account and under the **Directory** list, choose the Active Directory tenant where you wish to register your application.
3. Click on **More Services** in the left hand nav, and choose **Azure Active Directory**.
4. Click on **App registrations** and choose **Add**.
5. Enter a friendly name for the application, for example 'TodoListService-AppIdentity' and select 'Web Application and/or Web API' as the Application Type. For the sign-on URL, enter the base URL for the sample, which is by default `https://localhost:44321`. Click on **Create** to create the application.
6. While still in the Azure portal, choose your application, click on **Settings** and choose **Properties**.
7. Find the Application ID value and copy it to the clipboard.
8. For the App ID URI, enter https://\<your_tenant_name\>/TodoListService-AppIdentity, replacing \<your_tenant_name\> with the name of your Azure AD tenant. 

#### Register the TodoListWebApp web app

1. Sign in to the [Azure portal](https://portal.azure.com).
2. On the top bar, click on your account and under the **Directory** list, choose the Active Directory tenant where you wish to register your application.
3. Click on **More Services** in the left hand nav, and choose **Azure Active Directory**.
4. Click on **App registrations** and choose **Add**.
5. Enter a friendly name for the application, for example 'TodoListWebApp-AppIdentity' and select 'Web Application and/or Web API' as the Application Type. For the sign-on URL, enter the base URL for the sample, which is by default `https://localhost:44322`. Click on **Create** to create the application.
6. While still in the Azure portal, choose your application, click on **Settings** and choose **Properties**.
7. Find the Application ID value and copy it to the clipboard.
8. From the Settings menu, choose **Keys** and add a key - select a key duration of either 1 year or 2 years. When you save this page, the key value will be displayed, copy and save the value in a safe location - you will need this key later to configure the project in Visual Studio - this key value will not be displayed again, nor retrievable by any other means, so please record it as soon as it is visible from the Azure Portal.
9. Configure Permissions for your application - in the Settings menu, choose the 'Required permissions' section, click on **Add**, then **Select an API**, and type 'TodoListService' in the textbox. Then, click on  **Select Permissions** and select 'Access TodoListService-AppIdentity'.

### Step 3:  Configure the sample to use your Azure AD tenant

#### Configure the TodoListService project

1. Open the solution in Visual Studio 2013.
2. Open the `web.config` file.
3. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
4. Find the app key `ida:Audience` and replace the value with the App ID URI you registered earlier, for example `https://<your_tenant_name>/TodoListService-AppIdentity`.
5. Find the app key `ida:ClientId` and replace the value with the Client ID for the TodoListService from the Azure portal.

#### Configure the TodoListWebApp project

1. Open the solution in Visual Studio 2013.
2. Open the `web.config` file.
3. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
4. Find the app key `ida:ClientId` and replace the value with the Application ID for the TodoListWebApp-AppIdentity from the Azure portal.
5. Find the app key `ida:AppKey` and replace the value with the key for the TodoListWebApp-AppIdentity from the Azure portal.
6. If you changed the base URL of the TodoListWebApp sample, find the app key `ida:PostLogoutRedirectUri` and replace the value with the new base URL of the sample.
7. Find the app key `todo:TodoListBaseAdress` ane make sure it has the correct value for the address of the TodoListService project.
8. Find the app key `todo:TodoListResourceId` and replace the value with the App ID URI registered for the TodoListService, for example `https://<your_tenant_name>/TodoListService-AppIdentity`.

#### Configure the TodoListWebApp as a trusted caller to the TodoListService

1. Go back to the TodoListService project and open the `web.config` file.
2. Find the app key `todo:TrustedCallerClientId` and replace the value with the Application ID of the TodoListWebApp.  This is used to tell the service to trust the web app.

### Step 4:  Trust the IIS Express SSL certificate

Since the web API is SSL protected, the client of the API (the web app) will refuse the SSL connection to the web API unless it trusts the API's SSL certificate.  Use the following steps in Windows Powershell to trust the IIS Express SSL certificate.  You only need to do this once.  If you fail to do this step, calls to the TodoListService will always throw an unhandled exception where the inner exception message is:

"The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."

To configure your computer to trust the IIS Express SSL certificate, begin by opening a Windows Powershell command window as Administrator.

Query your personal certificate store to find the thumbprint of the certificate for `CN=localhost`:

```
PS C:\windows\system32> dir Cert:\LocalMachine\My


    Directory: Microsoft.PowerShell.Security\Certificate::LocalMachine\My


Thumbprint                                Subject
----------                                -------
C24798908DA71693C1053F42A462327543B38042  CN=localhost
```

Next, add the certificate to the Trusted Root store:

```
PS C:\windows\system32> $cert = (get-item cert:\LocalMachine\My\C24798908DA71693C1053F42A462327543B38042)
PS C:\windows\system32> $store = (get-item cert:\Localmachine\Root)
PS C:\windows\system32> $store.Open("ReadWrite")
PS C:\windows\system32> $store.Add($cert)
PS C:\windows\system32> $store.Close()
```

You can verify the certificate is in the Trusted Root store by running this command:

`PS C:\windows\system32> dir Cert:\LocalMachine\Root`

### Step 5:  Run the sample

Clean the solution, rebuild the solution, and run it.  You might want to go into the solution properties and set both projects as startup projects, with the service project starting first.

Explore the sample by signing in, clicking the To Do List link, adding items to the To Do list, signing out, and starting again.

## How To Deploy This Sample to Azure

Coming soon.

## About The Code

Coming soon.

## How To Recreate This Sample

First, in Visual Studio 2017 create an empty solution to host the  projects.  Then, follow these steps to create each project.

### Creating the TodoListService Project

1. In Visual Studio 2017, create a new `Visual C#` `ASP.NET Web Application (.NET Framework)` project  named `TodoListService`. In the next screen, choose the `Web Api` project template. While on this screen, click the **Change Authentication** button, select **Work or School Accounts**, choose **Cloud - Single Organization** in the first dropdown, and enter the name of your Azure AD tenant in the **Domain:** text box. You will be prompted to sign-in to your Azure AD tenant.  NOTE:  You must sign-in with a user that is in the tenant; you cannot, during this step, sign-in with a Microsoft account.
1. Set **SSL Enabled** to be True.  Note the SSL URL.
1. In the project properties, Web properties, set the Project Url to be the SSL URL.
1. In the `Models` folder add a new class called `TodoItem.cs`.  Copy the implementation of thi class from this sample into the newly created class.
1. Add a new, empty, `Web API 2 Controller - Empty` called `TodoListController` to the project.
1. Copy the implementation of the **TodoListController.cs** from this sample into the newly created controller.  Don't forget to add the `[Authorize]` attribute to the class.
1. In `TodoListController` resolving missing references as suggested by Visual Studio.
1. In `Web.config`, in `<appSettings>`, create a key `todo:TrustedCallerClientId` and set the value to the clientId (AppId) of the **TodoListWebApp** from the Azure portal.
1. Add the following ASP.Net OWIN middleware NuGets: **Microsoft.Owin.Security.ActiveDirectory**, **System.IdentityModel.Tokens.Jwt**, **Microsoft.IdentityModel.Protocols.WsFederation** and **Microsoft.Owin.Host.SystemWeb**.
1. In the `App_Start` folder, create a class `Startup.Auth.cs`.  You will need to remove `.App_Start` from the namespace name.  Replace the code for the `Startup` class with the code from the same file of the sample app.  Be sure to take the whole class definition!  The definition changes from `public class Startup` to `public partial class Startup`.
1. Right-click on the project, select Add,  select "OWIN Startup class", and name the class "Startup".  If "OWIN Startup Class" doesn't appear in the menu, instead select "Class", and in the search box enter "OWIN".  "OWIN Startup class" will appear as a selection; select it, and name the class `Startup.cs`.
1. In `Startup.cs`, replace the code for the `Startup` class with the code from the same file of the sample app.  Again, note the definition changes from `public class Startup` to `public partial class Startup`.

### Creating the TodoListWebApp Project

1. In the same solution, add a new `Visual C#` `ASP.NET Web Application (.NET Framework)` named `TodoListWebApp`. In the next screen, choose the `MVC` project template. While on this screen, leave with Authentication set to **No Authentication**.
1. Set **SSL Enabled** to be True.  Note the SSL URL.
1. In the project properties, Web properties, set the Project Url to be the SSL URL.
1. Add the following ASP.Net OWIN middleware NuGets: **System.IdentityModel.Tokens.Jwt**, **Microsoft.Owin.Security.OpenIdConnect**, **Microsoft.Owin.Security.Cookies** and **Microsoft.Owin.Host.SystemWeb**.
1. Add the Active Directory Authentication Library NuGet (`Microsoft.IdentityModel.Clients.ActiveDirectory`).
1. Add a reference for assembly **System.Security**.
1. In the `App_Start` folder, create a class `Startup.Auth.cs`.  You will need to remove `.App_Start` from the namespace name.  Replace the code for the `Startup` class with the code from the same file of the sample app.  Be sure to take the whole class definition!  The definition changes from `public class Startup` to `public partial class Startup`.
1. Right-click on the project, select Add,  select "OWIN Startup class", and name the class "Startup".  If "OWIN Startup Class" doesn't appear in the menu, instead select "Class", and in the search box enter "OWIN".  "OWIN Startup class" will appear as a selection; select it, and name the class `Startup.cs`.
1. In `Startup.cs`, replace the code for the `Startup` class with the code from the same file of the sample app.  Again, note the definition changes from `public class Startup` to `public partial class Startup`.
1. In the `Views` --> `Shared` folder, create a new **MVC 5 partial page (Razor)** named `_LoginPartial.cshtml`.  Replace the contents of the file with the contents of the file of same name from the sample.
1. In the `Views` --> `Shared` folder, replace the contents of `_Layout.cshtml` with the contents of the file of same name from the sample.  Effectively, all this will do is add a single line, `@Html.Partial("_LoginPartial")`, that lights up the previously added `_LoginPartial` view.
1. Create a new **MVC 5 Controller - Empty** named `AccountController`.  Replace the implementation with the contents of the file of same name from the sample.
1. If you want the user to be required to sign-in before they can see any page of the app, then in the `HomeController`, decorate the `HomeController` class with the `[Authorize]` attribute.  If you leave this out, the user will be able to see the home page of the app without having to sign-in first, and can click the sign-in link on that page to get signed in.
1. In the `Models` folder add a new class called `TodoItem.cs`.  Copy the implementation of TodoItem from this sample into the class.
1. 1. In the `Models` folder add a new class called `FileCache.cs`.  Copy the implementation of FileCache from this sample into the class.
1. Add a new **MVC 5 Controller - Empty** named `TodoListController` to the project.  Copy the implementation of the controller from the sample.  Remember to include the [Authorize] attribute on the class definition.
1. In `Views` --> `TodoList` create a new view, `Index.cshtml`, and copy the implementation from this sample.
1. In `Web.config`, in `<appSettings>`, create keys for `ida:ClientId`, `ida:AppKey`, `ida:Tenant`, `ida:AADInstance`, `ida:RedirectUri`, `ida:TodoListResourceId`, and `ida:TodoListBaseAddress`, and set the values accordingly.  For the public Azure AD, the value of `ida:AADInstance` is `https://login.microsoftonline.com/{0}`.

Finally, in the properties of the solution itself, set both projects as startup projects.
