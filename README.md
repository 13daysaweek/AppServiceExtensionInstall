# Installing Specific versions of App Service Site Extensions

## Overview
Azure App Serivce makes it possible to install site extensions, manually, via the extension gallery, or through ARM templates.  Extensions are sourced from the public nuget.org feed.  Whether installing manually from the gallery or via ARM templates, there isn't a way to explicitly specify the version of the site extension you wish to install, the most recent version will always be installed.  This can lead to situations where you test an application with one version of an extension in a lower environment, then promote to a higher environment, only to find that since you deployed to your lower environment, a new version of the extension has been released.  Here's an example of what that might look like:

## Solution
While it's not possible to explicitly specify a desired site extension version via ARM template or when installing manually from the gallery, the ARM APIs for App Service site extensions do allow for specifying which version of an extension you wish to install.  
