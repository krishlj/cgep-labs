The three policies at a glance
Control	File	What it requires
SC-28	policies/sc28_encryption.rego	Every bucket has a customer-managed encryption key.
AC-3	policies/ac3_no_public.rego	Buckets aren't public; firewalls don't open ports 22 or 3389 to the world.
CM-6	policies/cm6_required_tags.rego	Every taggable resource carries the four required labels.


Every deny message names the resource and the NIST control. That's deliberate: a developer who sees [SC-28] google_storage_bucket.bad_no_cmek: missing customer-managed encryption key fixes it themselves, without a GRC ticket or a meeting.

Where these files live
Policies live at the repo root in policies/, which is exactly where your capstone expects them. The throwaway test infrastructure goes in a primitive.

cgep-labs/
├── policies/
│   ├── sc28_encryption.rego
│   ├── ac3_no_public.rego
│   ├── cm6_required_tags.rego
│   ├── README.md
│   └── tests/
│       ├── sc28_encryption_test.rego
│       ├── ac3_no_public_test.rego
│       └── cm6_required_tags_test.rego
├── terraform/primitives/policy-fixture/   ← plan-only test bed
│   └── main.tf
└── evidence/lab-3-3/
    └── opa-test-results.json              ← filled in when you capture evidence
