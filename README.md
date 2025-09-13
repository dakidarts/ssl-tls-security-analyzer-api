# 🔐 SSL/TLS Security Analyzer API

The SSL/TLS Security Analyzer API is a lightweight and developer-friendly tool for analyzing SSL/TLS configurations of domains. It provides grading from A–F, detects weak ciphers, checks supported protocols, validates certificates, and highlights common vulnerabilities. Whether you’re building a security dashboard, monitoring system, or compliance tool, this API makes SSL/TLS checks seamless.

![SSL/TLS Security Analyzer API](https://res.cloudinary.com/ds64xs2lp/image/upload/v1757796548/ssl-analyzer_api_eo6b5f.gif)

* [Get Started on RapidAPI](https://rapidapi.com/dakidarts-dakidarts-default/api/ssl-tls-security-analyzer-api)
* [API Docs on Dakidarts](https://dakidarts.com/api/ssl-tls-security-analyzer-api/)

## Base URL

```
ssl-tls-security-analyzer-api.p.rapidapi.com
```

---

## **Endpoint: `/analyze`**

### Methods

* `GET`
* `POST`

### Description

Analyze a given domain’s SSL/TLS configuration.
Input: domain/host (and optional port/timeout).
Output: SSL grade, TLS versions, weak ciphers, certificate details, and vulnerability notes.

---

### **Query Parameters (GET)**

| Parameter | Type   | Required | Default | Description                                      |
| --------- | ------ | -------- | ------- | ------------------------------------------------ |
| `domain`  | string | ✅ Yes    | —       | Target domain or hostname (e.g., `example.com`). |
| `host`    | string | ❌ No    | —       | Alias for `domain`.                              |
| `port`    | int    | ❌ No     | 443     | Port to connect to.                              |
| `timeout` | int    | ❌ No     | 5       | Connection timeout in seconds.                   |

**Example GET Request**

```
/analyze?domain=example.com
```

---

### **Request Body (POST)**

```json
{
  "domain": "example.com",
  "port": 443,
  "timeout": 5
}
```

* `domain` (or `host`) – required
* `port` – optional, default `443`
* `timeout` – optional, default `5`

---

### **Response (200 OK)**

```json
{
  "domain": "example.com",
  "port": 443,
  "ssl_grade": "B",
  "tls_versions": ["TLSv1.2", "TLSv1.3"],
  "cipher_list": [
    "AES128-SHA",
    "AES256-SHA256"
  ],
  "weak_ciphers": [
    "AES128-SHA"
  ],
  "certificate": {
    "subject": "CN=example.com",
    "issuer": "CN=Let's Encrypt Authority X3, O=Let's Encrypt, C=US",
    "valid_from": "2025-08-01T12:00:00",
    "valid_to": "2025-10-30T12:00:00",
    "expired": false,
    "days_until_expiry": 54
  },
  "vulnerabilities": [
    "Legacy TLS 1.0/1.1 supported",
    "Weak ciphers: AES128-SHA",
    "Heartbleed: Deep Heartbleed checks not performed by default."
  ]
}
```

---

### **Error Responses**

| Code  | Example                                                               | Meaning                           |
| ----- | --------------------------------------------------------------------- | --------------------------------- |
| `400` | `{"error": "domain parameter is required (e.g. domain=example.com)"}` | Missing required parameter.       |
| `500` | `{"error": "internal error", "details": "traceback..."}`              | Unexpected internal server error. |

---

### **Grading Logic (A–F)**

Grades are based on protocol support, cipher strength, and certificate validity:

* **A** – Only modern TLS (1.2/1.3), strong ciphers, valid cert.
* **B** – Minor issues (e.g., TLS 1.0 support, weak cipher present).
* **C** – Legacy protocols/ciphers allowed, but not SSLv3.
* **D** – SSLv3 or multiple weak ciphers supported.
* **F** – Expired certs, critical misconfigurations, or only legacy SSL/TLS.

---

### **Notes**

* POODLE flagged if SSLv3 is enabled.
* Weak Diffie-Hellman flagged if DH params &lt; 2048 bits.
* Output JSON is always structured and safe for integration.

