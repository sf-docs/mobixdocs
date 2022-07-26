<details><summary>What is a test case or test scenario and why is it needed?</summary>
A test case is a recorded user's work in the application, so that there is no need to perform the same actions by hand every time to test a new version of the application. It is enough to record a test scenario once, and each subsequent automatic scan will replay the recorded actions in the application. For more information, see the <a href="../ug/testcases/">Test Cases</a> section in the User Guide.

</details>
<details><summary>What is the difference between manual and automatic scanning?</summary>
In manual scanning, the user handles the application (button clicks, text input, etc.) and Mobix then analyzes the captured data and identifies vulnerabilities. In automatic mode, Mobix requires you to record a test scenario. It records all your actions in the application and then plays back all these actions independently in automatic scan mode to simulate the work of a real user. For more information on scan types, see the <a href="../ug/zapusk_skanirovaniya/"> Scan Launch</a> section in the User Guide.

</details>
<details><summary>Is it possible to launch the automatic scanning mode without recording a test scenario?</summary>
Yes, it can be done, using the CLI tool. In this case, dynamic and static checks are performed without analyzing the data generated when the user works with the application. For more information, see the <a href="../ug/zapusk_skanirovaniya/#_4">Scan Launch from the Command Line</a> section in the User Guide.

</details>
<details><summary>When I run an auto-scan, the recorded test case does not play as it should. Why does this happen?</summary>
Since the application being scanned is affected by external actions during the scan, its operation speed can be slightly reduced. For recording test scenarios, it is recommended to take a small delay of 2-3 seconds between actions in the application interface.

</details>
<details><summary>The scan or test scenario is in the "Created" state and does not change. Why did this happen?</summary>
Scanning agents are used to perform a scan and record a test scenario. The scanning agent can process one scan or one test scenario and cannot perform multiple tasks in parallel. If no agents are available, the scan or test scenario is queued and waits for a suitable agent to become available. To solve the problem, you need to check if scans are currently running or test scenarios are being recorded and stop them. For more information, see the <a href="../ug/rezultaty_skanirovanij/">Scan Results</a> section in the User Guide.

</details>
<details><summary>What do the «Main Window lost focus» and «Too much AN messages» errors mean?</summary>
When starting a scan or recording a test scenario, a check is made before the start to ensure that the main window of the application is not overlapped by any messages, dialog boxes, etc. If the application was not loaded within the specified time interval and another application is displayed on the screen, the "Main Window lost focus" error occurs. If the overlapping window is a system error message and the number of these errors is higher than the specified number, the scan will not start with the "Too many ANR messages" error. In this case, it is recommended to restart the scanning agent.

</details>
<details><summary>What kind of checks does Mobix do?</summary>
Mobix currently implements SAST, DAST and IAST practices. This means checking decompiled sources, analyzing the interaction of the application with the system and third-party components, and simulating black-box attacks on applications. A complete list of all vulnerabilities that can be detected is available in the product documentation. Please refer to the <a href="../rg/">Remediation Guide</a>.

</details>
