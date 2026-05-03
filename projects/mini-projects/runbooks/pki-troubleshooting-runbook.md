# PKI Troubleshooting Runbook

---

## Title

OpenSSL Chain Validation Failure on Windows (Error 20: Unable to Get Local Issuer Certificate)

---

## Problem Statement

OpenSSL certificate validation failed on Windows with `verify error:num=20:unable to get local issuer certificate` when running `openssl s_client -connect google.com:443 -verify_return_error`, even though the root CA was present and trusted in the Windows certificate store. Expected result was `Verify return code: 0 (ok)`.

---

## Environment

**Tool(s) used:** OpenSSL 3.6.1

**Certificate type or component involved:** Public TLS certificate with full chain (leaf → intermediate → root), retrieved from google.com

**Any relevant configuration details:** OpenSSL on Windows does not use the Windows certificate store and relies on a separate CA bundle. Commands were run in cmd.exe — OpenSSL was not recognized in PowerShell.

---

## Symptoms

- `verify error:num=20:unable to get local issuer certificate`
- `Verify return code: 20 (unable to get local issuer certificate)`
- `openssl verify leaf_cert.pem` fails at depth 0
- Browser validation of google.com succeeds, but OpenSSL validation fails
- Windows Certificate Manager (certmgr.msc) shows GlobalSign Root CA as trusted

---

## Diagnostic Steps

1. Attempted to validate the certificate chain from google.com:

```cmd
openssl s_client -connect google.com:443 -verify_return_error
```

Returned `verify error:num=20:unable to get local issuer certificate`. Full chain was present in the output, ruling out a missing intermediate.

2. Tried exporting GlobalSign Root CA from certlm and importing it into certmgr (Trusted Root Certification Authorities). Re-ran the command — still returned error 20. Confirmed OpenSSL does not read from the Windows certificate store.

3. Converted the exported root certificate from DER to PEM format:

```cmd
openssl x509 -inform DER -in GlobalSignRoot.cer -out GlobalSignRoot.pem
```

4. Re-ran verification supplying the root certificate explicitly with `-CAfile`:

```cmd
openssl s_client -connect google.com:443 -CAfile GlobalSignRoot.pem -verify_return_error
```

Connection teardown messages were present but did not affect certificate validation; final result was Verify return code: 0 (ok).

5. Downloaded the full root and intermediate bundle from the GlobalSign support page, saved as `bundle.pem`. Re-ran to ensure the full chain was verified:

```cmd
openssl s_client -connect google.com:443 -CAfile bundle.pem -verify_return_error
```

Returned `Verify return code: 0 (ok)`.

---

## Resolution

Explicitly supplied a trusted CA bundle using the `-CAfile` parameter. OpenSSL does not integrate with the Windows certificate store, so the bundle must be provided manually.

```cmd
openssl s_client -connect google.com:443 -CAfile bundle.pem -verify_return_error
```

Expected output:

```
Verify return code: 0 (ok)
```

---

## Prevention Note

Always provide a trusted CA bundle when using OpenSSL on Windows, since it does not use the Windows certificate store. Validate results using the Verify return code: or OK output only, as OpenSSL may display connection or shutdown messages that are not related to certificate validation.

---

*Lab this scenario is drawn from: labs/week-04 — Trust Store Inspection*
