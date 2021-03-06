# Security

As a developer, you are expected to be considering the security of the
applications you develop at all times. You need to be aware of basic security
principles and work regularly to stay educated about security vulnerabilities
and how to reduce exposure to your applications.

**Secure Programmer's Pledge**

1. I will not store sensitive data in plain text, I will protect it in a suitable manner.
2. I will always protect my users' data as if it was my own.
3. I will only use vetted and published algorithms, I will not invent my own.
4. I will not assume that I know better, but instead will try to constantly learn.
5. I will not trust the security of systems that I have not personally examined.
6. I will always try to educate others.

### Required Reading

OWASP Top Ten

[https://www.owasp.org/index.php/Top10#OWASP_Top_10_for_2013](https://www.owasp.org/index.php/Top10#OWASP_Top_10_for_2013)

Additionally, review each of the items underneath the top 10.

Development Security Standards:

[https://mainframe.nerdery.com/docs/article/development-security-standards](https://mainframe.nerdery.com/docs/article/development-security-standards)

Data Security Policy for Developers: [https://mainframe.nerdery.com/docs/article/data-security-policy-for-development](https://mainframe.nerdery.com/docs/article/data-security-policy-for-development)



# ASP.NET / IIS Development Security Standards


## MUST for all applications

#### Configuration files or any other sensitive files are not accessible by the user

This is enabled by default in IIS via machine.config to prevent .config files
from being delivered to the user. Additionally, files under the App_Data
directory, as well as any files that do not have a MIME type mapping in
web.config will not be served to users.

Data files not necessary to be accessed by users SHOULD either be stored in
App_Data, or in a directory outside of the webroot.

See the standards on [Configuration](configuration.md) for more information on
configuration files.


## MUST for application which store and transmit user information such as usernames, passwords and/or email addresses.

#### A password requires at least six characters

In ASP.NET Identity, this is already set to 6 by default. To create a more
complex set of requirements, implement a custom IIdentityValidator.

#### Passwords are hashed using a modern algorithm with a random unique salt

The encryption that ships with ASP.NET Identity uses KDF which meets these
requirements.

#### Web apps: Session ID's are not read from or displayed in URLs

ASP.NET enables this by default.

#### Web apps: Session ID is rotated or regenerated after authentication or change in access level

Not supported out-of-box by ASP.NET

#### Application provides a logout feature to destroy session information

Use Session.Abandon()

#### User sessions should automatically expire after a set period of time

Default time for ASP.NET is 20 minutes, which should be sufficient for most
purposes. This can be changed via session timeout:
[https://msdn.microsoft.com/en-us/library/h6bb9cz9(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/h6bb9cz9(v=vs.85).aspx)

#### Web apps: Set the HttpOnly cookie flag for session or sensitive cookies

This is enabled by default in ASP.NET and cannot be changed

#### Web apps: If HTTPS is used application-wide, session cookies MUST use the "secure" flag

Set this via web.config

See: [http://msdn.microsoft.com/en-us/library/ms228262(v=vs.100).aspx](http://msdn.microsoft.com/en-us/library/ms228262(v=vs.100).aspx)

#### Authentication credentials, registration data and sensitive data are transmitted over SSL (in production)

Enforce SSL by using HTTPS Redirects with the URL Rewrite module and enabling
HSTS via HTTP Headers. See this post for more info:

[http://www.hanselman.com/blog/HowToEnableHTTPStrictTransportSecurityHSTSInIIS7.aspx](http://www.hanselman.com/blog/HowToEnableHTTPStrictTransportSecurityHSTSInIIS7.aspx)

#### 'Forgot Password' generates a unique token which is sent to confirmed email

This is enabled by default in ASP.NET Identity

#### All input data is validated and sanitized

Use MVC validation mechanisms

#### All application output is escaped or filtered

Use Razor outputs and standard serializers. Avoid using Html.Raw() unless you
are very aware of what is being output.

#### Input validation is executed on the server side, instead of or in addition to client-side

Use MVC validation mechanisms

#### Web apps: Forms contain a unique token, generated on the page view that is validated upon submission

Use Html.AntiForgeryToken().

When using the anti forgery token, it is important to set a machine key for the
application to prevent token decryption failures when the application recycles
and to ensure that the token can be decrypted across all machines in a cluster.
See https://support.microsoft.com/en-us/kb/312906 for an application that can be
used to generate a machine key.

Different machine keys should be used for **Staging** and **Production**
configurations.

#### Web apps: Properly encode variables within URLs

Use Uri.EscapeDataString()

#### Maximum file upload size is limited to a practical, realistic amount for the given application

Use the web.config to set the maximum upload size:

```
<configuration>
  <system.web>
    <httpRuntime maxRequestLength="xxx" />
  </system.web>
</configuration>
```

#### Uploaded files are always restricted to the appropriate file types

#### Web apps: Upload directory SHOULD NOT use open permissions

Write to a directory in App_Data, cloud storage, or other directory on the file
system outside of the web root.

#### Queries to external services are properly escaped or parameterized

Use standard APIs for interacting with external services rather than inputting
raw queries. Avoid string concatenation with user input.


## MUST for applications which store and transmit sensitive data such as credit card numbers, social security numbers, or other types of very sensitive information

#### Passwords require at least eight characters and contain at least one digit, uppercase letter and lowercase letter

For ASP.NET Identity create a custom IIdentityValidator

#### Account is locked out after a determined number of invalid login attempts

For ASP.NET Identity, See [http://www.jlum.ws/post/2014/5/27/user-lockouts-in-aspnet-identity-2-with-aspnet-mvc-5](http://www.jlum.ws/post/2014/5/27/user-lockouts-in-aspnet-identity-2-with-aspnet-mvc-5)

#### Any sensitive data (e.g., SSN) being stored SHOULD be encrypted

Leverage SQL Server encrypted columns for storage of this data.

