# 🔐 SSL/TLS Security Analyzer API v0.0.2

The SSL/TLS Security Analyzer API is a lightweight, developer-friendly REST API for analyzing SSL/TLS configurations of any domain or IP address. It provides security grading from A–F, detects weak ciphers and weak Diffie-Hellman parameters, checks supported protocols, validates certificates and certificate chains, enumerates cipher suites, monitors certificate expiry, and flags common vulnerabilities — all from a single HTTP request.

Whether you're building a security dashboard, monitoring system, compliance tool, or a certificate-expiry alerting pipeline, this API makes SSL/TLS checks seamless.

![SSL/TLS Security Analyzer API](https://res.cloudinary.com/ds64xs2lp/image/upload/v1757796548/ssl-analyzer_api_eo6b5f.gif)

---

## What's New in v0.0.2

- **5 new endpoints** — `/certificate`, `/expiry`, `/batch`, `/ciphers`, `/cipher/check`
- **Enhanced grading** — score-based A–F with per-rule notes, hostname validation, chain validation, forward secrecy, DROWN detection
- **Faster scans** — protocol probes now run in parallel (~30% speed improvement)
- **Full certificate chain** — SANs, key algorithm/size, serial number, signature algorithm, self-signed detection
- **Monitoring-friendly** — `/expiry` for cron-based cert monitoring, `/batch` for bulk scanning up to 50 domains
- **Security hardened** — input injection removed, strict hostname/port/cipher whitelisting, in-memory caching

---

## Base URL

```
ssl-tls-security-analyzer-api.p.rapidapi.com
```

---

## Quick Start

```bash
# Minimal scan
curl "https://ssl-tls-security-analyzer-api.p.rapidapi.com/analyze?domain=example.com"

# Check certificate expiry
curl "https://ssl-tls-security-analyzer-api.p.rapidapi.com/expiry?domain=example.com"

# Scan multiple domains at once
curl -X POST "https://ssl-tls-security-analyzer-api.p.rapidapi.com/batch" \
  -H "Content-Type: application/json" \
  -d '{"domains": ["example.com", "google.com", "github.com"]}'
```

---

## Endpoints Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/status` | Health check and service info |
| GET/POST | `/analyze` | Full SSL/TLS security analysis (flagship) |
| GET/POST | `/certificate` | Deep certificate inspection and chain validation |
| GET | `/expiry` | Fast certificate expiry check for monitoring |
| GET/POST | `/batch` | Parallel analysis of up to 50 domains |
| GET | `/ciphers` | Enumerate supported cipher suites for TLS 1.2/1.3 |
| GET | `/cipher/check` | Test whether a specific cipher suite is supported |

---

## Common Parameters

These parameters are shared across most endpoints:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | ✅ Yes | — | Target hostname or IP (e.g., `example.com`, `1.2.3.4`). Also accepts `host` as alias. |
| `port` | int | ❌ No | `443` | TCP port to connect to. Range: 1–65535. |
| `timeout` | int | ❌ No | `5` | Per-connection timeout in seconds. Range: 1–15. |

All endpoints return JSON. Errors include an `error` message and a machine-readable `code` field.

---

## Endpoint 1: GET /status

### Description

Health check endpoint. Returns the service status, version, whether openssl is available, the Python runtime version, server time, and uptime. Makes zero network calls — instant and free.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| *(none)* | — | — | — | This endpoint requires no parameters. |

### Example Request

```
GET /status
```

### Response (200 OK)

```json
{
  "status": "ok",
  "service": "SSL/TLS Security Analyzer API",
  "version": "0.0.2",
  "uptime_seconds": 42,
  "openssl_available": true,
  "python_version": "3.14.5",
  "server_time": "2026-08-26T10:00:00.000000+00:00"
}
```

---

## Endpoint 2: GET/POST /analyze

### Description

**Flagship endpoint.** Performs a comprehensive SSL/TLS security analysis of any hostname or IP address. Returns supported TLS protocol versions, all negotiated cipher suites, weak cipher detection, full parsed certificate details, certificate chain validation, hostname match verification, forward secrecy check, weak Diffie-Hellman detection, known vulnerability flags, and an overall letter grade from A to F. The grade uses a score-based heuristic with a transparent breakdown of every deduction in `grade_notes`.

### Query Parameters (GET)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | ✅ Yes | — | Target domain or hostname (e.g., `example.com`). |
| `host` | string | ❌ No | — | Alias for `domain`. |
| `port` | int | ❌ No | `443` | Port to connect to. |
| `timeout` | int | ❌ No | `5` | Connection timeout in seconds. |

### Request Body (POST)

```json
{
  "domain": "example.com",
  "port": 443,
  "timeout": 5
}
```

### Example GET Request

```
/analyze?domain=example.com
```

### Example POST Request

```
POST /analyze
Content-Type: application/json

{
  "domain": "example.com",
  "port": 443,
  "timeout": 5
}
```

### Response (200 OK)

```json
{
  "domain": "example.com",
  "port": 443,
  "ssl_grade": "A",
  "grade_notes": [],
  "tls_versions": ["TLSv1.2"],
  "protocols": ["TLSv1.2"],
  "best_protocol": "TLSv1.2",
  "negotiated_cipher": "ECDHE-RSA-AES128-GCM-SHA256",
  "forward_secrecy": true,
  "cipher_list": [
    "ECDHE-RSA-AES128-GCM-SHA256",
    "ECDHE-RSA-AES256-GCM-SHA384"
  ],
  "weak_ciphers": [],
  "weak_dh": {
    "weak": false,
    "bits": null
  },
  "certificate": {
    "subject": {
      "commonName": "example.com",
      "organizationName": "..."
    },
    "issuer": {
      "commonName": "Cloudflare TLS Issuing ECC CA 3",
      "organizationName": "SSL Corporation",
      "countryName": "US"
    },
    "common_name": "example.com",
    "serial_number": "0624D0AB311558780B7D...",
    "version": 3,
    "not_before": "2026-07-29T22:10:08+00:00",
    "not_after": "2026-10-27T22:17:21+00:00",
    "expired": false,
    "days_until_expiry": 79,
    "signature_algorithm": "ecdsa-with-SHA256",
    "subject_alt_names": [
      { "type": "DNS", "value": "example.com" }
    ],
    "is_ca": false,
    "has_server_auth": true,
    "has_client_auth": null
  },
  "certificate_key": {
    "algorithm": "id-ecPublicKey",
    "bits": 256,
    "serial_number": "0624D0AB311558780B7D..."
  },
  "chain": {
    "length": 3,
    "signing_order_valid": true,
    "signing_order_note": null,
    "self_signed": false
  },
  "hostname_valid": true,
  "vulnerabilities": [
    "Deep vulnerability checks (Heartbleed etc.) are not performed; use a dedicated scanner for protocol-specific exploits"
  ],
  "probe_notes": [],
  "scanned_at": "2026-08-26T10:00:00.000000+00:00",
  "duration_ms": 2350
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `domain` | Target hostname scanned |
| `port` | Target port scanned |
| `ssl_grade` | Overall grade A–F (see grading logic below) |
| `grade_notes` | List of human-readable reasons for the grade |
| `tls_versions` / `protocols` | Supported TLS protocol versions |
| `best_protocol` | Most modern supported protocol |
| `negotiated_cipher` | Cipher suite negotiated in a default handshake |
| `forward_secrecy` | `true` if any cipher uses ECDHE or DHE key exchange |
| `cipher_list` | All cipher suites seen during probing |
| `weak_ciphers` | Ciphers matching known-weak markers (RC4, DES, EXPORT, NULL, MD5, ADH) |
| `weak_dh` | Diffie-Hellman parameters — `weak: true` if below 2048 bits |
| `certificate` | Parsed leaf certificate: subject, issuer, SANs, validity dates, expiry |
| `certificate_key` | Public key algorithm and bit size |
| `chain` | Certificate chain length, signing-order validation, self-signed detection |
| `hostname_valid` | `true`/`false`/`null` — whether the certificate covers the hostname via SANs |
| `vulnerabilities` | Detected issues (POODLE, DROWN, legacy TLS, weak ciphers, expiring cert, etc.) |
| `probe_notes` | Notes about probe limitations (e.g. openssl not available) |
| `duration_ms` | Scan duration in milliseconds |
| `scanned_at` | ISO 8601 timestamp of the scan |

### Grading Logic (A–F)

Grades are scored from 100 with deductions for each weakness. The `grade_notes` array contains every deduction applied:

| Rule | Deduction |
|------|-----------|
| Expired certificate | −100 (auto grade F) |
| SSLv2 enabled (DROWN) | −45 |
| SSLv3 enabled (POODLE) | −40 |
| No TLS 1.2/1.3 supported | −40 |
| Broken certificate chain | −25 |
| TLSv1.0 enabled | −20 |
| Hostname mismatch on certificate | −20 |
| Weak cipher suites configured | −15 |
| Weak Diffie-Hellman parameters | −15 |
| Certificate expires within 14 days | −15 |
| TLSv1.1 enabled | −10 |
| Certificate expires within 30 days | −5 |

| Score | Grade |
|-------|-------|
| ≥ 90 | A |
| ≥ 75 | B |
| ≥ 60 | C |
| ≥ 45 | D |
| ≥ 30 | E |
| < 30 | F |

### Error Responses

| Code | Example | Meaning |
|------|---------|---------|
| 400 | `{"error": "domain parameter is required", "code": "INVALID_DOMAIN"}` | Missing or invalid domain parameter. |
| 400 | `{"error": "port must be an integer between 1 and 65535", "code": "INVALID_PORT"}` | Invalid port value. |
| 502 | `{"error": "analysis failed: ...", "code": "SCAN_FAILED"}` | Target unreachable or TLS handshake failed. |
| 500 | `{"error": "internal server error", "code": "INTERNAL_ERROR"}` | Unexpected server error. |

---

## Endpoint 3: GET/POST /certificate

### Description

Deep inspection of a server's SSL/TLS certificate and full certificate chain. Returns the complete leaf certificate (subject, issuer, common name, serial number, validity dates, expiry status, signature algorithm, Subject Alternative Names, key usage flags), the public key details (algorithm, bit size), the full certificate chain with chain-length count and a heuristic signing-order validation, self-signed detection, and hostname coverage verification (DNS SANs including wildcard support). This endpoint performs one handshake only — no protocol probing — making it the fastest way to inspect certificate health.

### Query Parameters (GET)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | ✅ Yes | — | Target domain or hostname. |
| `port` | int | ❌ No | `443` | Port to connect to. |
| `timeout` | int | ❌ No | `5` | Connection timeout in seconds. |

### Request Body (POST)

```json
{
  "domain": "example.com",
  "port": 443
}
```

### Example GET Request

```
/certificate?domain=example.com
```

### Example POST Request

```
POST /certificate
Content-Type: application/json

{
  "domain": "example.com",
  "port": 443
}
```

### Response (200 OK)

```json
{
  "domain": "example.com",
  "port": 443,
  "certificate": {
    "subject": {
      "commonName": "example.com",
      "organizationName": "..."
    },
    "issuer": {
      "commonName": "Cloudflare TLS Issuing ECC CA 3",
      "organizationName": "SSL Corporation",
      "countryName": "US"
    },
    "common_name": "example.com",
    "serial_number": "0624D0AB311558780B7D...",
    "version": 3,
    "not_before": "2026-07-29T22:10:08+00:00",
    "not_after": "2026-10-27T22:17:21+00:00",
    "expired": false,
    "days_until_expiry": 79,
    "signature_algorithm": "ecdsa-with-SHA256",
    "subject_alt_names": [
      { "type": "DNS", "value": "example.com" }
    ],
    "is_ca": false,
    "has_server_auth": true,
    "has_client_auth": null
  },
  "certificate_key": {
    "algorithm": "id-ecPublicKey",
    "bits": 256,
    "serial_number": "0624D0AB311558780B7D..."
  },
  "chain": {
    "length": 3,
    "signing_order_valid": true,
    "signing_order_note": null,
    "self_signed": false
  },
  "hostname_valid": true,
  "fetched_at": "2026-08-26T10:00:00.000000+00:00"
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `certificate.subject` | Certificate subject fields (commonName, organizationName, etc.) |
| `certificate.issuer` | Certificate issuer fields |
| `certificate.common_name` | Convenience alias for the commonName |
| `certificate.serial_number` | Certificate serial number in hex |
| `certificate.version` | X.509 certificate version |
| `certificate.not_before` | Certificate validity start (ISO 8601) |
| `certificate.not_after` | Certificate validity end (ISO 8601) |
| `certificate.expired` | `true` if current time is past not_after |
| `certificate.days_until_expiry` | Number of days until expiry (negative if expired) |
| `certificate.signature_algorithm` | Signature algorithm (e.g. `ecdsa-with-SHA256`, `sha256WithRSAEncryption`) |
| `certificate.subject_alt_names` | SANs array — each entry has `type` (DNS/IP Address) and `value` |
| `certificate.is_ca` | Whether this is a CA certificate |
| `certificate.has_server_auth` | Key usage: server authentication |
| `certificate.has_client_auth` | Key usage: client authentication |
| `certificate_key.algorithm` | Public key algorithm (e.g. `RSA`, `id-ecPublicKey`, `Ed25519`) |
| `certificate_key.bits` | Public key bit size (e.g. 2048, 256, 4096) |
| `certificate_key.serial_number` | Duplicate of leaf serial (from openssl x509) |
| `chain.length` | Number of certificates in the chain |
| `chain.signing_order_valid` | `true` if each cert's issuer matches the next cert's subject |
| `chain.signing_order_note` | Human-readable reason if signing_order_valid is false |
| `chain.self_signed` | `true` if the leaf is self-signed (single cert, subject == issuer) |
| `hostname_valid` | `true`/`false`/`null` — whether the certificate covers the requested hostname |
| `fetched_at` | ISO 8601 timestamp |

### Error Responses

| Code | Example | Meaning |
|------|---------|---------|
| 400 | `{"error": "domain parameter is required", "code": "INVALID_DOMAIN"}` | Missing or invalid domain parameter. |
| 502 | `{"error": "certificate fetch failed: ...", "code": "SCAN_FAILED"}` | Could not retrieve the certificate (no cert returned, connection refused). |
| 500 | `{"error": "internal server error", "code": "INTERNAL_ERROR"}` | Unexpected server error. |

---

## Endpoint 4: GET /expiry

### Description

Fast, single-handshake certificate expiry check designed for monitoring crons and certificate-expiry alerting services. Returns only the fields relevant to expiry monitoring: domain, common name, issuer, serial number, validity dates, days until expiry, and expired status. This is the cheapest and lightest endpoint in the entire API — one Python SSL handshake, no openssl probes, no protocol scanning. Results are cached for only 2 minutes (vs 5 minutes for other endpoints) so monitoring data stays fresh. Ideal for running on a daily or hourly cron schedule.

### Query Parameters (GET)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | ✅ Yes | — | Target domain or hostname. |
| `port` | int | ❌ No | `443` | Port to connect to. |
| `timeout` | int | ❌ No | `5` | Connection timeout in seconds. |

### Example GET Request

```
/expiry?domain=example.com
```

### Response (200 OK)

```json
{
  "domain": "example.com",
  "port": 443,
  "common_name": "example.com",
  "issuer": "SSL Corporation",
  "serial_number": "0624D0AB311558780B7D...",
  "not_before": "2026-07-29T22:10:08+00:00",
  "not_after": "2026-10-27T22:17:21+00:00",
  "days_until_expiry": 79,
  "expired": false,
  "checked_at": "2026-08-26T10:00:00.000000+00:00"
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `domain` | Target hostname scanned |
| `port` | Target port scanned |
| `common_name` | Certificate common name |
| `issuer` | Certificate issuer organization name |
| `serial_number` | Certificate serial number |
| `not_before` | Certificate validity start (ISO 8601) |
| `not_after` | Certificate validity end (ISO 8601) |
| `days_until_expiry` | Days until expiry (negative if expired) |
| `expired` | `true` if expired |
| `checked_at` | ISO 8601 timestamp of the check |

### Error Responses

| Code | Example | Meaning |
|------|---------|---------|
| 400 | `{"error": "domain parameter is required", "code": "INVALID_DOMAIN"}` | Missing or invalid domain parameter. |
| 502 | `{"error": "expiry check failed: ...", "code": "SCAN_FAILED"}` | Could not retrieve the certificate. |
| 500 | `{"error": "internal server error", "code": "INTERNAL_ERROR"}` | Unexpected server error. |

---

## Endpoint 5: GET/POST /batch

### Description

Analyze up to 50 domains in parallel with a single API request. Each domain is scanned concurrently across 8 parallel workers. If a domain fails input validation (e.g. invalid hostname), it is returned as an error entry in the results — a single bad domain never fails the whole batch. Results are returned in the same order you submitted the domains. Each result in the array is a complete `/analyze` output (grade, protocols, ciphers, certificate, vulnerabilities, etc.) for that domain. This endpoint is ideal for bulk scanning, portfolio monitoring, and CI/CD certificate checks.

### Query Parameters (GET)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domains` | string[] | ✅ Yes | — | List of domains to scan. Repeat the parameter for multiple values. Max 50. |
| `port` | int | ❌ No | `443` | Port applied to all domains in the batch. |
| `timeout` | int | ❌ No | `5` | Timeout applied to all domains in the batch. |

### Request Body (POST)

```json
{
  "domains": ["example.com", "google.com", "github.com"],
  "port": 443,
  "timeout": 5
}
```

- `domains` — required, array of strings, max 50
- `port` — optional, default `443`
- `timeout` — optional, default `5`

### Example GET Request

```
/batch?domains=example.com&domains=google.com&domains=github.com
```

### Example POST Request

```
POST /batch
Content-Type: application/json

{
  "domains": ["example.com", "google.com", "github.com"],
  "port": 443,
  "timeout": 5
}
```

### Response (200 OK)

```json
{
  "total": 3,
  "successful": 2,
  "failed": 1,
  "results": [
    {
      "domain": "example.com",
      "port": 443,
      "ssl_grade": "A",
      "grade_notes": [],
      "tls_versions": ["TLSv1.2"],
      "protocols": ["TLSv1.2"],
      "best_protocol": "TLSv1.2",
      "forward_secrecy": true,
      "cipher_list": ["ECDHE-RSA-AES128-GCM-SHA256"],
      "weak_ciphers": [],
      "certificate": { "common_name": "example.com", "expired": false, "days_until_expiry": 79 },
      "vulnerabilities": [],
      "scanned_at": "2026-08-26T10:00:00.000000+00:00",
      "duration_ms": 2350
    },
    {
      "domain": "google.com",
      "port": 443,
      "ssl_grade": "A",
      "tls_versions": ["TLSv1.2", "TLSv1.3"],
      "best_protocol": "TLSv1.3",
      "forward_secrecy": true,
      "scanned_at": "2026-08-26T10:00:00.000000+00:00",
      "duration_ms": 1800
    },
    {
      "domain": "not a host!!",
      "error": "invalid domain or IP address",
      "scanned_at": "2026-08-26T10:00:00.000000+00:00"
    }
  ]
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `total` | Number of domains in the results array |
| `successful` | Number of domains scanned without error |
| `failed` | Number of domains that failed (invalid input or scan error) |
| `results` | Array of per-domain results, in request order. Each entry is either a full `/analyze` result or an error object with `domain` and `error` fields. |

### Error Responses

| Code | Example | Meaning |
|------|---------|---------|
| 400 | `{"error": "domains parameter is required", "code": "INVALID_DOMAINS"}` | Empty domains list. |
| 400 | `{"error": "batch limited to 50 domains per request", "code": "TOO_MANY_DOMAINS"}` | More than 50 domains submitted. |
| 400 | `{"error": "port must be an integer between 1 and 65535", "code": "INVALID_PORT"}` | Invalid port value. |
| 500 | `{"error": "internal server error", "code": "INTERNAL_ERROR"}` | Unexpected server error. |

---

## Endpoint 6: GET /ciphers

### Description

Enumerate all cipher suites a server supports for a given TLS version (TLS 1.2 or TLS 1.3) by testing every locally available cipher suite in parallel. The API builds a list of cipher suite names from the local openssl installation, then attempts a handshake for each one (up to `limit` attempts) using a thread pool of 8 parallel workers. Only suites the server actually accepts are returned in the `supported` array. This is the most resource-intensive endpoint — it makes dozens of TLS handshakes per call. Requires the openssl CLI on the server.

### Query Parameters (GET)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | ✅ Yes | — | Target domain or hostname. |
| `tls_version` | string | ❌ No | `TLSv1.2` | Which TLS version to test suites for. Accepts `TLSv1.2` or `TLSv1.3`. |
| `limit` | int | ❌ No | `100` | Maximum number of suites to test. Range: 1–250. |
| `port` | int | ❌ No | `443` | Port to connect to. |
| `timeout` | int | ❌ No | `5` | Per-connection timeout in seconds. |

### Example GET Request

```
/ciphers?domain=example.com&tls_version=TLSv1.2&limit=50
```

### Response (200 OK)

```json
{
  "tls_version": "TLSv1.2",
  "tested": 50,
  "supported_count": 2,
  "supported": [
    "ECDHE-RSA-AES256-GCM-SHA384",
    "ECDHE-RSA-CHACHA20-POLY1305"
  ],
  "notes": []
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `tls_version` | The TLS version tested |
| `tested` | Number of cipher suites attempted |
| `supported_count` | Number of suites the server accepted |
| `supported` | Array of supported cipher suite names |
| `notes` | Any warnings (e.g. openssl rejected a suite name) |

### Notes

- For TLS 1.3, use `tls_version=TLSv1.3`. Suites use names like `TLS_AES_256_GCM_SHA384` and are tested via the `-ciphersuites` flag.
- For TLS 1.2, suites use names like `ECDHE-RSA-AES256-GCM-SHA384` and are tested via the `-cipher` flag.
- Higher `limit` values are more thorough but take longer. Each handshake takes roughly 0.1–0.4 seconds.

### Error Responses

| Code | Example | Meaning |
|------|---------|---------|
| 400 | `{"error": "domain parameter is required", "code": "INVALID_DOMAIN"}` | Missing or invalid domain parameter. |
| 400 | `{"error": "tls_version must be TLSv1.2 or TLSv1.3", "code": "INVALID_TLS_VERSION"}` | Invalid TLS version. |
| 400 | `{"error": "limit must be between 1 and 250", "code": "INVALID_LIMIT"}` | Invalid limit value. |
| 503 | `{"error": "openssl is not installed on the server", "code": "OPENSSL_MISSING"}` | openssl CLI not available on the server. |
| 502 | `{"error": "cipher enumeration failed: ...", "code": "SCAN_FAILED"}` | Connection to target failed. |

---

## Endpoint 7: GET /cipher/check

### Description

Test whether a server supports one specific cipher suite with a single TLS handshake. Returns `supported: true` if the server negotiated the cipher, or `supported: false` if it rejected it. This is a fast, lightweight endpoint — one handshake, one result. Useful for verifying compliance with specific security standards, checking whether a particular cipher is available on a server, answering pinning questions, and debugging TLS handshake failures. Works for both TLS 1.2 and TLS 1.3 suites.

### Query Parameters (GET)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | ✅ Yes | — | Target domain or hostname. |
| `cipher` | string | ✅ Yes | — | Cipher suite name to test. Must contain only letters, numbers, underscores, and hyphens. |
| `tls_version` | string | ❌ No | `TLSv1.2` | TLS version to test on. Must match the cipher's protocol. `TLSv1.2` or `TLSv1.3`. |
| `port` | int | ❌ No | `443` | Port to connect to. |
| `timeout` | int | ❌ No | `5` | Per-connection timeout in seconds. |

### Example GET Request (TLS 1.2)

```
/cipher/check?domain=example.com&cipher=ECDHE-RSA-AES256-GCM-SHA384
```

### Example GET Request (TLS 1.3)

```
/cipher/check?domain=example.com&cipher=TLS_AES_256_GCM_SHA384&tls_version=TLSv1.3
```

### Response — Supported (200 OK)

```json
{
  "cipher": "ECDHE-RSA-AES256-GCM-SHA384",
  "tls_version": "TLSv1.2",
  "supported": true,
  "note": null
}
```

### Response — Not Supported (200 OK)

```json
{
  "cipher": "AES128-SHA",
  "tls_version": "TLSv1.2",
  "supported": false,
  "note": null
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `cipher` | The cipher suite name that was tested |
| `tls_version` | The TLS version tested against |
| `supported` | `true` if the server accepted and negotiated the cipher, `false` if rejected |
| `note` | Any warning (e.g. if openssl could not parse the cipher name) |

### Common Cipher Suite Names

**TLS 1.2 suites:**
- `ECDHE-RSA-AES256-GCM-SHA384` — strong, forward secrecy
- `ECDHE-RSA-AES128-GCM-SHA256` — strong, forward secrecy
- `ECDHE-RSA-CHACHA20-POLY1305` — strong, modern
- `AES256-SHA256` — strong but no forward secrecy
- `AES128-SHA` — weak, no forward secrecy

**TLS 1.3 suites:**
- `TLS_AES_256_GCM_SHA384` — strongest TLS 1.3
- `TLS_CHACHA20_POLY1305_SHA256` — strong, mobile-friendly
- `TLS_AES_128_GCM_SHA256` — strong, standard TLS 1.3

### Error Responses

| Code | Example | Meaning |
|------|---------|---------|
| 400 | `{"error": "domain parameter is required", "code": "INVALID_DOMAIN"}` | Missing or invalid domain parameter. |
| 400 | `{"error": "cipher parameter is required", "code": "INVALID_CIPHER"}` | Missing or invalid cipher name. |
| 400 | `{"error": "tls_version must be TLSv1.2 or TLSv1.3", "code": "INVALID_TLS_VERSION"}` | Invalid TLS version. |
| 503 | `{"error": "openssl is not installed on the server", "code": "OPENSSL_MISSING"}` | openssl CLI not available. |
| 502 | `{"error": "cipher check failed: ...", "code": "SCAN_FAILED"}` | Connection to target failed. |

---

## Error Handling

All non-200 responses are JSON with a stable, machine-readable `code` so you can handle errors programmatically:

```json
{
  "error": "human-readable message explaining what went wrong",
  "code": "MACHINE_READABLE_CODE"
}
```

### Error Code Reference

| HTTP | Code | Meaning |
|------|------|---------|
| 400 | `INVALID_DOMAIN` | domain parameter missing, empty, or not a valid hostname/IP |
| 400 | `INVALID_PORT` | port is not an integer or outside 1–65535 |
| 400 | `INVALID_TIMEOUT` | timeout is not an integer or outside 1–15 |
| 400 | `INVALID_TLS_VERSION` | tls_version is not TLSv1.2 or TLSv1.3 |
| 400 | `INVALID_LIMIT` | limit is not an integer or outside 1–250 |
| 400 | `INVALID_CIPHER` | cipher parameter missing or contains disallowed characters |
| 400 | `INVALID_DOMAINS` | batch endpoint called with an empty domains list |
| 400 | `TOO_MANY_DOMAINS` | batch request contains more than 50 domains |
| 404 | `NOT_FOUND` | Unknown route |
| 405 | `METHOD_NOT_ALLOWED` | Wrong HTTP method for this endpoint |
| 500 | `INTERNAL_ERROR` | Unexpected server error |
| 502 | `SCAN_FAILED` | Target unreachable, no certificate returned, or TLS handshake failed |
| 503 | `OPENSSL_MISSING` | openssl CLI not installed on the server (affects /ciphers and /cipher/check) |

---

## Changelog

### v0.0.2 — August 26, 2026

**New endpoints:**
- `GET|POST /certificate` — Full certificate inspection: leaf details, key algorithm/size, SANs, full chain, signing-order validation, self-signed detection, hostname coverage check
- `GET /expiry` — Fast single-handshake certificate expiry check for monitoring crons (shorter 2-minute cache)
- `GET|POST /batch` — Parallel analysis of up to 50 domains per request with per-domain error isolation
- `GET /ciphers` — Enumerate supported cipher suites for TLS 1.2/1.3 by parallel suite testing
- `GET /cipher/check` — Single-cipher support test for TLS 1.2 and TLS 1.3

**Enhancements to /analyze:**
- Score-based grading with transparent `grade_notes` breakdown
- New fields: `best_protocol`, `negotiated_cipher`, `forward_secrecy`, `weak_dh`, `certificate_key`, `chain`, `hostname_valid`, `probe_notes`, `duration_ms`, `scanned_at`
- Protocol probes now run in parallel (~30% faster scans)
- DROWN (SSLv2) detection added
- Certificate chain validation and self-signed detection

**Security:**
- Input injection removed — all subprocess calls use argument lists, never shell strings
- Strict hostname, port, timeout, and cipher name validation
- In-memory TTL cache with configurable expiry

### v0.0.1 — September 6, 2025

- Initial release
- `GET|POST /analyze` — SSL/TLS analysis with grade, protocols, weak ciphers, certificate info, vulnerability heuristics
- `GET /status` — Health check

---

## Support

For questions, bug reports, or feature requests, contact the API provider through RapidAPI.
