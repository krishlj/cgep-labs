# Compliant GCS Bucket Module

## What we built, and the controls behind it

The module produces, in one unit, a KMS keyring, a customer-managed encryption key that rotates on a schedule, an IAM binding so the storage service can use that key, and a bucket that's hardened by default. Two consumers (dev and prod) call the module with different business settings and inherit the identical security posture.

| **Control** | **What it means in plain terms** | **Where the module enforces it** |
|---|---|---|
| **SC-12** | You establish and own the encryption key, rather than letting the provider hold it. | `google_kms_key_ring` + `google_kms_crypto_key` |
| **SC-13 / SC-28** | Data is encrypted at rest with that key (a CMEK), and the key rotates. | `encryption {}` block + `rotation_period` |
| **AC-3** | Access is uniform and the public can't reach the bucket. | `uniform_bucket_level_access` + `public_access_prevention` |
| **AU-11** | Records are retained for a set period. | `retention_policy` |
| **CM-6** | Required labels are present and can't be dropped. | merged `labels` |

## Architecture

```text
                          consumer: dev                consumer: prod
                                │                            │
                                ▼                            ▼
                  ┌──────────────────────────────────────────────┐
                  │        module: compliant-gcs-bucket           │
                  │                                               │
                  │   KMS keyring ─▶ crypto key (rotates,        │
                  │                      SC-12/13)                │
                  │                      │                        │
                  │                      │ encrypter              │
                  │                      ▼                        │
                  │            hardened GCS bucket                │
                  │                                               │
                  │   uniform access · CMEK · versioning ·        │
                  │   retention · required labels · public block  │
                  └──────────────────────────────────────────────┘


The important design is that Dev and Prod use the same security module. They can have different business settings, such as retention periods, while inheriting the same security posture.

Where these files live

The module is a reusable template, so it goes under terraform/modules/. The consumers are things you actually deploy, so they're primitives under terraform/primitives/.

cgep-labs/
├── terraform/
│   ├── modules/
│   │   └── compliant-gcs-bucket/   ← the module (the security floor)
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       ├── outputs.tf
│   │       └── README.md
│   │
│   └── primitives/
│       ├── compliant-gcs/          ← consumer: dev (you apply this)
│       │   └── main.tf
│       │
│       ├── compliant-gcs-prod/     ← consumer: prod (you only plan this)
│       │   └── main.tf
│       │
│       └── compliant-gcs-negative/ ← the validation-failure demo (plan only)
│           └── main.tf
│
└── evidence/
    └── lab-2-4/                    ← compliance evidence

The module and the consumers are separated so the distinction stays visible.

A consumer is a few lines of business configuration.

Compliance validation

The module also validates configuration during terraform plan.

For example, production requires a minimum retention period of 365 days:

environment = prod
retention_days = 30
        ↓
     ❌ FAIL
        ↓
retention_days must be >= 365

This prevents an invalid production configuration from reaching the deployment stage.

Dev and Prod consumers

Both environments use the same module:

                  compliant-gcs-bucket
                          │
                 ┌────────┴────────┐
                 │                 │
                DEV               PROD
                 │                 │
           30-day retention   365-day retention
                 │                 │
               APPLY              PLAN

The security controls remain inside the reusable module instead of being duplicated in each environment.

Compliance Evidence

The module exposes a machine-readable compliance_attestation output containing information about the enforced controls.

The Dev consumer re-exposes this output as:

attestation

Evidence for the lab is stored under:

evidence/lab-2-4/
├── plan.json
└── attestation.json

What this README is documenting

Your lab now has a clean separation:

MODULE
terraform/modules/compliant-gcs-bucket/
        │
        ├── Security controls
        ├── KMS
        ├── Encryption
        ├── Public access prevention
        ├── Versioning
        ├── Retention
        ├── Labels
        └── Validation
                 │
                 ▼
       ┌─────────┴─────────┐
       ▼                   ▼
     DEV                  PROD
   Consumer             Consumer
    APPLY                PLAN
