# Device And Artifact Catalog

This directory answers five separate questions:

1. Which device identities are present in a current maintained inventory?
2. Which carrier artifacts are present in a current maintained vendor index?
3. Which devices match exact model scope recorded by the carrier-profile evidence?
4. Which source checks are complete, still running over time, or cannot be made?
5. Which exact devices have evidence establishing whether carrier data is relevant?

Those questions are intentionally separate. A listed device is not automatically
tested, and a family-level carrier artifact is not proof that every carrier
feature works on every model.

Files:

- `index.json`: counts by platform and retail brand, plus source revisions;
- `android.json`: merged current and historical Android device identities plus
  per-device carrier-data coverage;
- `apple.json`: current and historical Apple product types from Apple's carrier
  index;
- `apple-carrier-artifacts.json`: current public-safe Apple artifact metadata.
- `android-carrier-artifacts.json`: current public-safe Android artifact metadata
  plus exact source-discovery and terminal scope records.

The index includes coverage-state counts per retail brand. Schema-version-2
indexes also include exact carrier-relevance totals and a complete, zero-filled
coverage-by-relevance matrix. This makes missing evidence and OEM adapters
visible without opening tens of thousands of device records.

Schema-version-2 inventories require a `carrier_relevance` object on every
device. Its status is one of `evidence_confirmed_cellular`,
`evidence_confirmed_non_cellular`, or `not_established`. Confirmed statuses
require a sorted, unique, non-empty evidence list; `not_established` requires an
empty list. Each evidence record names its exact maintained source and one of:
`exact_carrier_observation`, `extracted_carrier_configuration`,
`exact_product_type_carrier_bundle`, `official_connectivity_specification`, or
`official_connectivity_variant`. Existing schema-version-1 inventories remain
temporarily valid during the publication migration, but cannot contain the v2
field.

Exact observation and extracted-configuration evidence is Android-only; an
extracted catalog must use `exact_device_id` and contain only the enclosing
Android ID. Exact product-type carrier-bundle evidence is Apple-only. Official
connectivity specification and exact-variant evidence can classify either
platform when its source is declared and joined to that exact device.

Relevance is never inferred from a family, product name, form factor, or
carrier-data coverage status. In particular, Apple TV and iPod labels and
`platform_out_of_scope` do not establish that a device is non-cellular. A
non-cellular classification needs exact official connectivity evidence;
carrier observations, extracted carrier configuration, and exact product-type
carrier bundles are conflicting cellular evidence and fail validation. A v2
non-cellular classification and `carrier_data_not_applicable` must occur
together on both platforms.

`verification: indexed` means the artifact and its digest are present in the
current official index. `verification: verified` means automation also fetched
the package and matched that digest. Failed downloads or digest mismatches are
quarantined and are not included in the public artifact file.

For Android, `indexed` means an exact vendor query confirmed a current artifact.
`extracted` means the device-scoped carrier source was also obtained and
integrity-checked. Discovery states distinguish work in progress, a completed
check with no artifact, and identities that lack a usable vendor query key.

In schema version 2, `carrier_data_not_applicable` requires an
evidence-confirmed non-cellular classification for both Apple and Android;
Android also retains its exact official connectivity-source requirement.
`source_transport_untrusted` records an exact Android
source whose archive or binary-package transport does not meet unattended
integrity requirements; it has no artifacts and is not a successful carrier
extraction. Unrecognized present or future identities remain `inventory_only`
until evidence classifies them.

One device can be listed by more than one maintained inventory. Such records
merge only when their canonical device ID is identical, and
`inventory_sources` keeps each source's current or historical state visible.
Android artifact and discovery records may link directly through that canonical
ID. `match_kind: exact_device_id` means the maintained source adapter supplied
that exact link; it does not rely on a similar marketing name or fuzzy alias.

Use the schemas under `schemas/` and run:

```bash
python3 tools/validate_device_catalog.py generated/devices
```
