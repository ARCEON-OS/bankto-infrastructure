# BPMD-001 — Synthetic-State Staging Runbook

## Scope

This runbook is executable only for the authorised `bankto-staging-synthetic` environment. All fixture identifiers must begin with `test-` or `test:` and all successful financial-integrity instructions must use the deterministic Sprint 1 amount of `100 USD`.

## Pre-deployment stop conditions

Stop immediately and classify the gate as blocked if any condition is true: a production account, credential, Auth0 tenant/audience, database, balance, rail endpoint, ATM service, blockchain RPC endpoint, Stripe, Circle, Fireblocks, or BNKTO deployment reference is present; an image lacks a recorded digest/SBOM/provenance record; `auth-egress` is absent or unrestricted; backup/restore or alert routing is not provisioned; or the release authority is unassigned.

## Controlled deployment

1. Verify the Core/API image SHA, immutable image digest, archive checksum, SBOM, and source-provenance record.
2. Create fresh synthetic PostgreSQL databases and apply reviewed migrations. Record migration tables and schema checksums.
3. Create the firewall/proxy-backed `auth-egress` network, which permits only the Auth0 Development JWKS and token endpoints. Retain its policy export as evidence.
4. Inject secrets only through the approved staging secret manager. Do not create a populated `.env` file, compose override, GitHub secret printout, or report containing values.
5. Deploy `docker-compose.staging-synthetic.yml`. Before API/Core startup, install and verify the source-scoped default-deny `DOCKER-USER` rule for the dynamic `backplane` subnet. It must allow only same-subnet service traffic; only API may attach to the separately egress-restricted Auth0 network.
6. Run health, readiness, dependency-loss/recovery, Auth0 HTTP, service-auth, ledger, audit, reconciliation, and observability matrices in the certification gate.

The staging overlay uses an `intell-advisory` health sentinel because API readiness declares an INTELL dependency while Sprint 1 deliberately excludes recommendation processing. The sentinel exposes only `/health`; it contains no recommendation, acceptance, or execution endpoint. This is a bounded readiness dependency test double, not evidence of full INTELL operational certification.

## Rollback and recovery

Stop rollout and preserve evidence upon any mandatory-control failure. Roll back only to the previously approved immutable image digest after checking migration compatibility. If migration rollback is unsafe, stop the application, restore the declared encrypted synthetic backup to a new database, validate migration state and ledger/audit/reconciliation invariants, then direct traffic only after readiness passes. Do not delete failure evidence.

## Backup, restore, and disaster recovery

Before the first financial test, capture an encrypted synthetic-only backup. Restore it into a fresh isolated target at least once. Record duration, migration-version equality, row-level integrity checks, and the measured RTO/RPO. A failure to restore or reconstruct the service environment from the committed compose file, image archives, and approved secret references blocks certification.

## Completion and teardown

On completion, export redacted logs, alert acknowledgements, dashboards, migration records, checksums, release manifest, and drill results. Revoke synthetic Auth0 access, rotate test service/database credentials, destroy test databases and volumes under the approved retention policy, remove temporary image archives from the execution host, and retain only redacted evidence and integrity hashes.
