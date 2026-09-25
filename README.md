# SOC-detection-response-platform
SOC detection platform using Sigma rules on Wazuh, with attack simulations mapped to MITRE ATT&amp;CK.

# SOC Detection & Response Platform

A custom detection engineering project built on Wazuh, demonstrating the full lifecycle of writing, deploying, and validating custom security detection rules — mapped to real MITRE ATT&CK techniques.

## What This Project Does

- Builds on a Wazuh SIEM deployment (manager, indexer, dashboard, live macOS agent)
- Implements a custom log source pipeline for security event injection
- Writes and deploys a custom Wazuh detection rule from scratch
- Validates the full detection chain: log generation to alert firing
- Maps detections to MITRE ATT&CK (T1548.003 - Abuse Elevation Control Mechanism)

## Architecture

[Log Source] --> [Wazuh Agent] --> [Wazuh Manager: Rule Engine] --> [Indexer] --> [Dashboard Alert]

## Custom Detection Rule

Located in `sudo_rule.xml`. Deployed to the Wazuh manager's `local_rules.xml`.

```xml
<group name="local,syscheck,">
  <rule id="100010" level="12">
    <match>SOC_TEST_ALERT</match>
    <description>Suspicious activity detected - simulated attack</description>
    <mitre>
      <id>T1548.003</id>
    </mitre>
    <group>privilege_escalation,mitre_attack,</group>
  </rule>
</group>
```

## Attack Simulation

A custom log source (`/var/log/soc-demo/security.log`) simulates a privilege escalation attempt:

```bash
echo "$(date '+%b %d %H:%M:%S') $(hostname) security-monitor: SOC_TEST_ALERT suspicious privilege escalation attempt detected" | sudo tee -a /var/log/soc-demo/security.log
```

This triggers a Level 12 (High Severity) alert in the Wazuh dashboard within seconds.

## Setup

1. Requires a running Wazuh manager (see companion project: [SIEM-log-analysis-platform](https://github.com/PrajwalVijaykumar123/SIEM-log-analysis-platform))
2. Deploy the custom rule:
```bash
   docker cp sudo_rule.xml <manager-container-name>:/var/ossec/etc/rules/local_rules.xml
   docker restart <manager-container-name>
```
3. Configure the agent to monitor a custom log source (see `ossec.conf` localfile block)
4. Trigger the simulation command above
5. View the alert in the Wazuh dashboard under Events / Threat Hunting

## What I Learned

- Wazuh's rule engine: how `if_sid` chaining works, and why standalone rules are sometimes more reliable when working with non-standard log sources
- Debugging a full SIEM pipeline: log source to agent to manager to rule engine to dashboard
- Docker-based management of Wazuh's manager container (copying config files, restarting services)
- Mapping custom detections to MITRE ATT&CK techniques
- macOS-specific logging quirks (unified logging system vs. legacy syslog files) and how they affect security tooling originally designed for Linux

## Next Steps

- Add more Sigma-inspired rules covering additional MITRE techniques (e.g., reverse shell detection, brute-force login attempts)
- Build an automated attack simulation script covering multiple scenarios
- Add rule severity tuning and false-positive reduction
