# Step 1: Validate Log Format

## Purpose

Confirm that the incoming logs are arriving in an expected and supportable format, such as Common Event Format (CEF), structured JSON, Windows Event XML, CSV, TSV, or another agreed structured format.

This is the first validation check because the log format determines how reliably fields can be extracted, normalised, searched, and mapped to downstream use cases.

## Validation Method

For each log source and log type, retrieve a representative sample event from the SIEM.

Inspect the raw event and confirm the format being received.

Examples of acceptable formats include:

- Structured JSON
- Common Event Format (CEF)
- Windows Event Log / XML-derived events
- Key-value formatted logs
- Comma-separated values (CSV)
- Tab-separated values (TSV)
- Vendor-supported structured syslog

CSV and TSV logs should only be accepted where the field order, delimiter, headers, and schema are consistent and documented.

Document the observed format in the validation record.

## Example

For an Active Directory log source ingested through Winlogbeat, the expected format is structured JSON.

A sample event should be retrieved from the SIEM and reviewed to confirm that the event is arriving as JSON, with clear field names and values available for parsing and normalisation.

The sample event should then be added to the validation record as supporting evidence.

## Why This Matters

Unstructured syslog-style logs can often be parsed, but the field extraction process is usually more brittle and maintenance-heavy.

Structured formats reduce parsing complexity, improve field consistency, and make the log source easier to support over time.

Enforcing structured log formats during onboarding helps avoid downstream issues with detection logic, dashboards, correlation searches, data models, and long-term maintenance.

## Evidence to Capture

Record the following evidence:

- Log source name
- Product or technology
- Ingestion method
- Expected format
- Observed format
- Sample raw event
- Timestamp of sample event
- Index and sourcetype
- Validation result
- Notes or issues identified

## Pass Criteria

This step passes when:

- Logs are arriving in the expected format
- The format is structured or vendor-supported
- The sample event can be clearly interpreted
- Key fields are visible and consistently represented
- The observed format matches the agreed onboarding design
- CSV or TSV logs have consistent field counts, headers, column order, delimiters, and schema documentation where applicable

## Fail Criteria

This step fails when:

- Logs are arriving in an unexpected format
- Logs are unstructured when a structured format is available
- The event format changes between samples
- The raw event is truncated or malformed
- Required fields are embedded in free text with no reliable extraction pattern
- The format does not support the intended SIEM use cases
- CSV or TSV logs have inconsistent field counts, missing headers, changing column order, or unreliable delimiters

## Remediation

If the log format does not meet requirements, review the source configuration and ingestion path.

Where possible, reconfigure the source system, collector, forwarder, or integration to send logs in a structured format.

If only unstructured logs are available, document the limitation and define the required parsing, field extraction, and maintenance approach before proceeding to later validation steps.
