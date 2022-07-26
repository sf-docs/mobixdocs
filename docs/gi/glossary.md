# Glossary

**Project** — a top-level entity that contains application-specific scan settings and profiles. Each project is explicitly linked to the application package to be scanned, either during project creation or during the first scan. At the project level, only the scan rules to be applied to that project can be redefined. The project name is unique within the system.

**Profile** — a scan profile that contains the settings of each module, a list of standards and information security requirements to be checked. The profile name is unique within its project.

**Module** — a component for collecting various information when scanning an application on the device (monitoring the system log, file usage, database operations, etc.). Each module has its own unique settings and may depend on the results of other modules.

**Test cases** — a recorded scenario of the user interaction with the application. It includes all user actions (mouse clicks, text inputs, any interaction with the application interface, etc.). Test case is linked to a particular project and can only be reproduced within that project. A test case can only be executed if the name of the application under test (package\_id) matches the name of the application used to record the test case.

**Scan** — the process of application analysis, by manual interaction with the application by the user or by the system using pre-recorded test cases. During the scan, the system collects all available information about the application and then searches for vulnerabilities and checks for compliance with security standards.

**Scan/launch method** — the method that determines whether to run the previously recorded test case or wait for manual operations on the application. There are two options:

* **Auto** — starts scanning and runs the selected test case.
* **Manual** — after starting a scan, you need to manually perform operations on the running application.

**Package name** — the name of the package of the application to be scanned.

**Rules** — the scan analysis rules used to search for part of the vulnerabilities. Rules are a set of strings or regular expressions to be searched in the collected data. For convenience, adding rules is designed as a constructor. In this constructor you have to specify what string to search for, the list of modules that collected data for the search, and the exact location of the data (XML tag, JSON value, etc.).

**Requirement** — the application can be checked for compliance with the information security requirement. Each requirement has certain types of defects associated with it. If these defects are found in the application, the requirement is considered not fulfilled. Requirements can be grouped into categories or apply directly to standards.

**Category** — a set of information security requirements grouped by specific criteria.

**Standard** — a set of information security requirements or categories of information security requirements. The application can be checked for compliance with the information security standard. The standards can be either global or internal company standards.

**Defects** — application security defects, i.e. vulnerabilities discovered during the scan. Each defect has a type, description and recommendations for remediation.

**Collected Data** — all information collected during the scan about the operation of the application. The data is divided into modules. Each module is responsible for collecting specific information. This data can be downloaded and analyzed if required.

**CI/CD (Continuous Integration / Continuous Delivery)** — systems for continuous integration and continuous delivery of the application. Examples of such systems are Jenkins, TeamCity, GitLab CI.

**Emulator** — a virtual emulator that simulates a real Android device. It can have various architecture and version of operating system.