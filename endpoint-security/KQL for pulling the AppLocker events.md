```kusto
DeviceEvents
| where Timestamp > ago(2d)
| where ActionType in (
    "AppControlCodeIntegrityPolicyBlocked",
    "AppControlExecutableBlocked",
    "AppControlCodeIntegrityPolicyAudited",
    "AppControlExecutableAudited",
    "AppControlScriptAudited",
    "AppControlMSIAudited",
    "AppControlDllAudited",
    "AppControlCodeIntegrityDriverRevoked",
    "AppControlCIScriptBlocked",
    "AppControlCIScriptAudited",
    "AppControlCodeIntegrityOriginBlocked",
    "AppControlPackagedAppAudited",
    "AppControlPackagedAppBlocked"
)
| project
    Timestamp,
    ActionType,
    DeviceName,
    FileName,
    FolderPath,
    InitiatingProcessFileName,
    InitiatingProcessAccountName
| order by Timestamp desc
```
