# Installing Specific versions of App Service Site Extensions

## Overview
Azure App Serivce makes it possible to install site extensions, manually, via the extension gallery, or through ARM templates.  Extensions are sourced from the public nuget.org feed.  Whether installing manually from the gallery or via ARM templates, there isn't a way to explicitly specify the version of the site extension you wish to install, the most recent version will always be installed.  This can lead to situations where you test an application with one version of an extension in a lower environment, then promote to a higher environment, only to find that since you deployed to your lower environment, a new version of the extension has been released.  Here's an example of what that might look like:

## Solution
While it's not possible to explicitly specify a desired site extension version via ARM template or when installing manually from the gallery, the ARM API for App Service site extensions do allow for specifying which version of an extension you wish to install.  Additionally, the API can be used to specify an alternate source besides nuget.org for the installation source.

Sincce all that's needed for this solution is a REST call, specifically an HTTP PUT, any tooling capable of sending REST request will work.  However, [ARMClient.exe, available through Project Kudu](https://github.com/projectkudu/ARMClient) is a simple console application that makes calling ARM APIs easy and removes the need to know the details of how to compose these requests.  ARMClient.exe supports interactive Azure AD-based authentication, as well as Azure AD service principal authentication, making this approach suitable for using in a CI/CD pipeline.

## Installing a specific version of a Site Extension manually
Install ARMClient.exe following the [instructions in the GitHub repo](https://github.com/projectkudu/ARMClient#armclient).  Once you have the tool installed, create a JSON file with the following contents:

```
{
    "properties": {
      "version": "{version to install}",
      "feed_url": "https://www.nuget.org/api/v2/"
    }
}
```
Replace `{version to install}` with the version of the extension you wish to install.  Note the `feed_url` value is for the default, public nuget.org feed.  If you use a different feed, replace the nuget.org URL with the URL for your feed.

Once you've created the JSON file, you need ARMClient to fetch an access token.  Run the following command:

```
armclient login
```

You'll be promoted to login with your Azure AD credentials.  Once logged in, run the following command:

```
armclient put /subscriptions/{subscription}/resourceGroups/{resourceGroup}/providers/Microsoft.Web/sites/{siteName}/siteextensions/{siteExtensionName}?api-version={apiVersion} @{jsonFileName}.json

```
Replace the following tokens:
* {subscription} with the Subscription ID for the sub that has your App Service resaource
* {resourceGroup} with the name of the Resource Group that contains your App Service resource
* {siteName} with the name of your App Service resource
* {siteExtensionName} with the name of the Site Extension you wish to install
* {apiVersion} with the [version of the Site Extensions ARM API you wish to use](https://docs.microsoft.com/en-us/azure/templates/microsoft.web/sites/siteextensions?tabs=json).  (Latest is 2021-02-01 when this README was writen)
* {jsonFileName} with the name of the JSON file you created abvove.  Note that you must include the `@` in front of the file name, this tells ARMClient.exe to source the JSON from a file rather than `stdin`.



After the command completes, the specified version of your extension should be installed on your App Service resource.

## Installing a specific version of a Site Extension via Azure Pipelines
The included [install-extension.yaml](/.pipelines/install-extension.yaml) file shows a simple Azure Pipeline that automates the process of installing a specified version of a Site Extension.  The pipeline runs the same `armclient put` command shown above, however it uses a service principal for authentication, rather than interactive auth.  Note that the service principal will need to have the appropriate permissions to update your App Serivce resource.  

One other consideration when needing to run ARMClient.exe in a pipeline is that hosted Azure DevOps agents and GitHub Actions workers do not have ARMClient.exe installed.  Per the documentation in the ARMClient.exe repo, ARMClient.exe can be installed with [Cholately](https://chocolatey.org/), which is suitable for automation and installed by default on hosted Azure DevOps agents and GitHub Actions workers.

For service principal authentication, the pipeline retrieves credentials from an Azure Key Vault, linked to an Azure DevOps variable group.

The pipeline also uses the File Transform task which allows for changing the version number of the feed URL in the payload.json file within the pipeline, sourcing values for the transform from variables, in this case variables within a Variable Group.