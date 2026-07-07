# Step 2: Validate Required Fields

## Purpose

Confirm that each log source contains the minimum fields required for SIEM search, detection, investigation, event correlation, reporting, and normalisation.

This step validates whether the incoming data contains enough context to support security use cases. A log may be technically ingesting, but if key fields are missing, inconsistent, or only available in free text, its operational value may be limited.

## Validation Method

For each log source and log type, retrieve a representative set of sample events from the SIEM.

Review the raw event and extracted fields to confirm whether the required fields are present.

For each field category, record one of the following values:

| Status | Meaning |
|---|---|
| Y | Field is present and reliably populated |
| N | Field is missing |
| Partial | Field is present only for some event types, conditions, or authentication states |

Add notes explaining the field mapping, any caveats, and whether the field is extracted directly, derived, normalised, or conditionally populated.

This table becomes the living validation record for the log source.

## Required Field Validation Table

| Field Category | Expected Field Names | Example Value | Required Validation | Notes |
|---|---|---|---|---|
| Log received date/time | `@timestamp`, `eventtime`, `_time` | `2026-07-08T04:15:22Z` | Must be present and normalised to UTC | Represents when the SIEM received, indexed, or normalised the event time |
| Log creation date/time | `event.created`, `created_time`, `device_time` | `2026-07-08T04:15:18Z` | Must be present where the source provides it | Represents when the event was created by the originating system |
| System type | `device_type`, `product`, `vendor`, `sourcetype` | `firewall`, `Windows Security`, `FortiGate` | Must identify the source technology or product | Example: firewall, endpoint, operating system, identity provider, vendor/product name |
| Source address | `src_ip`, `src_host`, `source.ip`, `host`, `host.name` | `10.10.25.14`, `workstation-01` | Must identify the source system where applicable | May be IP address, hostname, device name, or workload identity |
| Device / asset identifier | `mac_address`, `device_id`, `asset_id`, `host.id`, `agent.id`, `endpoint_id`, `serial_number`, `instance_id`, `computer_id` | `00:1A:2B:3C:4D:5E`, `i-0abc123def456` | Required where the source provides a stable device, host, endpoint, asset, or workload identifier | Used to correlate activity to a specific device or asset, especially where IP address or hostname may change |
| User identity | `user`, `usrName`, `user.name`, `src_user` | `jsmith`, `DOMAIN\jsmith` | Must be present for authenticated activity | Mark as Partial where only populated for authenticated or user-linked events |
| Severity / priority | `level`, `severity`, `priority`, `risk` | `high`, `critical`, `5` | Must indicate event importance where supported | Confirm whether values are vendor-specific or normalised |
| Action / message | `event.action`, `action`, `event.type`, `event_type`, `msg`, `message`, `status_code` | `user_login`, `connection_blocked`, `4625` | Must describe what occurred, including the event type or action taken | Should support investigation without relying only on raw text. Where applicable, include event type and status code to clarify the activity |
| Command / process executed | `command`, `command_line`, `process.command_line`, `process.name`, `process.executable`, `cmdline`, `process.args`, `script_block_text` | `powershell.exe -enc ...`, `net user test /add` | Required where the log source records command execution, process creation, script execution, shell activity, or administrative actions | Important for endpoint, server, EDR, PowerShell, Linux auditd, cloud admin, and privileged activity logs |
| Impacted object | `object`, `file`, `account`, `target`, `dest`, `destination`, `resource` | `C:\Temp\payload.exe`, `admin_user`, `server-02` | Required where the event involves a target object | Mark as Partial where only relevant to file, account, process, configuration, resource, or access events |
| Result of activity | `status`, `outcome`, `result`, `event.outcome`, `status_code`, `response_code` | `success`, `failure`, `403`, `blocked` | Must indicate success, failure, allowed, blocked, denied, error, or equivalent | Required for authentication, access, policy, API, application, and control events. Status codes should be mapped or interpreted where possible |
| Session / transaction ID | `session_id`, `transaction_id`, `trace_id`, `correlation_id`, `request_id`, `event.id` | `abc123-session-789`, `req-45f9a1` | Required where the source supports session, transaction, request, or correlation tracking | Used to link related events across a single user session, authentication flow, network connection, API request, transaction, or investigation timeline |
| Unique event identifier | `event.id`, `event_id`, `id`, `record_id`, `uid`, `uuid`, `guid`, `message_id` | `550e8400-e29b-41d4-a716-446655440000` | Required where the source provides a unique identifier for each event | Used to uniquely identify a specific log record and assist with deduplication, event correlation, investigation traceability, and cross-system referencing |

## Evidence to Capture

Record the following evidence:

- Log source name
- Product or technology
- Ingestion method
- Index and sourcetype
- Sample event timestamp
- Sample raw event
- Extracted fields observed
- Required field status: Y, N, or Partial
- Observed field names
- Field mapping notes
- Caveats or conditional population rules
- Validation result

## Example

For an Active Directory log source ingested through Winlogbeat, the validation record may show:

| Field Category | Status | Observed Field | Notes |
|---|---:|---|---|
| Log received date/time | Y | `@timestamp` | Normalised to UTC |
| Log creation date/time | Y | `event.created` | Created time from Windows event metadata |
| System type | Y | `winlog.provider_name`, `event.provider` | Identifies Windows Security event provider |
| Source address | Partial | `source.ip`, `host.name` | Source IP only present for some network logon events |
| Device / asset identifier | Y | `agent.id`, `host.id` | Stable endpoint or collector identifier available |
| User identity | Y | `user.name` | Present for authentication and account activity |
| Severity / priority | Partial | `log.level` | Not meaningful for all Windows event IDs |
| Action / message | Y | `event.action`, `event.code`, `message` | Event action and rendered message available |
| Command / process executed | Partial | `process.command_line` | Only present for process creation, PowerShell, or command execution events |
| Impacted object | Partial | `winlog.event_data.TargetUserName` | Only present for certain account or object events |
| Result of activity | Y | `event.outcome`, `event.code` | Success or failure mapped from event ID |
| Session / transaction ID | Partial | `winlog.activity_id` | Only present for some Windows event types |
| Unique event identifier | Y | `event.id`, `winlog.record_id` | Used to identify the individual Windows event record |

## Pass Criteria

This step passes when:

- Required fields are present, mapped, or documented as conditionally populated
- Field names are consistent across sample events of the same log type
- Time fields are available and normalised to UTC
- User, source, action, result, and target fields are available where relevant
- Device or asset identifiers are present where the source provides stable identifiers
- Command or process execution fields are present where the source records execution activity
- Session, transaction, request, or correlation identifiers are present where the source supports multi-event activity tracking
- A unique event identifier is present where the source supports one
- Partial fields have clear notes explaining when they are expected to appear
- Field mappings are suitable for detection, investigation, reporting, correlation, and normalisation

## Fail Criteria

This step fails when:

- Required fields are missing with no acceptable reason
- Time fields are absent, inconsistent, or not normalised to UTC
- User identity is unavailable for authenticated activity
- Source address or source host is missing for events where origin context is required
- Device or asset identifiers are missing where asset correlation is required
- Command or process execution details are missing for log types intended to record execution activity
- Action, message, event type, or result fields are only available inside unparsed free text
- Status codes are present but not extracted, mapped, or interpretable
- Session, transaction, request, or correlation identifiers are missing for log types where event chaining is required
- A unique event identifier is missing where the source provides one or where event-level traceability is required
- Field names change inconsistently between similar events
- Partial fields are not documented with clear caveats
- The event lacks enough context to support the intended SIEM use cases

## Remediation

If required fields are missing or unreliable, review the source configuration, ingestion method, parsing logic, field extraction rules, and normalisation mappings.

Where possible, reconfigure the source system, collector, forwarder, or integration to emit structured fields. If the field exists in the raw event but is not extracted, update the parsing, field extraction, or normalisation configuration.

If a field is only conditionally available, document the condition clearly in the validation record.

If required fields cannot be provided by the source, record the limitation and assess whether the log source still supports the intended security use cases.
