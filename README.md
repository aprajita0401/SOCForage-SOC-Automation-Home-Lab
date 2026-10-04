# SOC Automation Home Lab | Wazuh, Shuffle, VirusTotal & TheHive

A hands-on Security Operations Center (SOC) home lab demonstrating endpoint monitoring, security event detection, threat intelligence enrichment, incident management, and automated email notifications.

## Overview

This project simulates a basic SOC workflow by integrating open-source security tools and automation platforms. It demonstrates how suspicious endpoint activity can be collected, analyzed, enriched with threat intelligence, documented as an incident, and forwarded to an analyst through an automated workflow.

The lab uses a Windows virtual machine as the monitored endpoint, Wazuh for security monitoring and detection, Shuffle for security orchestration and automation, VirusTotal for file-hash reputation lookups, and TheHive for incident case management.

The objective is to understand how the components of a SOC work together to reduce repetitive manual investigation tasks and improve the consistency of incident handling.

## Objectives

* Build a virtualized endpoint monitoring environment.
* Collect Windows security telemetry using Sysmon.
* Forward endpoint events to Wazuh using the Wazuh agent.
* Review security events and create a custom Wazuh detection rule.
* Integrate Wazuh with Shuffle using a webhook.
* Use SHA-256 file-hash information for VirusTotal lookups.
* Integrate Shuffle with TheHive using API authentication.
* Configure email notifications through a Shuffle workflow.
* Understand the end-to-end process of security event detection and incident handling.

## Architecture

```text
                  Windows 11 VM
                (VirtualBox)
                      |
              +-------+-------+
              |               |
            Sysmon        Wazuh Agent
              |               |
              +-------+-------+
                      |
                Endpoint Events
                      |
                      v
               Wazuh Manager
             /        |        \
            /         |         \
      Wazuh Indexer   Rules   Dashboard
                      |
                Security Alert
                      |
                      v
               Shuffle Webhook
                      |
                 Shuffle SOAR
                      |
              +-------+--------+
              |       |        |
              v       v        v
          SHA-256  VirusTotal TheHive
           Hash     Lookup     Cases
                               |
                               v
                         Email Alert
```

*Note: This is a conceptual representation of the workflow. The exact data mappings and execution order depend on the configured Shuffle playbook.*

## Tools and Technologies

| Tool                             | Purpose                                                                |
| -------------------------------- | ---------------------------------------------------------------------- |
| VirtualBox                       | Runs the Windows virtual machine used as the test endpoint.            |
| Vultr                            | Hosts the cloud-based security infrastructure.                         |
| Windows 11                       | Generates endpoint activity for monitoring and testing.                |
| Sysmon                           | Records detailed Windows system activity.                              |
| Wazuh Agent                      | Collects configured endpoint logs and forwards them to Wazuh.          |
| Wazuh Manager                    | Analyzes incoming events and evaluates detection rules.                |
| Wazuh Indexer                    | Stores and indexes data for searching and analysis.                    |
| Wazuh Dashboard                  | Provides a web interface for monitoring and investigation.             |
| Shuffle                          | Orchestrates the automated security workflow.                          |
| VirusTotal v3 API                | Provides file-hash reputation and threat-intelligence information.     |
| TheHive                          | Organizes incident cases and investigation information.                |
| SHA-256                          | Provides a file fingerprint for hash-based lookups.                    |
| Apache Cassandra / Elasticsearch | Supporting data services documented in the lab installation procedure. |

## Lab Environment

The lab consists of:

* A Windows virtual machine running in VirtualBox.
* A Vultr instance hosting Wazuh.
* A separate Vultr instance intended to host TheHive.
* Sysmon and the Wazuh agent configured on the Windows endpoint.
* A Shuffle workflow connected to VirusTotal and TheHive.
* An email action configured to notify a recipient.

### Prerequisites

* A host computer with sufficient RAM and disk space.
* VirtualBox installed.
* A Windows virtual machine.
* Linux command-line and SSH basics.
* Access to a cloud server provider.
* A Wazuh deployment.
* Shuffle account access.
* VirusTotal API access.
* A TheHive deployment and API credentials.
* An email integration supported by the selected Shuffle workflow.

Resource requirements depend on the number of services running simultaneously. The original lab documentation recommends at least 8 GB of RAM and 50 GB of free disk space on the host.

## Implementation

### 1. Endpoint Telemetry with Sysmon

Installed Sysmon on the Windows virtual machine and configured it using a Sysmon configuration file.

Sysmon records selected system activity, such as process creation and other security-relevant events, depending on the enabled configuration.

**Purpose:** Generate detailed endpoint telemetry for security monitoring.

### 2. Wazuh Deployment

Deployed Wazuh on a cloud instance and configured the manager, indexer, and dashboard.

Accessed the dashboard to monitor the environment and review incoming security data.

**Purpose:** Centralize endpoint monitoring, event analysis, and alert generation.

### 3. Wazuh Agent Integration

Installed the Wazuh agent on the Windows VM and configured the agent to collect the Sysmon Operational event channel.

Restarted the agent after updating its configuration.

**Purpose:** Forward relevant Windows telemetry to the Wazuh manager.

### 4. Custom Detection Rule

Reviewed collected event data in Wazuh Discover and configured a custom rule named `Mimikatz Usage Detection`.

Used controlled test activity to evaluate whether the rule matched the expected event data.

**Purpose:** Learn how custom detection logic can identify selected suspicious behavior.

### 5. Wazuh and Shuffle Integration

Created a Shuffle workflow and configured a webhook to receive data from the Wazuh integration.

Tested the webhook and inspected the workflow's debug output.

**Purpose:** Connect security alerting with automated downstream actions.

### 6. VirusTotal Enrichment

Added a SHA-256 hashing step and the VirusTotal v3 integration to the Shuffle playbook.

Configured authentication and the data passed to the hash-report action.

**Purpose:** Retrieve external reputation information associated with the relevant file hash.

A VirusTotal result provides additional investigation context; it does not independently prove that a file is malicious or safe.

### 7. TheHive Integration

Created separate user accounts for analyst and automation use, generated an API key for the SOAR account, and configured TheHive authentication in Shuffle.

Connected the TheHive action to the workflow.

**Purpose:** Pass incident information to a case-management platform for investigation and tracking.

### 8. Email Notification

Configured an email action in the Shuffle workflow and tested the notification path.

**Purpose:** Notify the configured recipient when the workflow reaches the email action.

## Workflow

1. Test activity is generated on the Windows VM.
2. Sysmon records the selected endpoint events.
3. The Wazuh agent forwards configured events to the Wazuh manager.
4. Wazuh evaluates the events against its rules and generates alerts when conditions match.
5. The configured integration sends relevant alert data to Shuffle.
6. Shuffle processes the data and performs the configured SHA-256 and VirusTotal actions.
7. The workflow submits incident information to TheHive.
8. The configured email action sends a notification.

The exact workflow behavior depends on the alert payload, field mappings, API responses, and playbook configuration.

## Validation and Testing

The lab documentation describes testing the pipeline using Mimikatz as a controlled security test program.

Validation should be performed at each stage:

| Component   | Validation                                                           |
| ----------- | -------------------------------------------------------------------- |
| Sysmon      | Confirm relevant events appear in Event Viewer.                      |
| Wazuh Agent | Confirm the endpoint is connected and sending data.                  |
| Wazuh       | Confirm events are indexed and the intended detection rule triggers. |
| Shuffle     | Confirm the webhook receives the expected data.                      |
| SHA-256     | Confirm the hash corresponds to the intended executable.             |
| VirusTotal  | Confirm the hash-report action returns a valid response.             |
| TheHive     | Confirm the intended case or incident record is created.             |
| Email       | Confirm the notification is delivered to the configured recipient.   |

A successful test at one stage does not guarantee that every downstream integration works. Each stage should be validated independently.

## Security Considerations

* Use an isolated, disposable virtual machine for security testing.
* Do not execute credential-access tools on production systems or devices containing real credentials.
* Keep Microsoft Defender and other endpoint protections enabled wherever possible.
* Do not exclude the entire system drive from antivirus scanning.
* Restrict cloud firewall rules and administrative access.
* Never commit API keys, passwords, webhook URLs containing secrets, or private server addresses.
* Use environment variables or a secrets manager for sensitive configuration.
* Apply least-privilege permissions to automation accounts.
* Treat third-party threat-intelligence results as supporting evidence, not definitive verdicts.
* Restore security settings and remove test artifacts after the lab is complete.

## Limitations

* Detection depends on the telemetry collected and the configured Wazuh rules.
* Hash-based lookups depend on obtaining the correct file hash.
* A lack of VirusTotal detections does not guarantee that a file is safe.
* Automated case creation and notification depend on successful API authentication and field mappings.
* This lab demonstrates selected detection and response workflow stages. It does not demonstrate a complete enterprise SOC or automated endpoint containment.

## Skills Demonstrated

* SIEM fundamentals and security event analysis.
* Windows endpoint monitoring.
* Sysmon configuration.
* Wazuh agent deployment and log collection.
* Custom detection rule configuration.
* Linux command-line administration.
* SSH and service management.
* Webhook integration.
* REST API authentication concepts.
* Threat intelligence enrichment.
* SOAR workflow design.
* Incident case-management integration.
* Security troubleshooting and validation.

## Future Improvements

* Add more Windows event sources and detection rules.
* Test additional attack simulations using safe, controlled techniques.
* Add workflow error handling and API failure paths.
* Implement duplicate-alert handling.
* Include a structured incident severity and triage process.
* Improve access restrictions and secrets management.
* Add screenshots of successful detections, workflow execution, TheHive cases, and email delivery.
* Document test cases with expected and actual results.
* Explore additional response actions in an isolated test environment.

## Documentation

See [`documentation/`](docs/) for architecture, setup notes, configuration examples, and validation evidence.


## Security Disclaimer

This project is intended for educational and defensive cybersecurity purposes in an isolated lab.

Mimikatz was used as a test program to generate suspicious activity. Do not run credential-related tools on systems or accounts without explicit authorization.

Do not expose administrative interfaces or API keys publicly. Avoid disabling endpoint security on everyday or production systems. Any temporary changes made for lab testing should be reversed after testing.

## Documentation

* [Project documentation and installation guide](docs/SOC Automation Home Lab.pdf)
* [Screenshots and configuration evidence](screenshots/)

## Author

**Aprajita Nandkeuliar**

Cybersecurity | SOC Operations | SIEM | SOAR | Threat Intelligence
