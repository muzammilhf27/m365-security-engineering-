# Phishing Investigation Workflow (Microsoft Defender)

## Overview
This document outlines the practical approach I follow to investigating phishing incidents using Microsoft 365 Defender.

## Detection
- User-reported phishing
- Defender alerts
- Suspicious sign-in activity

## Investigation Steps
1. Review message headers and sender reputation
2. Use message trace to identify affected users
3. Analyse URLs and attachments
4. Check user sign-in logs for suspicious activity

## Response Actions
- Remove malicious emails
- Block sender/domain
- Reset user credentials if required
- Enforce MFA or Conditional Access if not already done

## Lessons Learned
- Importance of impersonation protection
- Value of user awareness training
- Continuous tuning of anti-phish policies

## ProTips

-Be proactive with Threat Intelligence
Leverage Threat Intelligence in Microsoft Defender to review indicators of compromise (IOCs) such as malicious domains, URLs, IPs, and file hashes. You can proactively block publicly available threat indicators before they are observed in your environment.

-Safely analyse URLs and attachments
If you want to go the extra mile, use Windows Sandbox or an isolated analysis environment to safely inspect suspicious URLs or attachments without risking your production system.

-Automate where possible
Use Advanced Hunting and automated investigation & response (AIR) capabilities to reduce investigation time and contain phishing threats faster.
