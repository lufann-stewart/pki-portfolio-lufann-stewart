# Lab 03: HSM Key Ceremony — Observation Report

Lufann Stewart    
May 23, 2026      
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-03-hsm-observation.md`

> **Note:** This is an instructor-led demonstration. You observe and document — no hands-on configuration required. The instructor runs SoftHSM2 on a dedicated demo machine. Your deliverable is a structured observation report.

---

## Overview

A Hardware Security Module (HSM) is a physical device that stores cryptographic keys in tamper-resistant hardware. Enterprise CAs use HSMs to protect CA private keys — the most sensitive material in the PKI environment. In this demonstration, the instructor runs a **PKCS#11 key ceremony** using **SoftHSM2**, a software-based HSM simulator that exposes the same interface as a physical HSM like the Thales Luna.

Your task: observe each step, document what the instructor ran, what it produced, and what the equivalent step would look like with a real HSM in an enterprise environment.

---

## Part A — Setup Context

**What is SoftHSM2?**

In your own words, describe what SoftHSM2 is and why it is being used for this demonstration instead of a physical HSM:

```
SoftHSM2 is a software-based cryptographic store that mimics the behavior of a physical Hardware Security Module (HSM) through the PKCS#11 interface. It was used in this demonstration because it provides a free, open-source way to practice and simulate key ceremonies without needing costly dedicated hardware. In real-world production environments, however, organizations typically use physical HSMs because they offer stronger security protections, including tamper resistance and features like automatic key zeroization if the device is compromised.
```

**What is PKCS#11?**

In your own words, describe what the PKCS#11 interface is:

```
PKCS#11 is the interface that sits between the CA and the HSM. It lets the CA request cryptographic operations like key generation and signing without ever directly touching the private key material inside the HSM.
```

**HSM being simulated in this demo:**

| Item | Value |
|------|-------|
| Software used | SoftHSM2 (macOS/Homebrew)) |
| PKCS#11 library path |`/opt/homebrew/lib/softhsm/libsofthsm2.so` |
| Companion tool used (key ceremony commands) |`pkcs11-tool` |

---

## Part B — Step-by-Step Observation

Document each step of the ceremony as the instructor runs it. Record the command, what it did, and what you observed in the output.

---

### Step 1 — Token Slot Initialization

**Command run by instructor:**

```
softhsm2-util --show-slots
```

**What this command does:**

```
This command checks available HSM slots and confirms whether any tokens have already been initialized. It ensures the environment is clean before beginning the key ceremony.
```

**Output or confirmation observed:**

```
The output displayed detailed slot and token information, including the slot description, manufacturer ID, hardware and firmware versions, token presence status, token model, serial number, initialization status, user PIN initialization status, and token label.
```

**Why this step is necessary:**

```
This step is necessary to confirm that the slot is clean and does not already contain initialized tokens or existing cryptographic keys. Verifying the environment beforehand helps prevent accidentally overwriting existing key material or building on top of a misconfigured or previously used token.
```

---

### Step 2 — Confirm Slot/Token State

**Command run:**

```
softhsm2-util --init-token --slot 0 \
--label "CVI-IssCA-Key" \
--pin <USER_PIN> \
--so-pin <SO_PIN>
```

**Output:**

```
The token was successfully initialized and reassigned to a new slot (2076269524).
```

**What did the output confirm?**

```
The output confirmed that the token was successfully initialized, assigned a new slot number, and configured with security credentials such as the SO PIN and user PIN. It also confirmed that a token was now present and active in the slot.
```

---

### Step 3 — Key Generation

**Command run:**

```
pkcs11-tool --module /opt/homebrew/lib/softhsm/libsofthsm2.so  \
--login --pin <USER_PIN> \
--keypairgen --key-type rsa:2048 \
--label "CVI-IssCA-Key" \
--id 01
```

**Key parameters used (record what you observed):**

| Parameter | Value |
|-----------|-------|
| Key type |RSA |
| Key size |2048|
| Key label |CVI-IssCA-Key |
| Token |Yes |

**Output:**

```
The output showed that an RSA 2048-bit public/private key pair was successfully generated. It displayed attributes such as the key label, object ID, usage permissions, access settings, modulus, public exponent, and PKCS#11 URI information for both the private and public key objects.
```

**What this step accomplished:**

```
This step generated an RSA public/private key pair directly inside the HSM token so the private key never leaves the cryptographic boundary.
```

---

### Step 4 — Confirm Key is Token-Resident

**Command run:**

```
pkcs11-tool --module /opt/homebrew/lib/softhsm/libsofthsm2.so \
--login --pin <USER_PIN> \
--list-objects
```

**Output:**

```
The output listed the objects stored on the token, including both the public and private key objects and their associated attributes.
```

**What the output proves:**

```
This output is proof that our keys are permanently stored on the token itself, rather than just being temporary files that get wiped out when the session closes.
```

---

### Step 5 — Any Additional Steps Demonstrated

*(Use this section if the instructor ran additional commands — e.g., key attribute inspection, PKCS#11 URI display, CA configuration reference)*

**Command:**

```
softhsm2-util --version
```

**What it showed:**

```
The version of softhsm2 currently installed.
```

---

## Part C — SoftHSM2 vs. a Physical HSM

**Physical HSM reference: Thales Luna Network HSM**

Fill in the table based on the lesson and demonstration:

| Dimension | SoftHSM2 (Demo) | Thales Luna (Enterprise) |
|-----------|-----------------|--------------------------|
| Key storage location | Software (filesystem) |Physical HSM |
| Tamper resistance | None |Yes |
| PKCS#11 interface | Same |Same |
| FIPS validation | Not applicable |Yes |
| Used in production CAs | No — simulation only | Yes|
| Cost | Free / open source |Cost |
| Required for a CA private key | No | Common in enterprise PKI |

**In your own words: what does a physical HSM protect against that SoftHSM2 cannot?**

```
A physical HSM protects against key extraction, disk or memory compromise, and unauthorized administrative access by ensuring private keys never exist outside the hardware boundary.
```

---

## Part D — The Key Ceremony Concept

**What is a key ceremony?**

In your own words, describe what a key ceremony is and why enterprise PKI deployments conduct them with multiple administrators present:

```
A key ceremony is a formal, controlled process used to generate and manage highly sensitive cryptographic keys inside an HSM. Enterprise PKI environments often require multiple administrators to participate using quorum controls or split knowledge so that no single person has complete control over the CA private key.
```

**What would be different about this ceremony if CVI Issuing CA 1 were using a physical HSM?**

```
A physical HSM ceremony would involve dedicated tamper-resistant hardware instead of software-based token files stored on the filesystem. Access controls would typically be stricter, and the private key material would remain protected inside the physical device throughout the ceremony.
```

---

## Part E — Connection to Phase 2

**Software-protected CA key vs. HSM-protected CA key**

CVI Issuing CA 1 in your lab environment stores its private key in software (the Windows CNG Key Storage Provider). Based on what you observed in this demonstration:

**What is the operational risk of a software-stored CA private key vs. an HSM-protected key?**

```
A software-stored key can be copied off disk, extracted from a backup, or accessed by anyone with filesystem/admin access, whereas an HSM-protected key never leaves the hardware boundary.
```

**When you back up the CA in Week 13, what specific step is more sensitive because the private key is software-protected?**

```
The CA private key will be exported as part of the backup, and since it's software-protected (not bound to hardware), that exported key material is portable and could be stolen if the backup isn't properly secured. The risk is key portability, not just encryption.
```

---

## Reflection

**The most important thing you took away from this demonstration:**

```
The most important thing I took away from this demonstration was how quorum controls are used in HSM environments so that no single administrator has complete control over highly sensitive cryptographic keys.
```

**One question the demonstration raised that you want to understand better:**

```
How are key ceremonies planned in enterprise environments, and how do organizations determine which administrators participate in the quorum process?
```

---

## Submission Checklist

- [X] Part A: SoftHSM2 and PKCS#11 described in own words
- [X] Part B: All ceremony steps documented — commands, outputs, explanations
- [X] Part C: SoftHSM2 vs. Thales Luna comparison table completed
- [X] Part C: Physical HSM protection question answered
- [X] Part D: Key ceremony concept explained in own words
- [X] Part D: Enterprise ceremony differences described
- [X] Part E: Software key vs. HSM key risk addressed
- [X] Part E: Connection to Week 13 backup noted
- [X] Reflection completed
- [X] File saved as `lab-03-hsm-observation.md`
- [X] File committed to portfolio repo under `labs/week-10/`
