# Feature Abuse
- Targeting enterprise applications which are not built keeping security in mind.
- On Windows, many enterprise applications need either Administrative privileges or SYSTEM privileges.
- Machines used by developers mostly have enterprise applications in external services.
- Continuous Integration (CI) tools mostly have capabilities run OS-level command execution.
### Jenkins
- Jenkins is a widely used Continuous Integration tool.
- Jenkins application mostly does not have authentication if it is located in internal network.
- If there is an authentication mechanism, check username as a password to log in.

* `http://jenkin-server:8080/script`: In the script console, we can execute groovy scripts.

