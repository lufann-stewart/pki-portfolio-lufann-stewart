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
(your answer here)
```

**What is PKCS#11?**

In your own words, describe what the PKCS#11 interface is:

```
(your answer here)
```

**HSM being simulated in this demo:**

| Item | Value |
|------|-------|
| Software used | SoftHSM2 (Windows) |
| PKCS#11 library path | |
| Companion tool used (key ceremony commands) | |

---

## Part B — Step-by-Step Observation

Document each step of the ceremony as the instructor runs it. Record the command, what it did, and what you observed in the output.

---

### Step 1 — Token Slot Initialization

**Command run by instructor:**

```
(paste or type the exact command here)
```

**What this command does:**

```
(describe in your own words)
```

**Output or confirmation observed:**

```
(paste or describe output)
```

**Why this step is necessary:**

```
(your explanation)
```

---

### Step 2 — Confirm Slot/Token State

**Command run:**

```
(paste command)
```

**Output:**

```
(paste output)
```

**What did the output confirm?**

```
(your observation)
```

---

### Step 3 — Key Generation

**Command run:**

```
(paste command)
```

**Key parameters used (record what you observed):**

| Parameter | Value |
|-----------|-------|
| Key type | |
| Key size | |
| Key label | |
| Token | |

**Output:**

```
(paste output)
```

**What this step accomplished:**

```
(your explanation)
```

---

### Step 4 — Confirm Key is Token-Resident

**Command run:**

```
(paste command)
```

**Output:**

```
(paste output)
```

**What the output proves:**

```
(describe what it demonstrates about where the key lives)
```

---

### Step 5 — Any Additional Steps Demonstrated

*(Use this section if the instructor ran additional commands — e.g., key attribute inspection, PKCS#11 URI display, CA configuration reference)*

**Command:**

```
(paste command)
```

**What it showed:**

```
(your observation)
```

---

## Part C — SoftHSM2 vs. a Physical HSM

**Physical HSM reference: Thales Luna Network HSM**

Fill in the table based on the lesson and demonstration:

| Dimension | SoftHSM2 (Demo) | Thales Luna (Enterprise) |
|-----------|-----------------|--------------------------|
| Key storage location | Software (filesystem) | |
| Tamper resistance | None | |
| PKCS#11 interface | Same | |
| FIPS validation | Not applicable | |
| Used in production CAs | No — simulation only | |
| Cost | Free / open source | |
| Required for a CA private key | No | Common in enterprise PKI |

**In your own words: what does a physical HSM protect against that SoftHSM2 cannot?**

```
(your answer here — 2–3 sentences)
```

---

## Part D — The Key Ceremony Concept

**What is a key ceremony?**

In your own words, describe what a key ceremony is and why enterprise PKI deployments conduct them with multiple administrators present:

```
(your answer — 3–5 sentences)
```

**What would be different about this ceremony if CVI Issuing CA 1 were using a physical HSM?**

```
(your answer — think about: who would be in the room, what physical steps would be required, what documentation would be produced)
```

---

## Part E — Connection to Phase 2

**Software-protected CA key vs. HSM-protected CA key**

CVI Issuing CA 1 in your lab environment stores its private key in software (the Windows CNG Key Storage Provider). Based on what you observed in this demonstration:

**What is the operational risk of a software-stored CA private key vs. an HSM-protected key?**

```
(your answer)
```

**When you back up the CA in Week 13, what specific step is more sensitive because the private key is software-protected?**

```
(your answer — think about what certutil -backup exports)
```

---

## Reflection

**The most important thing you took away from this demonstration:**

```
(your answer)
```

**One question the demonstration raised that you want to understand better:**

```
(your question)
```

---

## Submission Checklist

- [ ] Part A: SoftHSM2 and PKCS#11 described in own words
- [ ] Part B: All ceremony steps documented — commands, outputs, explanations
- [ ] Part C: SoftHSM2 vs. Thales Luna comparison table completed
- [ ] Part C: Physical HSM protection question answered
- [ ] Part D: Key ceremony concept explained in own words
- [ ] Part D: Enterprise ceremony differences described
- [ ] Part E: Software key vs. HSM key risk addressed
- [ ] Part E: Connection to Week 13 backup noted
- [ ] Reflection completed
- [ ] File saved as `lab-03-hsm-observation.md`
- [ ] File committed to portfolio repo under `labs/week-10/`
