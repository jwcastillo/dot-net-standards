Configuration Management
===========================================

Configuration concerns addressed here are scoped to only managing and
maintaining credential sets and other secret information, and usage of that
secret information within source control and .NET projects. We do NOT address
delivery of configuration settings to different environments in this
documentation, although below some tool sets are recommend for handling that.

The following are goals of proper configuration for an application:
* Minimal developer ramp up
* Minimal redundancy of secret information storage
* Ease of discoverability and access of configuration values
* Automation of configuration value replacement wherever possible.

### Mainframe

You MUST store any/all sets of credentials, or other files that would be used
for configuration for all dev/qa/production environments, in Mainframe. An
example of this would be a connection string, a shared secret key for an API
HMAC, a simple remote desktop username/password, or the password to an encrypted
zip file or exported SSL Certificate.

Mainframe limits access to credentials to project members and keeps an audit history
of who accesses them.

Credentials SHOULD be properly labeled as Dev/QA/Prod categories, and SHOULD
appropriately describe the system they are for. Dev/QA/Prod versions of the same
credential SHOULD all be labeled the same but be properly tagged under the
appropriate Dev/QA Category.  Multiple credential sets to a single system SHOULD
be grouped under a single system record.

Credentials for accessing secure files, such as an exported SSH Key Pair, SSL Certificate or encrypted Zip file.

Use the following fields as guidance:

* *Name*: System Name (e.g., Application Database) or Associated file name in
  the Files tab
* *Type*: Service Type (e.g., SQL Server)
* *Description*: (optional) Description if not obvious by the name and type.
* *IP Address / Domain*: Connection information to the system, such as Base URL
  endpoint for a rest API, server name and port (if non-default) for TCP based
  server like SQL Server
* *Credentials*: List any/all credential sets associated with this. If there are
  multiple unique logins for the same environment, these SHOULD be listed here.
  Credential sets for a _different_ environment (e.g., admin user on a
  stage/dev/prod) SHOULD NOT be listed here, and SHOULD be listed as a separate
  system with the same name in the proper category
* *Name*: Description of user account (e.g., Admin, App User)
* *Username / Password*: Credential set for accessing account.


For credentials that do not have a username, such as an SSH key, use dashes “--”
for the name.

When working with multiple projects associated with the same environment, it is
RECOMMENDED that the lead to go through any existing / new systems and properly
associate them with the new project. This means going through any/all systems
associated with the client and ensuring the new project has them associated. Not
doing so will lead to duplicates or difficulty in combing through a large list
of systems.

### Source Control

Assuming source control has it’s own method of controlled access that is private
to The Nerdery and its clients, it is generally acceptable to store secure
values within source control. Nerdery internal Bitbucket or GitHub Private git repo are an
example of a controlled source control environment.

Open source projects ARE NOT considered a controlled environment and this does
not apply to those projects.

If a previously protected project becomes open source, any secure values stored
within source control MUST be invalidated. It is not sufficient to simply add
another commit removing them: they are forever available in git history. There
are bots that scrape GitHub looking for AWS keys and stealing data or running up
thousands of dollars in usage fees. Don't think your project is safe by being
obscure.

If a project has a chance of being open sourced, its better to start good habits
at the beginning and avoid storing secrets in git altogether.

### App Settings Wrapper

Any C# project (class library, web application, etc.) that utilizes an XML
application settings file (app.config, web.config, etc.) SHOULD create a C#
object wrapper for the settings file.  Property getters SHOULD be used for this.
Getters SHOULD read from the System.Configuration namespace to get the
appropriate data out of the configuration file, and the Configuration namespace
SHOULD NOT be used anywhere else within the application.

For example:

```csharp Settings.cs
public static class Settings
{
    public static string ApiKey => ConfigurationManager.AppSettings["ApiKey];

    public static bool ShouldSendEmails => ConfigurationManager.AppSettings["SendEmails"] === "true";
}
```

### Custom Configuration Sections

Custom Configuration sections MAY be used and are RECOMMENDED when configuration
is complex or contains collections.  When used configuration sections SHOULD be
named according to the namespace of the configuration code.  Custom
Configurations MAY transform raw configuration data into properties and methods
as needed.

### Class Configuration

Any classes that require configuration SHOULD declare this configuration need
through constructor arguments and SHOULD NOT access the app settings wrapper
directly.

If a configuration parameter is optional, the value SHOULD be checked for null
and defaulted to a configuration value. If there is no reasonable default
configuration value, an exception SHOULD be thrown.

Good:

```csharp
public class MyCoolApiClient
{
    public MyCoolApiClient(string apiKey) // GOOD!
    {
        // do stuff with it
    }
}
```

Bad:

```csharp
public class MyCoolApiClient
{
    public MyCoolApiClient()
    {
        var apiKey = Settings.ApiKey; // BAD!
    }
}
```

Reaching out to the `Settings` class makes the object hard to test and hard to
use in different contexts.


### Recommended Tool Sets

#### SlowCheetah

SlowCheetah is a visual studio extension (and optional MSBuild target) that
allows your application to apply config transformations to non-web.config files
when using Web Deploy for deployment. It is recommended for use if you’re doing
any config transformations at all, just for the preview capability alone.

VS 2015: https://visualstudiogallery.msdn.microsoft.com/05bb50e3-c971-4613-9379-acae2cfe6f9e

VS 2013 and earlier: https://visualstudiogallery.msdn.microsoft.com/69023d00-a4f9-4a34-a6cd-7e854ba318b5

#### Octopus Deploy

Octopus deploy has a number of capabilities to update config files with values
provided into Octopus, using a decision matrix that is built into the deployment
tool. If Octopus Deploy is available for use on your project, consider using it
for storing your configuration values for deployments.

See: http://docs.octopusdeploy.com/display/OD/Configuration+files

Note: The Nerdery does not currently have an Octopus Deploy server for general
use, although we have used it very successfully on several projects in the past.

#### Azure App Services Configuration

When deploying to an Azure App Service, configuration values can be updated by
Azure at deploy time when stored in the “settings” for your azure app service.
This SHOULD be limited to web.config use only, so if additional configuration
files need to be deployed, use an alternate solution.

See “App Settings” https://azure.microsoft.com/en-us/documentation/articles/web-sites-configure/

Note “Slot-Specific settings” https://azure.microsoft.com/en-us/documentation/articles/web-sites-staged-publishing/

#### Azure Cloud Services Configuration

Configuration for Azure Cloud services are managed through configuration files.
These files SHOULD be stored in source control and SHOULD have an associated
Azure Cloud project for managing them.

See https://azure.microsoft.com/en-us/documentation/services/cloud-services/
