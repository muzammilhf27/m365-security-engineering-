# Attack Surface Reduction (ASR) Rules – Enterprise Implementation using GPO & Intune

## Overview

This guide is a **real-world implementation of Microsoft Defender Attack Surface Reduction (ASR) rules** in a **hybrid enterprise environment**.

The environment consisted of:

* **Microsoft Intune (MDM-managed devices)**
* **Group Policy (GPO-managed devices)**

The primary objective was to **reduce endpoint attack surface while maintaining business continuity**. To achieve this, ASR rules were deployed using a **phased rollout approach**, starting in **Audit mode**, reviewing Defender telemetry, creating exclusions where required, and finally enforcing the rules in **Block mode**.

This documentation focuses on:

* Step-by-step **deployment using Intune and GPO**
* Monitoring and validation using **Microsoft Defender**
* Managing **ASR exclusions**
* Operational considerations in **hybrid and MECM-integrated environments**
* Lessons learned from operating ASR rules in production

---

## Environment Design

* **Endpoint Security Platform**: Microsoft Defender for Endpoint
* **MDM**: Microsoft Intune
* **On-prem Management**: Group Policy
* **Initial ASR Mode**: Audit
* **Final ASR Mode**: Block

Some devices were managed using Intune, while others remained GPO-managed based on operational and tooling requirements.

---

## ASR Rule to GUID Matrix

| Rule Name                                                           | Rule GUID                            |
| ------------------------------------------------------------------- | ------------------------------------ |
| Block abuse of exploited vulnerable signed drivers                  | 56a863a9-875e-4185-98a7-b882c64b5ce5 |
| Block Adobe Reader from creating child processes                    | 7674ba52-37eb-4a4f-a9a1-f0f9a1619a2c |
| Block all Office applications from creating child processes         | d4f940ab-401b-4efc-aadc-ad5f3c50688a |
| Block credential stealing from LSASS                                | 9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2 |
| Block executable content from email client and webmail              | be9ba2d9-53ea-4cdc-84e5-9b1eeee46550 |
| Block executable files unless they meet prevalence/age/trusted list | 01443614-cd74-433a-b99e-2ecdc07bfc25 |
| Block execution of potentially obfuscated scripts                   | 5beb7efe-fd9a-4556-801d-275e5ffc04cc |
| Block JavaScript or VBScript from launching downloaded executables  | d3e037e1-3eb8-44c8-a917-57927947596d |
| Block Office applications from creating executable content          | 3b576869-a4ec-4529-8536-b80a7769e899 |
| Block Office applications from injecting code                       | 75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84 |
| Block Office communication apps from creating child processes       | 26190899-1602-49e8-8b27-eb1d0a1ce869 |
| Block persistence through WMI event subscription                    | e6db77e5-3df2-4cf1-b95a-636979351e5b |
| Block process creation from PSExec and WMI                          | d1e49aac-8f56-4280-b9ba-993a6d77406c |
| Block rebooting machine in Safe Mode                                | 33ddedf1-c6e0-47cb-833e-de6133960387 |
| Block untrusted and unsigned processes from USB                     | b2b3f03d-6a65-4f7b-a9c7-1c7ef74a9ba4 |
| Block use of copied or impersonated system tools                    | c0033c00-d16d-4114-a5a0-dc9b3a7d2ceb |
| Block Webshell creation (Servers)                                   | a8f5898e-1dc8-49a9-9878-85004b8a61e6 |
| Block Win32 API calls from Office macros                            | 92e97fa1-2edf-4476-bdd6-9dd0b4dddc7b |
| Use advanced protection against ransomware                          | c1db55ab-c21a-4637-bb3f-a12568109d35 |

---

## ASR Rule Modes

| Mode                      | Code | Description                       |
| ------------------------- | ---- | --------------------------------- |
| Disabled / Not configured | 0    | Rule is not enabled               |
| Block                     | 1    | Rule is enforced                  |
| Audit                     | 2    | Rule is evaluated but not blocked |
| Warn                      | 6    | User is warned and can bypass     |

---

## Phase 1 – Initial Deployment (Audit Mode)

All ASR rules were initially deployed in **Audit mode** across both Intune and GPO-managed devices to safely assess impact.

---

## ASR Deployment Using Microsoft Intune

### Create ASR Policy in Intune

1. Go to **Intune Admin Center**
2. Navigate to:
   **Endpoint security → Attack surface reduction**
3. Select **Create policy**
4. Platform: **Windows 10 and later**
5. Profile: **Attack surface reduction rules**
6. Name the policy appropriately (e.g., `ASR – Audit Mode`)

---

### Configure ASR Rules (Audit Mode)

1. Set required ASR rules to **Audit**
2. Leave high-risk rules disabled initially if needed
3. Save configuration

---

### Assign Policy

1. Assign to a **pilot device group**
2. Expand assignment to broader groups after validation

---

## ASR Deployment Using Group Policy (GPO)

### Configure ASR Rules via GPO

1. Open **Group Policy Management Console**
2. Create or edit a GPO
3. Navigate to:

```
Computer Configuration
→ Administrative Templates
→ Windows Components
→ Microsoft Defender Antivirus
→ Microsoft Defender Exploit Guard
→ Attack Surface Reduction
```

4. Open **Configure Attack Surface Reduction rules**
5. Enable the policy
6. Add rules in the format:

```
<Rule GUID>=<Mode>
```

Example:

```
d4f940ab-401b-4efc-aadc-ad5f3c50688a=2
```

(Mode `2` = Audit)

7. Apply and link the GPO to the appropriate OU

---

## Monitoring & Validation

### Advanced Hunting (KQL)

```kql
DeviceEvents
| where Timestamp > ago(30d)
| where ActionType startswith "Asr"
| summarize EventCount = count() by ActionType
```

Used to identify:

* Frequently triggered rules
* Devices impacted
* Rules requiring exclusions

---

### Defender Portal Reports

```
Microsoft Defender Portal
→ Reports
→ Endpoints
→ Attack Surface Reduction
```

Used to review audit and block activity visually.

---

## ASR Exclusions Strategy

Exclusions were added **only when business impact was confirmed**.

Exclusions can be:

* **Global** (applies to all ASR rules)
* **Rule-specific**

---

## ASR Exclusions – Intune

### Global Exclusions

1. Endpoint security → Attack surface reduction
2. Open ASR policy
3. Navigate to **Exclusions**
4. Add file, folder, or process paths

---

### Rule-Specific Exclusions

1. Open ASR policy
2. Select specific ASR rule
3. Add exclusions for that rule only

---

## ASR Exclusions – GPO

### Global Exclusions

```
Computer Configuration
→ Administrative Templates
→ Windows Components
→ Microsoft Defender Antivirus
→ Microsoft Defender Exploit Guard
→ Attack Surface Reduction
```

* Configure **Exclude files and paths from ASR rules**

---

### Rule-Specific Exclusions

* Configure **Configure Attack Surface Reduction rules per rule exclusions**
* Apply exclusions to specific GUIDs

---

## Important Consideration – SCCM / MECM Environments

### Block process creation from PSExec and WMI

**Rule GUID:** `d1e49aac-8f56-4280-b9ba-993a6d77406c`

This rule **should NOT be applied** to **SCCM / MECM-managed devices**.

**Reason:**

* MECM relies on **WMI** for inventory, deployments, baselines, and remote actions

**Recommended approach:**

* Apply this rule **only to Intune (MDM)-managed devices**

---

## Phase 2 – Audit Review Period

* Rules remained in Audit mode for ~2 weeks
* Defender telemetry reviewed regularly
* Required exclusions added and validated

---

## Phase 3 – Enforcement (Block Mode)

* Rules moved from **Audit (2)** to **Block (1)**
* Updated in both Intune and GPO
* No major operational impact observed

---

## Lessons Learned

* Audit mode is critical for safe ASR rollout
* Defender telemetry and KQL are essential
* Exclusions must be minimal and justified
* Endpoint management tooling must be considered
* Hybrid environments require policy alignment
* Gradual enforcement significantly reduces risk

---

## Conclusion

This implementation demonstrates a **practical and production-ready approach** to deploying ASR rules in a hybrid enterprise environment.

By combining phased deployment, telemetry-driven decisions, careful exclusion management, and awareness of management tooling dependencies, it is possible to significantly reduce endpoint attack surface while maintaining operational stability.

This repository serves as a **technical reference and proof of hands-on experience** for security and endpoint administrators deploying ASR rules in real-world environments.

