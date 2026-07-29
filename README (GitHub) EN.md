# Bildungshub RLP

**Master-data platform for schools, school authorities and regional media centres in Rhineland-Palatinate, Germany.**

Replaces spreadsheet-based record keeping with a shared, auditable database that the state's education advisors use to coordinate school digitalisation programmes.

---

> **Status: source code not published yet.**
> This repository currently documents the system. The working codebase is an internal application of a German state authority and contains real school master data plus service credentials in its Git history. It cannot be published as-is. The plan is to extract and release a data-free, reusable core under an open licence. See [Roadmap](#roadmap-to-open-source).

---

## The problem

Every German federal state, and most larger school authorities within them, keeps its school landscape in spreadsheets. Who is the responsible authority for a given school. Which regional media centre supports it. Which state digitalisation programmes have already reached it. Who the local coordinator is.

Spreadsheets do not survive collaboration. There is no single current version, no change history, no access control, no way to ask "show me every secondary school in this district that has not yet started programme X".

Bildungshub RLP is the answer to that, built for one state and designed to generalise.

## Scale

| | |
|---|---|
| Schools | 1,664 |
| School authorities (Schulträger) | 385 |
| Regional media centres (Medienzentren) | 30 |
| Tracked programme fields per school | 22 |
| Automated tests | 402 |
| Schema migrations | 46 |
| Current version | v4.6 |

Population covered: the full public school landscape of a federal state with roughly 4 million residents.

## What it does

**Search and filter.** Full-text and faceted search across all three record types, with autocompleting filters for place, district, region and programme participation. Saved filter and column presets, refinable before they are applied.

**Map view.** Geocoded schools and authorities on an interactive map, searchable by name or official number, with popups that respect the viewer's permission level.

**Reporting.** Configurable table reports and per-record detail reports, exportable to PDF via WeasyPrint, with a consistent card layout shared between screen and print.

**Collaborative editing.** Role-based permissions, edit locks to prevent concurrent overwrites, and a full change audit trail. Personal contact data is visible only to staff roles and is deliberately excluded from the map payload, enforced by a guard test.

**CSV round trip.** Import and export share a single column-name contract, so an exported file can be edited and re-imported without mapping work. Overlay files keep manually researched data (homepages, phone numbers) stable across full data rebuilds.

**Historical tracking.** Programme participation history and periodic metric snapshots, enabling trend statements rather than point-in-time answers.

## Architecture

| Layer | Technology |
|---|---|
| Web framework | Django 5.2 LTS (Python 3.12) |
| Database | PostgreSQL 16 |
| Application server | Gunicorn |
| Reverse proxy | Nginx with Let's Encrypt |
| Authentication | Active Directory over LDAP |
| Static assets | WhiteNoise, compressed manifest storage |
| PDF rendering | WeasyPrint |
| Deployment | Docker / docker compose |

### Domain model

Three primary entities with a supporting cast:

- **Schule** (school): official number, type, address, contacts, 22 programme participation fields, links to its authority and supporting media centre
- **Träger** (school authority): municipal or private body responsible for a set of schools, including inter-municipal cooperation (IKZ) groupings
- **Medienzentrum** (regional media centre): supports schools in a district; its school count is derived, not maintained
- **Programmverlauf**: participation history per school and programme
- **KennzahlSnapshot**: periodic metric snapshots for trend analysis

## Engineering notes

Some decisions that shaped the codebase and would carry over to any extracted version:

- **Field labels are load bearing.** Django `verbose_name` values double as CSV column headers and as the import matching key. Changing one for cosmetic reasons silently breaks the import round trip. Human-readable display is handled by a separate label layer instead, which means no migration is needed to rename something on screen.
- **Cleanup logic never lives only in a migration.** A data migration runs once; the next import undoes it if the same normalisation is not also present in the importer. Migration heals the existing records, importer keeps them clean.
- **Green tests are not proof a fix works.** UI and layout defects get reproduced in a browser first, root-caused second, and only then covered by a test. Tests written directly from an assumed cause tend to encode the same wrong assumption.
- **Validation strictness belongs where humans enter data,** not in shared normalisation helpers. The web import rejects unknown values; the CLI seed stays tolerant, because the source spreadsheets contain historically messy cells that would otherwise block a full rebuild.

## Security and data protection

The system handles personal data of school staff and is built for a German public authority, so it operates under GDPR.

- Independent security review completed, with a follow-up review after the v3.0 rewrite
- Records of processing and technical/organisational measures documented, pending Data Protection Officer sign-off
- Contact data segregated by role, with automated tests asserting that personal fields never reach client-side payloads
- Packaging now builds from a Git commit rather than an exclusion list, after an audit found that earlier release archives had inadvertently included an environment file

That last point is also why this repository does not yet contain code. Publishing the existing history would publish those secrets.

## Roadmap to open source

1. **Reach production.** Complete GDPR documentation and DPO sign-off, then production rollout.
2. **Separate data from application.** Move all state-specific seed data, overlays and source spreadsheets out of the source tree.
3. **Extract a reusable core.** Domain model, import/export pipeline, role and audit concept, reporting layer. Nothing Rhineland-Palatinate specific.
4. **Publish with a clean history.** New repository, no inherited commits, licence and contribution guidelines in place.

The goal is that another state, or a school authority anywhere, can deploy this against their own data instead of building the same thing again.

**Intended licence:** ⟪MIT oder EUPL 1.2 — entscheiden⟫

## Context

Built and maintained by me

I am an IT consultant advising schools and school authorities, not a professional software engineer. Django, PostgreSQL and database design are things I learned building this. The domain knowledge, requirements and testing are mine; Claude does the implementation. This project exists because that combination works.
