# Logging

## NuGet Package

### log4net

You SHOULD use [log4net](https://www.nuget.org/packages/log4net/) as your default
choice for a logging framework. It's well understood within The Nerdery and industry
and has many plugins for any target you could need.

### Serilog

If you are intending to write logs to a structured log storage like Logstash,
we recommend [Serilog](https://www.nuget.org/packages/Serilog/).

### Library Selection

When selecting a logging framework, if an existing platform is already in use
that contains a platform for logging, the existing platform logging library
SHOULD be used. As with everything else, it is best to not add redundant
libraries when possible.

For example, Umbraco CMS uses **log4net**. Prefer to use **log4net** over
**Serilog**, as they both perform the same function.

## Log Data:

Logs on a per message basis SHOULD contain the following information:

*REQUIRED*
* DateTime in UTC in 24 hour format, leading 0’s. (yyyy-MM-dd HH:mm:ss,fff)
* Severity / Level
* Path (Class/Method)
* Message

*RECOMMENDED*
* Thread ID

Additionally, you MAY want to include any other data points for being able to
trace specific to a session (session id) or user (user id) for troubleshooting
purposes, depending on your application.

For Example:

```
2016-03-02 08:56:20,133 [StudentRepository, ksykora] Warning (0): Unable to find student by ID 00000000-0000-0000-0000-000000000000
```

You MUST NOT log any data that would be considered personal health information
(PHI) or if discovered by an outsider, would allow a user to gain access to
yours or another system (such as passwords or password hashes, salts, credit
card details, etc.)

## Log Severity / Levels

### Verbose

Overly chatty messaging that would not add value to logs other than being able
to trace exactly what is occurring or dumping entire sets of data.

These statements SHOULD only be enabled temporarily for diagnostic reasons, and
additionally SHOULD be removed before being committed to source control.
Exceptions can be made if critical to gain insight into an environment you do
not have direct debug access to.

It is NOT RECOMMENDED to persist this information to a store during normal
operation.

Example

```
Enter Method GetStudentById()
```

### Debug

Developer oriented information that can assist in troubleshooting or helping to
understand how a framework is behaving. For example, you might use debug
statements to observe application behavior and use the output to tweak constants
or timeouts.

These statements SHOULD only be enabled temporarily for diagnostic reasons.
However, unlike verbose messages, these can be left into source control if they
do not significantly disrupt the readability of your code files.

It is NOT RECOMMENDED to persist this information to a store during normal
operation.

Example

```
Background time remaining {0} seconds.
Background time remaining not specified. Running without timeout.
Response size: {0} bytes
Discovered data characteristics {0}
```

### Informational (Info)

Business or development oriented information marking "interesting" moments in an
application.  User events, communications like email sending, and other business
process info is typical at this level.  This level and higher is typically
enabled in every environment, local to production.

These messages MAY be persisted to a data store (file, database, log storage).

Example

```
Registered new user {0}
Sent push notification {0} to {1}
```

### Warning

Events that are be brought to attention but do not disrupt a user or business
flow.  Application states that are unexpected, performance thresholds exceeded,
user events like exceeding login attempts are example candidates.

These messages SHOULD be persisted to a data store (file, database, log
storage).

Example

```
Unable to find student by ID {0}
Network call timed out, retrying (retry {0}).
```

### Error

Events that disrupt user or business flows but, on a system level, is
recoverable and allow for the application to continue servicing other requests
and continue to run.

These messages MUST be persisted to a data store (file, database, log storage).

Examples

```
Network call failed.
Exception thrown when attempting to perform profile edit action
Received error code {0} from remote system
```

*Note: log frameworks, including log4net and Serilog, typically allow you to
pass the exception directly into the framework. Wherever possible, use those
mechanisms and do not print the exception type, message, or stack trace into
your log message unless they add additional value. Use your log message to
describe the nature under which the exception occurred.*

### Fatal

Anything that terminates processing unexpectedly and would cause an application
to crash outright or cause, on a server level, the system to stop servicing all
requests.

These messages MUST be persisted to a data store (file, database, log storage).

Examples

```
No available disk space
Unable to create web socket listener
```
