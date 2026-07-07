# Common Date and Time Formats

The preferred timestamp format for SIEM validation is **ISO 8601 / RFC 3339 in UTC**, with millisecond precision where supported.

Other formats may be observed in source logs, but they should be normalised to UTC during ingestion before being used for SIEM search, detection, alerting, reporting, or event correlation.

| Common Standard / Format Name | Example Value | Splunk `TIME_FORMAT` Example | Description | Validation Notes |
|---|---|---|---|---|
| ISO 8601 UTC with milliseconds | `2026-03-24T00:15:32.451Z` | `%Y-%m-%dT%H:%M:%S.%QZ` | Preferred standard format using UTC time. The `Z` suffix indicates UTC. | Preferred format for normalised SIEM event time |
| ISO 8601 with UTC offset | `2026-03-24T11:15:32.451+1100` | `%Y-%m-%dT%H:%M:%S.%Q%z` | Includes local time plus explicit UTC offset. | Acceptable if reliably converted to UTC during ingestion |
| ISO 8601 with colon UTC offset | `2026-03-24T11:15:32.451+11:00` | `%Y-%m-%dT%H:%M:%S.%Q%:z` | Includes local time plus explicit UTC offset with a colon. | Confirm the Splunk parser supports colon-based offsets in the deployed version/configuration |
| RFC 3339 | `2026-03-24T00:15:32.451Z` | `%Y-%m-%dT%H:%M:%S.%QZ` | Internet profile of ISO 8601 commonly used in APIs, JSON logs, and cloud services. | Preferred for API and cloud log sources |
| RFC 3164 syslog timestamp | `Mar 24 00:15:32` | `%b %d %H:%M:%S` | Traditional BSD syslog timestamp without year or timezone. | Not preferred unless year and timezone are added or reliably inferred |
| RFC 5424 syslog timestamp | `2026-03-24T00:15:32.451Z` | `%Y-%m-%dT%H:%M:%S.%QZ` | Structured syslog timestamp format with ISO 8601-style timestamp. | Preferred over RFC 3164 for syslog sources |
| Unix epoch time, seconds | `1774311332` | `%s` | Number of seconds since `1970-01-01 00:00:00 UTC`. | Acceptable if correctly converted to UTC timestamp |
| Unix epoch time, milliseconds | `1774311332451` | `%s%Q` | Number of milliseconds since `1970-01-01 00:00:00 UTC`. | Acceptable where millisecond precision is required |
| Windows FILETIME | `133871625324510000` | Custom conversion required | Number of 100-nanosecond intervals since `1601-01-01 UTC`. | Must be converted before use in SIEM searches or correlation |
| Windows Event Log timestamp | `2026-03-24T00:15:32.4510000Z` | `%Y-%m-%dT%H:%M:%S.%QZ` | Timestamp commonly seen in Windows Event XML. | Acceptable if normalised and mapped to the correct SIEM event time |
| Apache / NGINX access log time | `24/Mar/2026:00:15:32 +0000` | `%d/%b/%Y:%H:%M:%S %z` | Common web server access log timestamp format. | Acceptable if the timezone offset is present and parsed correctly |
| Database timestamp | `2026-03-24 00:15:32.451` | `%Y-%m-%d %H:%M:%S.%Q` | Common SQL-style timestamp format. | Must include or infer timezone before normalisation |
| Local time without timezone | `2026-03-24 11:15:32` | `%Y-%m-%d %H:%M:%S` | Timestamp without UTC offset or timezone context. | Not preferred; requires documented source timezone |
| Date-only value | `2026-03-24` | `%Y-%m-%d` | Contains date but no time component. | Not suitable as the primary event timestamp for event correlation |
| Time-only value | `00:15:32.451` | `%H:%M:%S.%Q` | Contains time but no date component. | Not suitable unless the event date is available elsewhere |
| Vendor-specific timestamp | `Tue Mar 24 00:15:32 UTC 2026` | `%a %b %d %H:%M:%S %Z %Y` | Custom timestamp format produced by a vendor, platform, or application. | Acceptable only if parsing is reliable and documented |

## Timestamp Format Preference

Use the following preference order when validating timestamp handling:

| Preference | Format Type | Guidance |
|---:|---|---|
| 1 | ISO 8601 / RFC 3339 UTC with milliseconds | Preferred format |
| 2 | ISO 8601 / RFC 3339 with UTC offset | Acceptable if normalised to UTC |
| 3 | RFC 5424 syslog timestamp | Acceptable for structured syslog |
| 4 | Epoch timestamp in seconds or milliseconds | Acceptable if correctly converted |
| 5 | Vendor-specific timestamp with timezone | Acceptable if parsing is reliable |
| 6 | Local timestamp without timezone | Use only where the source timezone is known and documented |
| 7 | Date-only or time-only values | Not acceptable as the primary event timestamp |

## Splunk Timestamp Parsing Notes

When documenting timestamp handling for a log source, include the Splunk `TIME_FORMAT` string used or expected for parsing.

Common Splunk date and time format tokens include:

| Token | Meaning | Example |
|---|---|---|
| `%Y` | Four-digit year | `2026` |
| `%m` | Two-digit month | `03` |
| `%d` | Two-digit day | `24` |
| `%H` | Hour in 24-hour format | `00` |
| `%M` | Minute | `15` |
| `%S` | Second | `32` |
| `%Q` | Fractional seconds / milliseconds, depending on parsing configuration | `451` |
| `%z` | Numeric UTC offset | `+0000`, `+1100` |
| `%:z` | Numeric UTC offset with colon | `+00:00`, `+11:00` |
| `%Z` | Timezone name | `UTC` |
| `%b` | Abbreviated month name | `Mar` |
| `%a` | Abbreviated weekday name | `Tue` |
| `%s` | Unix epoch seconds | `1774311332` |

Preferred Splunk `TIME_FORMAT` for ISO 8601 UTC with milliseconds:

```text
%Y-%m-%dT%H:%M:%S.%QZ
```

For sources with a numeric UTC offset:

```text
%Y-%m-%dT%H:%M:%S.%Q%z
```

For sources with a numeric UTC offset that includes a colon:

```text
%Y-%m-%dT%H:%M:%S.%Q%:z
```

Depending on the Splunk version and timestamp parsing approach, fractional seconds may also be represented using `%N`, such as `%3N` for milliseconds. The validation record should document the actual format used in the source configuration.
