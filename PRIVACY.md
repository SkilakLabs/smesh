# SkilakMesh privacy statement

_Effective 2026-08-01. Version 1.0._

## Public website

The static marketing-site code collects no personal data, uses no analytics or
tracking cookies, and makes no automatic external requests. It contains no
forms, remote fonts, or third-party runtime scripts. A visitor may choose to
open an external link or email link.

The static hosting and network providers that deliver the site may necessarily
process ordinary request information, such as an IP address, timestamp, and
requested path, under their own operational policies. The SkilakMesh site code
does not add analytics or send that information to a SkilakMesh application
backend.

## Self-hosted product

SkilakMesh runs on infrastructure controlled by its operator. The product is
designed to keep request and response payloads out of its audit trail and to
expose bounded aggregates in the local dashboard. Audit metadata and keyed
fingerprints can still be sensitive. The operator controls deployment access,
retention, backups, exporters, and any reverse proxy or support integration.

SkilakMesh has no product telemetry enabled by default. See `README.md` and
`LIMITATIONS.md` for the exact technical boundaries.

## Regulatory disclaimer

SkilakMesh™ is a self-hosted software tool and does not guarantee complete
identification or redaction of Protected Health Information (PHI) or other
regulated data. Skilak LLC is not a HIPAA compliance certification
provider, does not act as a Business Associate, and does not execute Business
Associate Agreements (BAAs) for self-hosted deployments. The operator of the
software assumes sole and absolute responsibility for compliance with HIPAA,
HITECH, and all other applicable data privacy regulations.

Detection of names and other free-form personal data is statistical and
incomplete: the optional person-name detector is measured at roughly 85–92%
recall, and it is off by default. Do not treat SkilakMesh™ as a de-identification
control.

Questions: hello@skilak.ai.

SkilakMesh™ is a trademark of Skilak LLC.
