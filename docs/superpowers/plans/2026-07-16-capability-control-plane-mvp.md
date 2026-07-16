# Capability Control Plane MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (- [ ]) syntax for tracking.

**Goal:** Build a local, read-only architecture-to-policy compiler that turns current MCP/tool observations and architecture intent into candidate contracts, compares them with an independently reviewed prior-approved baseline, and emits minimum bundles, policy simulations, target diffs, drift findings, and explainable assurance reports.

**Architecture:** The MVP runs out of band and never proxies runtime traffic or handles credentials. A deterministic pipeline ingests sanitized current observations, a separate prior-approved contract baseline, versioned evidence and equivalence records, optional Part I Architecture Packs, and optional sanitized traces. Current observations and exact-schema Part I imports can create review candidates and drift evidence but can never create their own approval baseline. Same-implementation observations are aggregated with explicit precedence and conflicting protected projections are quarantined. Drift and evidence freshness are resolved before bundle compilation; every bundle field is re-derived from authoritative inputs before simulation. Persisted outputs are verified against both semantic and exact-byte digests. Existing hosts and gateways remain the runtime enforcement plane.

**Tech Stack:** Node.js ESM 22.23.1 or newer, Node built-in test runner, Ajv 8.20.0, ajv-formats 3.0.1, yaml 2.9.0, Node crypto, Markdown/YAML/JSON, GitHub Actions.

## Global Constraints

- RFC 0002 is normative: docs/rfcs/0002-architecture-driven-capability-control-plane.md.
- MCP 2025-11-25 is the current normative baseline; the compatibility target uses the RFC wire value `2026-07-28`. Its release-candidate maturity is tracked in documentation, never by appending `-rc` to `protocol_version` or `protocolVersion`.
- The MVP MUST remain outside the runtime data path and MUST NOT proxy MCP calls.
- The MVP MUST NOT accept, store, exchange, or log OAuth tokens, API keys, cookies, or production credentials.
- The MVP MUST NOT apply target configuration; it emits reviewable files and diffs only.
- Policy eligibility is deterministic. Semantic ranking or an LLM may never grant authorization.
- Server annotations and descriptions remain untrusted source assertions.
- Current discovery output and Part I Architecture Pack output are candidate-only. Neither may enter an eligible bundle without a separate, prior review record that binds the exact contract digest.
- A first-run assessment with no independently supplied prior-approved baseline may emit candidates and gaps, but MUST include zero eligible capabilities, no conformance suggestion, and zero applicable target proposals.
- The prior-approved baseline, approval records, evidence ledger, equivalence proofs, and current observations are separate input documents with separate manifest digests.
- Drift is computed against the prior-approved implementation projection and quarantine is applied before bundle compilation.
- Unknown identity, tenant, audience, scope, schema, write effect, evidence freshness, or equivalence yields indeterminate and therefore deny in simulation.
- Bundle fields are evidence, not authority. Simulation MUST recompute the complete expected included item from policy, contract, evidence, and context and reject any field-level tampering.
- Offline approval records bind the exact contract digest and reviewed implementation digest, carry a unique nonce and approval digest, and are single-use within a tenant-scoped replay ledger.
- A target adapter MUST fail on an unsupported mandatory semantic; it MUST NOT silently weaken policy.
- OpenClaw may emit a non-authoritative conformance suggestion, but its separate policy-enforcement result is unsupported, preserves no mandatory Policy IR semantics, and can never be cited as authorization or IR-preservation evidence.
- Docker may report server/profile exposure analysis, but when purpose, data-flow, argument binding, approval, write behavior, or server-level least privilege is mandatory and unsupported it MUST return `status: unsupported`, `proposal: null`, and no actionable diff.
- High-risk write behavior is offline simulation only.
- The deterministic one-server fixture is a smoke test, not design-partner or market evidence; pilot claims require digest-bound prior-approved/bundle/drift/target authority, one current eligible read, no critical quarantine, official/private/live sources, two authoritative target surfaces, 3–5 servers, 10–30 capabilities, independently identified passing suites, one shared approved high-risk write/destructive contract binding, and zero derived false permits from sanitized partner inputs.
- Public distribution remains blocked until the rights owner commits the repository license text and repository administrators configure real CODEOWNERS principals and protected-branch rules.
- Pin every npm dependency exactly and commit package-lock.json.
- Use test-driven development and commit after every task.

---

## File Map

### Package and contracts

- control-plane/package.json — isolated CLI package and exact dependencies.
- control-plane/schemas/source-snapshot.schema.json — sanitized source snapshot.
- control-plane/schemas/assessment-inputs.schema.json — explicit multi-source input manifest and trust-boundary declaration.
- control-plane/schemas/capability-mappings.schema.json — explicitly reviewed source-to-capability mappings.
- control-plane/schemas/assessment-context.schema.json — exact assessment identity, workflow, purpose, region, and time context.
- control-plane/schemas/capability-contract.schema.json — organization-approved capability semantics.
- control-plane/schemas/approved-baseline.schema.json — independently reviewed prior-approved contracts and approval records.
- control-plane/schemas/evidence-ledger.schema.json — versioned, expiring evidence records bound to reviewed implementations.
- control-plane/schemas/equivalence-proof.schema.json — reviewed, versioned fallback equivalence proof.
- control-plane/schemas/approval-record.schema.json — digest- and nonce-bound offline approval input.
- control-plane/schemas/part1-candidate-capability-map.schema.json — exact Part I candidate handoff vocabulary, vendored for independent validation.
- control-plane/schemas/architecture-pack-import.schema.json — candidate-only import result from a Part I pack.
- control-plane/schemas/trace-set.schema.json — sanitized selection, outcome, retry, poisoning, and contract/action-bound high-risk traces.
- control-plane/schemas/target-baselines.schema.json — exact versioned target configuration baselines.
- control-plane/schemas/pilot-scorecard.schema.json — authority-digest-bound technical RFC MVP pilot gate result.
- control-plane/schemas/workflow-contract.schema.json — stage-specific architecture intent.
- control-plane/schemas/policy-ir.schema.json — vendor-neutral deterministic policy.
- control-plane/schemas/source-inventory.schema.json — provenance-bound source inventory output.
- control-plane/schemas/capability-bundle.schema.json — included and excluded capability evidence.
- control-plane/schemas/assessment-manifest.schema.json — complete output manifest.

### Deterministic core

- control-plane/src/canonical-json.mjs — stable JSON canonicalization and SHA-256.
- control-plane/src/validate.mjs — shared Ajv validation.
- control-plane/src/evidence/load-snapshot.mjs — read-only YAML/JSON ingestion and secret rejection.
- control-plane/src/evidence/sanitized-boundary.mjs — canonical-root, per-component non-symlink input resolver.
- control-plane/src/evidence/freshness.mjs — current-source and reviewed-evidence freshness checks.
- control-plane/src/evidence/verify-artifacts.mjs — semantic and exact-persisted-byte artifact digest verifier.
- control-plane/src/import/architecture-pack.mjs — Part I manifest/digest verifier that emits candidates only.
- control-plane/src/contracts/implementation-projection.mjs — protected implementation projection used for approval and drift.
- control-plane/src/contracts/normalize-capability.mjs — explicit tool-to-business-capability mapping.
- control-plane/src/contracts/integrity.mjs — cross-field contract and approval-record integrity checks.
- control-plane/src/contracts/store.mjs — in-memory prior-approved contract index; candidates use a separate collection.
- control-plane/src/compile/policy.mjs — Workflow Contract to Policy IR.
- control-plane/src/compile/bundle.mjs — deterministic minimum eligible bundle.
- control-plane/src/adapters/openclaw.mjs — non-authoritative conformance suggestion plus unsupported policy-enforcement result.
- control-plane/src/adapters/docker.mjs — limited profile/server artifact.
- control-plane/src/drift/analyze.mjs — protected-contract drift and quarantine.
- control-plane/src/drift/aggregate-observations.mjs — source-precedence aggregation, corroboration refs, and multi-source conflict quarantine.
- control-plane/src/eval/simulate.mjs — policy, composition, approval, retry, and fallback simulation.
- control-plane/src/eval/replay-ledger.mjs — tenant-scoped single-use nonce ledger for offline approval simulation.
- control-plane/src/eval/pilot-gate.mjs — smoke-versus-RFC-pilot evidence gate.
- control-plane/src/explain/explain.mjs — decision explanation.
- control-plane/src/assessment.mjs — end-to-end assessment assembly.
- control-plane/src/cli.mjs — local assess command.

### Fixtures, reports, and governance

- control-plane/tests/*.test.mjs — unit, adapter, security, and end-to-end tests.
- control-plane/tests/fixtures/smoke/ — deterministic one-server smoke input, including an independent synthetic prior baseline.
- control-plane/tests/fixtures/first-run/ — current observations with an empty prior-approved baseline.
- control-plane/tests/pilot-gate.test.mjs — synthetic boundary tests for gate logic only; a pilot pass requires a real sanitized partner input pack and trace set.
- control-plane/README.md — exact MVP boundary and commands.
- docs/research/control-plane-design-partner-protocol.md — interview and pilot falsification protocol.
- .github/workflows/control-plane-ci.yml — deterministic tests with no credentials or network.

---

### Task 1: Establish the isolated control-plane package

**Files:**
- Create: control-plane/package.json
- Create: control-plane/package-lock.json
- Create: control-plane/.gitignore
- Create: control-plane/tests/package-metadata.test.mjs

**Interfaces:**
- Produces: npm test and node src/cli.mjs entry points.
- Keeps dependencies independent from the Skill package.

- [ ] **Step 1: Write the failing package metadata test**

Create control-plane/tests/package-metadata.test.mjs:

~~~js
import assert from "node:assert/strict";
import { readFile } from "node:fs/promises";
import test from "node:test";

test("control-plane package is private, ESM, pinned, and Node 22 compatible", async () => {
  const manifest = JSON.parse(await readFile(new URL("../package.json", import.meta.url), "utf8"));
  assert.equal(manifest.private, true);
  assert.equal(manifest.type, "module");
  assert.equal(manifest.engines.node, ">=22.23.1");
  assert.deepEqual(manifest.dependencies, {
    ajv: "8.20.0",
    "ajv-formats": "3.0.1",
    yaml: "2.9.0"
  });
  assert.equal(manifest.scripts.test, "node --test");
  assert.equal(await readFile(new URL("../.gitignore", import.meta.url), "utf8"), "node_modules/\n.tmp/\n");
});
~~~

- [ ] **Step 2: Run the test and verify the missing manifest failure**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/package-metadata.test.mjs
~~~

Expected: FAIL with ENOENT for control-plane/package.json.

- [ ] **Step 3: Add the exact package manifest**

Create control-plane/package.json:

~~~json
{
  "name": "@ai-native-architecture/capability-control-plane",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "engines": {
    "node": ">=22.23.1"
  },
  "bin": {
    "capability-control": "./src/cli.mjs"
  },
  "scripts": {
    "test": "node --test",
    "assess:fixture": "node src/cli.mjs assess tests/fixtures/smoke .tmp/assessment"
  },
  "dependencies": {
    "ajv": "8.20.0",
    "ajv-formats": "3.0.1",
    "yaml": "2.9.0"
  }
}
~~~

Create control-plane/.gitignore:

~~~gitignore
node_modules/
.tmp/
~~~

- [ ] **Step 4: Generate and verify the lockfile**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm install --package-lock-only --ignore-scripts
npm ci --ignore-scripts
npm test
~~~

Expected: package-lock.json is created and one test passes.

- [ ] **Step 5: Commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/package.json control-plane/package-lock.json control-plane/.gitignore control-plane/tests/package-metadata.test.mjs
git commit -m "test: add control-plane harness"
~~~

---

### Task 2: Implement stable canonicalization and digest binding

**Files:**
- Create: control-plane/src/canonical-json.mjs
- Create: control-plane/tests/canonical-json.test.mjs

**Interfaces:**
- Produces: canonicalize(value) -> string.
- Produces: sha256(value) -> string prefixed with sha256:.
- Rejects undefined, functions, symbols, bigint, non-finite numbers, and cyclic objects.

- [ ] **Step 1: Write the failing canonicalization tests**

Create control-plane/tests/canonical-json.test.mjs:

~~~js
import assert from "node:assert/strict";
import { mkdir, mkdtemp, symlink, writeFile } from "node:fs/promises";
import { tmpdir } from "node:os";
import { join } from "node:path";
import test from "node:test";
import { canonicalize, sha256 } from "../src/canonical-json.mjs";

test("object key order does not change digest", () => {
  const left = { b: 2, a: { d: 4, c: 3 } };
  const right = { a: { c: 3, d: 4 }, b: 2 };
  assert.equal(canonicalize(left), '{"a":{"c":3,"d":4},"b":2}');
  assert.equal(sha256(left), sha256(right));
  assert.match(sha256(left), /^sha256:[a-f0-9]{64}$/);
});

test("array order remains significant", () => {
  assert.notEqual(sha256(["a", "b"]), sha256(["b", "a"]));
});

test("prototype-sensitive JSON keys remain hash-bound", () => {
  const protectedKeys = JSON.parse('{"__proto__":{"type":"string"},"constructor":{"type":"number"},"prototype":true,"safe":1}');
  assert.notEqual(sha256(protectedKeys), sha256({ safe: 1 }));
  assert.match(canonicalize(protectedKeys), /"__proto__"/);
});

test("unsupported and cyclic values fail closed", () => {
  assert.throws(() => canonicalize({ value: undefined }), /unsupported value/);
  assert.throws(() => canonicalize(Number.POSITIVE_INFINITY), /non-finite/);
  const cyclic = {};
  cyclic.self = cyclic;
  assert.throws(() => canonicalize(cyclic), /cyclic/);
});
~~~

- [ ] **Step 2: Run the tests and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/canonical-json.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement canonicalize and sha256**

Create control-plane/src/canonical-json.mjs:

~~~js
import { createHash } from "node:crypto";

function normalize(value, seen) {
  if (value === null || typeof value === "string" || typeof value === "boolean") return value;
  if (typeof value === "number") {
    if (!Number.isFinite(value)) throw new TypeError("non-finite number");
    return Object.is(value, -0) ? 0 : value;
  }
  if (typeof value !== "object") throw new TypeError("unsupported value type: " + typeof value);
  if (seen.has(value)) throw new TypeError("cyclic value");
  seen.add(value);
  try {
    if (Array.isArray(value)) return value.map((item) => normalize(item, seen));
    const output = Object.create(null);
    for (const key of Object.keys(value).sort()) {
      if (value[key] === undefined) throw new TypeError("unsupported value at key: " + key);
      output[key] = normalize(value[key], seen);
    }
    return output;
  } finally {
    seen.delete(value);
  }
}

export function canonicalize(value) {
  return JSON.stringify(normalize(value, new Set()));
}

export function sha256(value) {
  return "sha256:" + createHash("sha256").update(canonicalize(value)).digest("hex");
}
~~~

- [ ] **Step 4: Run tests and commit**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
~~~

Expected: all tests pass.

Commit:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/src/canonical-json.mjs control-plane/tests/canonical-json.test.mjs
git commit -m "feat: add stable contract hashing"
~~~

---

### Task 3: Publish and validate the versioned contract schemas

**Files:**
- Create: control-plane/schemas/source-snapshot.schema.json
- Create: control-plane/schemas/assessment-inputs.schema.json
- Create: control-plane/schemas/capability-mappings.schema.json
- Create: control-plane/schemas/assessment-context.schema.json
- Create: control-plane/schemas/capability-contract.schema.json
- Create: control-plane/schemas/approved-baseline.schema.json
- Create: control-plane/schemas/evidence-ledger.schema.json
- Create: control-plane/schemas/equivalence-proof.schema.json
- Create: control-plane/schemas/approval-record.schema.json
- Create: control-plane/schemas/part1-candidate-capability-map.schema.json
- Create: control-plane/schemas/architecture-pack-import.schema.json
- Create: control-plane/schemas/trace-set.schema.json
- Create: control-plane/schemas/target-baselines.schema.json
- Create: control-plane/schemas/pilot-scorecard.schema.json
- Create: control-plane/schemas/workflow-contract.schema.json
- Create: control-plane/schemas/policy-ir.schema.json
- Create: control-plane/schemas/source-inventory.schema.json
- Create: control-plane/schemas/capability-bundle.schema.json
- Create: control-plane/schemas/assessment-manifest.schema.json
- Create: control-plane/src/validate.mjs
- Create: control-plane/tests/schemas.test.mjs
- Create: control-plane/tests/fixtures/smoke/source-snapshot.yaml
- Create: control-plane/tests/fixtures/smoke/workflow-contract.yaml

**Interfaces:**
- Produces: validate(schemaName, value) -> { valid, errors }.
- Schema names: source-snapshot, assessment-inputs, capability-mappings, assessment-context, capability-contract, approved-baseline, evidence-ledger, equivalence-proof, approval-record, part1-candidate-capability-map, architecture-pack-import, trace-set, target-baselines, pilot-scorecard, workflow-contract, policy-ir, source-inventory, capability-bundle, assessment-manifest.

- [ ] **Step 1: Write the failing schema loader test**

Create control-plane/tests/schemas.test.mjs:

~~~js
import assert from "node:assert/strict";
import { readFile } from "node:fs/promises";
import test from "node:test";
import YAML from "yaml";
import { validate } from "../src/validate.mjs";

test("pilot source and workflow fixtures validate", async () => {
  const source = YAML.parse(await readFile(new URL("./fixtures/smoke/source-snapshot.yaml", import.meta.url), "utf8"));
  const workflow = YAML.parse(await readFile(new URL("./fixtures/smoke/workflow-contract.yaml", import.meta.url), "utf8"));
  assert.deepEqual(validate("source-snapshot", source), { valid: true, errors: [] });
  assert.deepEqual(validate("workflow-contract", workflow), { valid: true, errors: [] });
});

test("unknown schema and missing required fields fail", () => {
  assert.throws(() => validate("unknown", {}), /unknown schema/);
  const result = validate("capability-contract", { apiVersion: "capability.ai-native.dev/v1alpha1" });
  assert.equal(result.valid, false);
  assert.ok(result.errors.some((error) => error.keyword === "required"));
});

test("invalid RFC 3339 calendar dates fail", async () => {
  const workflow = YAML.parse(await readFile(new URL("./fixtures/smoke/workflow-contract.yaml", import.meta.url), "utf8"));
  workflow.metadata.expiresAt = "2026-02-30T00:00:00Z";
  const result = validate("workflow-contract", workflow);
  assert.equal(result.valid, false);
  assert.ok(result.errors.some((error) => error.keyword === "format"));
});
~~~

- [ ] **Step 2: Run the test and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/schemas.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND for src/validate.mjs.

- [ ] **Step 3: Add the source snapshot schema**

Create control-plane/schemas/source-snapshot.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "source-snapshot/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "source", "servers"],
  "properties": {
    "schema_version": { "const": "source-snapshot/v1alpha1" },
    "source": {
      "type": "object",
      "additionalProperties": false,
      "required": ["type", "locator", "observed_at", "expires_at"],
      "properties": {
        "type": { "enum": ["official_registry", "private_registry", "live_schema_export", "host_config", "gateway_config"] },
        "locator": { "type": "string", "minLength": 1 },
        "observed_at": { "type": "string", "format": "date-time" },
        "expires_at": { "type": "string", "format": "date-time" }
      }
    },
    "servers": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["server_ref", "protocol_version", "endpoint_class", "endpoint_origin", "owner", "execution", "tools"],
        "properties": {
          "server_ref": { "type": "string", "minLength": 1 },
          "protocol_version": { "enum": ["2025-11-25", "2026-07-28"] },
          "endpoint_class": { "type": "string", "minLength": 1 },
          "endpoint_origin": { "type": "string", "minLength": 1 },
          "owner": { "type": "string", "minLength": 1 },
          "execution": { "type": "object" },
          "tools": {
            "type": "array",
            "items": {
              "type": "object",
              "additionalProperties": false,
              "required": ["name", "title", "description", "inputSchema", "outputSchema", "annotations", "execution", "auth"],
              "properties": {
                "name": { "type": "string", "minLength": 1 },
                "title": { "type": "string" },
                "description": { "type": "string" },
                "inputSchema": { "type": "object" },
                "outputSchema": { "type": "object" },
                "annotations": { "type": "object" },
                "execution": { "type": "object" },
                "auth": {
                  "type": "object",
                  "additionalProperties": false,
                  "required": ["mode", "audience", "scopes", "tenant_binding", "credential_passthrough"],
                  "properties": {
                    "mode": { "type": "string", "minLength": 1 },
                    "audience": { "type": "string", "minLength": 1 },
                    "scopes": { "type": "array", "items": { "type": "string" } },
                    "tenant_binding": { "enum": ["required", "not_required", "unknown"] },
                    "credential_passthrough": { "enum": ["forbidden", "detected", "unknown"] }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/capability-mappings.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "capability-mappings/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "mappings"],
  "properties": {
    "schema_version": { "const": "capability-mappings/v1alpha1" },
    "mappings": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["serverRef", "toolName", "capabilityId", "version", "owner", "status", "reviewedBy", "reviewedAt", "expiresAt", "actionClass", "destructive", "reversible", "idempotency", "readsPrivateData", "seesUntrustedContent", "canExternalizeData", "dataClasses", "successPredicate", "principalTypes", "delegationMode", "riskTier", "environments", "allowedAgents", "approvedAudience", "approvedScopes", "approvedTenantBinding", "approvedCredentialPassthrough", "approvedRegion", "verifiedProperties", "primary", "equivalenceGroup", "equivalenceProofRef", "testVectors", "evalSuite", "evidenceRefs"],
        "properties": {
          "serverRef": { "type": "string", "minLength": 1 },
          "toolName": { "type": "string", "minLength": 1 },
          "capabilityId": { "type": "string", "pattern": "^[a-z][a-z0-9.-]+$" },
          "version": { "type": "string", "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$" },
          "owner": { "type": "string", "minLength": 1 },
          "status": { "const": "candidate" },
          "reviewedBy": { "type": "string", "minLength": 1 },
          "reviewedAt": { "type": "string", "format": "date-time" },
          "expiresAt": { "type": "string", "format": "date-time" },
          "actionClass": { "enum": ["read", "internal_write", "external_write", "destructive", "administrative"] },
          "destructive": { "type": "boolean" },
          "reversible": { "type": "boolean" },
          "idempotency": { "enum": ["verified", "conditional", "none", "unknown"] },
          "readsPrivateData": { "type": "boolean" },
          "seesUntrustedContent": { "type": "boolean" },
          "canExternalizeData": { "type": "boolean" },
          "dataClasses": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
          "successPredicate": { "type": "string", "minLength": 1 },
          "principalTypes": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
          "delegationMode": { "type": "string", "minLength": 1 },
          "riskTier": { "enum": ["low", "medium", "high", "critical"] },
          "environments": { "type": "array", "items": { "type": "string" }, "minItems": 1, "uniqueItems": true },
          "allowedAgents": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
          "approvedAudience": { "type": "string", "minLength": 1 },
          "approvedScopes": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
          "approvedTenantBinding": { "const": "required" },
          "approvedCredentialPassthrough": { "const": "forbidden" },
          "approvedRegion": { "type": "string", "minLength": 1 },
          "verifiedProperties": { "type": "object" },
          "primary": { "type": "boolean" },
          "equivalenceGroup": { "type": ["string", "null"] },
          "equivalenceProofRef": { "type": ["string", "null"] },
          "testVectors": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
          "evalSuite": { "type": "string", "minLength": 1 },
          "evidenceRefs": { "type": "array", "items": { "type": "string" }, "minItems": 1 }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/assessment-context.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "assessment-context/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "subject", "agent", "tenant", "workflow", "stage", "purpose", "environment", "dataClasses", "region", "now"],
  "properties": {
    "schema_version": { "const": "assessment-context/v1alpha1" },
    "subject": { "type": "string", "minLength": 1 },
    "agent": { "type": "string", "minLength": 1 },
    "tenant": { "type": "string", "minLength": 1 },
    "workflow": { "type": "string", "pattern": "^[a-z][a-z0-9.-]+$" },
    "stage": { "type": "string", "pattern": "^[a-z][a-z0-9.-]+$" },
    "purpose": { "type": "string", "minLength": 1 },
    "environment": { "type": "string", "minLength": 1 },
    "dataClasses": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
    "region": { "type": "string", "minLength": 1 },
    "now": { "type": "string", "format": "date-time" }
  }
}
~~~

- [ ] **Step 4: Add the capability and workflow schemas**

Create control-plane/schemas/capability-contract.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "capability-contract/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["apiVersion", "kind", "metadata", "implementation", "semantics", "identity", "policy", "routing", "assurance"],
  "properties": {
    "apiVersion": { "const": "capability.ai-native.dev/v1alpha1" },
    "kind": { "const": "CapabilityContract" },
    "metadata": {
      "type": "object",
      "additionalProperties": false,
      "required": ["id", "version", "owner", "status", "reviewedAt", "expiresAt"],
      "properties": {
        "id": { "type": "string", "pattern": "^[a-z][a-z0-9.-]+$" },
        "version": { "type": "string", "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$" },
        "owner": { "type": "string", "minLength": 1 },
        "status": { "enum": ["candidate", "approved", "quarantined", "deprecated"] },
        "reviewedAt": { "type": "string", "format": "date-time" },
        "expiresAt": { "type": "string", "format": "date-time" }
      }
    },
    "implementation": {
      "type": "object",
      "additionalProperties": false,
      "required": ["serverRef", "protocolVersion", "toolName", "endpointClass", "inputSchemaHash", "outputSchemaHash", "sourceDescription", "sourceAnnotations", "verifiedProperties"],
      "properties": {
        "serverRef": { "type": "string", "minLength": 1 },
        "protocolVersion": { "enum": ["2025-11-25", "2026-07-28"] },
        "toolName": { "type": "string", "minLength": 1 },
        "endpointClass": { "type": "string", "minLength": 1 },
        "inputSchemaHash": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "outputSchemaHash": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "sourceDescription": { "type": "string" },
        "sourceAnnotations": { "type": "object" },
        "verifiedProperties": { "type": "object" }
      }
    },
    "semantics": {
      "type": "object",
      "additionalProperties": false,
      "required": ["actionClass", "destructive", "reversible", "idempotency", "readsPrivateData", "seesUntrustedContent", "canExternalizeData", "dataClasses", "successPredicate"],
      "properties": {
        "actionClass": { "enum": ["read", "internal_write", "external_write", "destructive", "administrative"] },
        "destructive": { "type": "boolean" },
        "reversible": { "type": "boolean" },
        "idempotency": { "enum": ["verified", "conditional", "none", "unknown"] },
        "readsPrivateData": { "type": "boolean" },
        "seesUntrustedContent": { "type": "boolean" },
        "canExternalizeData": { "type": "boolean" },
        "dataClasses": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
        "successPredicate": { "type": "string", "minLength": 1 }
      }
    },
    "identity": {
      "type": "object",
      "additionalProperties": false,
      "required": ["principalTypes", "delegationMode", "requiredScopes", "audience", "tenantBinding", "credentialPassthrough"],
      "properties": {
        "principalTypes": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
        "delegationMode": { "type": "string", "minLength": 1 },
        "requiredScopes": { "type": "array", "items": { "type": "string" } },
        "audience": { "type": "string", "minLength": 1 },
        "tenantBinding": { "const": "required" },
        "credentialPassthrough": { "const": "forbidden" }
      }
    },
    "policy": {
      "type": "object",
      "additionalProperties": false,
      "required": ["environments", "riskTier", "allowedAgents", "approval"],
      "properties": {
        "environments": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
        "riskTier": { "enum": ["low", "medium", "high", "critical"] },
        "allowedAgents": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
        "approval": {
          "type": "object",
          "additionalProperties": false,
          "required": ["required", "binds"],
          "properties": {
            "required": { "type": "boolean" },
            "binds": { "type": "array", "items": { "type": "string" } }
          }
        }
      }
    },
    "routing": {
      "type": "object",
      "additionalProperties": false,
      "required": ["primary", "equivalenceGroup", "equivalenceProofRef", "fallback", "retry"],
      "properties": {
        "primary": { "type": "boolean" },
        "equivalenceGroup": { "type": ["string", "null"] },
        "equivalenceProofRef": { "type": ["string", "null"] },
        "fallback": { "enum": ["forbidden", "verified_equivalence_only"] },
        "retry": {
          "type": "object",
          "additionalProperties": false,
          "required": ["transientOnly", "maxAttempts"],
          "properties": {
            "transientOnly": { "type": "boolean" },
            "maxAttempts": { "type": "integer", "minimum": 0, "maximum": 3 }
          }
        }
      }
    },
    "assurance": {
      "type": "object",
      "additionalProperties": false,
      "required": ["evidenceRefs", "testVectors", "evalSuite", "reviewedBy", "approvedImplementation", "approvedImplementationDigest"],
      "properties": {
        "evidenceRefs": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
        "testVectors": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
        "evalSuite": { "type": "string", "minLength": 1 },
        "reviewedBy": { "type": "string", "minLength": 1 },
        "approvedImplementation": {
          "type": "object",
          "additionalProperties": false,
          "required": ["server_ref", "protocol_version", "endpoint_class", "endpoint_origin", "server_owner", "server_execution", "name", "title", "description", "inputSchema", "outputSchema", "annotations", "tool_execution", "auth", "source_provenance_ref"],
          "properties": {
            "server_ref": { "type": "string", "minLength": 1 },
            "protocol_version": { "enum": ["2025-11-25", "2026-07-28"] },
            "endpoint_class": { "type": "string", "minLength": 1 },
            "endpoint_origin": { "type": "string", "minLength": 1 },
            "server_owner": { "type": "string", "minLength": 1 },
            "server_execution": { "type": "object" },
            "name": { "type": "string", "minLength": 1 },
            "title": { "type": "string" },
            "description": { "type": "string" },
            "inputSchema": { "type": "object" },
            "outputSchema": { "type": "object" },
            "annotations": { "type": "object" },
            "tool_execution": { "type": "object" },
            "auth": { "type": "object" },
            "source_provenance_ref": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
          }
        },
        "approvedImplementationDigest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
      }
    }
  }
}
~~~

Create control-plane/schemas/workflow-contract.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "workflow-contract/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["apiVersion", "kind", "metadata", "intent", "capabilities", "constraints", "assurance"],
  "properties": {
    "apiVersion": { "const": "capability.ai-native.dev/v1alpha1" },
    "kind": { "const": "WorkflowContract" },
    "metadata": {
      "type": "object",
      "additionalProperties": false,
      "required": ["workflowId", "stageId", "version", "owner", "status", "reviewedBy", "reviewedAt", "expiresAt"],
      "properties": {
        "workflowId": { "type": "string", "pattern": "^[a-z][a-z0-9.-]+$" },
        "stageId": { "type": "string", "pattern": "^[a-z][a-z0-9.-]+$" },
        "version": { "type": "string", "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$" },
        "owner": { "type": "string", "minLength": 1 },
        "status": { "const": "approved" },
        "reviewedBy": { "type": "string", "minLength": 1 },
        "reviewedAt": { "type": "string", "format": "date-time" },
        "expiresAt": { "type": "string", "format": "date-time" }
      }
    },
    "intent": {
      "type": "object",
      "additionalProperties": false,
      "required": ["actor", "agent", "purpose", "outcome"],
      "properties": {
        "actor": { "type": "string", "minLength": 1 },
        "agent": { "type": "string", "minLength": 1 },
        "purpose": { "type": "string", "minLength": 1 },
        "outcome": { "type": "string", "minLength": 1 }
      }
    },
    "capabilities": {
      "type": "object",
      "additionalProperties": false,
      "required": ["allowed", "prohibited"],
      "properties": {
        "allowed": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
        "prohibited": { "type": "array", "items": { "type": "string" }, "uniqueItems": true }
      }
    },
    "constraints": {
      "type": "object",
      "additionalProperties": false,
      "required": ["environment", "tenant", "dataClasses", "region", "approval", "fallback", "redactionProfile", "failureMode"],
      "properties": {
        "environment": { "type": "string", "minLength": 1 },
        "tenant": { "type": "string", "minLength": 1 },
        "dataClasses": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
        "region": { "type": "string", "minLength": 1 },
        "approval": { "enum": ["none", "required"] },
        "fallback": { "enum": ["forbidden", "verified_read_only_equivalence_only"] },
        "redactionProfile": { "type": "string", "minLength": 1 },
        "failureMode": { "const": "deny_and_escalate" }
      }
    },
    "assurance": {
      "type": "object",
      "additionalProperties": false,
      "required": ["auditLevel", "evalSuites"],
      "properties": {
        "auditLevel": { "enum": ["decision_only", "decision_and_summary"] },
        "evalSuites": { "type": "array", "items": { "type": "string" }, "minItems": 1 }
      }
    }
  }
}
~~~

- [ ] **Step 5: Add the Policy IR, source inventory, bundle, and assessment schemas**

Create control-plane/schemas/policy-ir.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "policy-ir/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["apiVersion", "kind", "metadata", "match", "effect", "capabilities", "constraints", "obligations"],
  "properties": {
    "apiVersion": { "const": "policy.ai-native.dev/v1alpha1" },
    "kind": { "const": "CapabilityPolicy" },
    "metadata": {
      "type": "object",
      "required": ["id", "version", "sourceDigest", "reviewedBy", "reviewedAt", "expiresAt"],
      "additionalProperties": false,
      "properties": {
        "id": { "type": "string", "minLength": 1 },
        "version": { "type": "string", "minLength": 1 },
        "sourceDigest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "reviewedBy": { "type": "string", "minLength": 1 },
        "reviewedAt": { "type": "string", "format": "date-time" },
        "expiresAt": { "type": "string", "format": "date-time" }
      }
    },
    "match": {
      "type": "object",
      "required": ["subjects", "agents", "workflows", "environments", "tenants"],
      "additionalProperties": false,
      "properties": {
        "subjects": { "type": "array", "items": { "type": "string" } },
        "agents": { "type": "array", "items": { "type": "string" } },
        "workflows": { "type": "array", "items": { "type": "string" } },
        "environments": { "type": "array", "items": { "type": "string" } },
        "tenants": { "type": "array", "items": { "type": "string" } }
      }
    },
    "effect": { "const": "allow" },
    "capabilities": {
      "type": "object",
      "required": ["include", "exclude"],
      "additionalProperties": false,
      "properties": {
        "include": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
        "exclude": { "type": "array", "items": { "type": "string" }, "uniqueItems": true }
      }
    },
    "constraints": {
      "type": "object",
      "required": ["purpose", "dataClasses", "region", "schemaStatus", "evidenceStatus", "approval", "fallback", "redactionProfile"],
      "additionalProperties": false,
      "properties": {
        "purpose": { "type": "string", "minLength": 1 },
        "dataClasses": { "type": "array", "items": { "type": "string" } },
        "region": { "type": "string" },
        "schemaStatus": { "const": "approved_only" },
        "evidenceStatus": { "const": "current_only" },
        "approval": { "enum": ["none", "required"] },
        "fallback": { "enum": ["forbidden", "verified_read_only_equivalence_only"] },
        "redactionProfile": { "type": "string", "minLength": 1 }
      }
    },
    "obligations": {
      "type": "object",
      "required": ["audit", "evaluations", "failureMode"],
      "additionalProperties": false,
      "properties": {
        "audit": { "type": "string" },
        "evaluations": { "type": "array", "items": { "type": "string" } },
        "failureMode": { "const": "deny_and_escalate" }
      }
    }
  }
}
~~~

Create control-plane/schemas/source-inventory.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "source-inventory/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "evidence", "servers"],
  "properties": {
    "schema_version": { "const": "source-inventory/v1alpha1" },
    "evidence": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["source_type", "locator", "observed_at", "expires_at", "parser", "digest", "provenance_digest"],
        "properties": {
          "source_type": { "type": "string", "minLength": 1 },
          "locator": { "type": "string", "minLength": 1 },
          "observed_at": { "type": "string", "format": "date-time" },
          "expires_at": { "type": "string", "format": "date-time" },
          "parser": { "type": "string", "minLength": 1 },
          "digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "provenance_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
        }
      }
    },
    "servers": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["server_ref", "protocol_version", "endpoint_class", "tools"],
        "properties": {
          "server_ref": { "type": "string", "minLength": 1 },
          "protocol_version": { "enum": ["2025-11-25", "2026-07-28"] },
          "endpoint_class": { "type": "string", "minLength": 1 },
          "tools": { "type": "array", "items": { "type": "string" }, "uniqueItems": true }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/capability-bundle.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "capability-bundle/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "context", "policy_digest", "expires_at", "included", "excluded"],
  "properties": {
    "schema_version": { "const": "capability-bundle/v1alpha1" },
    "context": {
      "type": "object",
      "required": ["subject", "agent", "tenant", "workflow", "stage", "purpose", "environment", "dataClasses", "region", "now"],
      "additionalProperties": false,
      "properties": {
        "subject": { "type": "string" },
        "agent": { "type": "string" },
        "tenant": { "type": "string" },
        "workflow": { "type": "string" },
        "stage": { "type": "string" },
        "purpose": { "type": "string" },
        "environment": { "type": "string" },
        "dataClasses": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
        "region": { "type": "string" },
        "now": { "type": "string", "format": "date-time" }
      }
    },
    "policy_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "expires_at": { "type": "string", "format": "date-time" },
    "included": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["capability_id", "contract_version", "implementation", "contract_digest", "approved_implementation_digest", "schema_hashes", "audience", "required_scopes", "region", "approval_required", "expires_at", "source_observed_at", "source_expires_at", "source_evidence_refs", "source_provenance_refs", "source_types", "evidence_refs", "evidence_digest", "evidence_expires_at", "eval_suite", "equivalence_proof_digest", "derived_digest"],
        "additionalProperties": false,
        "properties": {
          "capability_id": { "type": "string" },
          "contract_version": { "type": "string" },
          "implementation": { "type": "string" },
          "contract_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "approved_implementation_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "schema_hashes": { "type": "array", "items": { "type": "string" }, "minItems": 2, "maxItems": 2 },
          "audience": { "type": "string", "minLength": 1 },
          "required_scopes": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
          "region": { "type": "string", "minLength": 1 },
          "approval_required": { "type": "boolean" },
          "expires_at": { "type": "string", "format": "date-time" },
          "source_observed_at": { "type": "string", "format": "date-time" },
          "source_expires_at": { "type": "string", "format": "date-time" },
          "source_evidence_refs": { "type": "array", "items": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }, "minItems": 1, "uniqueItems": true },
          "source_provenance_refs": { "type": "array", "items": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }, "minItems": 1, "uniqueItems": true },
          "source_types": { "type": "array", "items": { "enum": ["official_registry", "private_registry", "live_schema_export", "host_config", "gateway_config"] }, "minItems": 1, "uniqueItems": true },
          "evidence_refs": { "type": "array", "items": { "type": "string" }, "minItems": 1 },
          "evidence_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "evidence_expires_at": { "type": "string", "format": "date-time" },
          "eval_suite": { "type": "string", "minLength": 1 },
          "equivalence_proof_digest": { "type": ["string", "null"], "pattern": "^sha256:[a-f0-9]{64}$" },
          "derived_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
        }
      }
    },
    "excluded": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["capability_id", "reason"],
        "additionalProperties": false,
        "properties": {
          "capability_id": { "type": "string" },
          "reason": { "type": "string" }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/assessment-manifest.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "assessment-manifest/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "generated_at", "input_digests", "artifact_digests", "source_inventory", "candidate_contracts", "approved_contracts", "architecture_import", "policy", "bundle", "target_artifacts", "drift_report", "evaluation_report", "pilot_scorecard", "pilot_authority_digests", "assurance_gaps", "explanations"],
  "properties": {
    "schema_version": { "const": "assessment-manifest/v1alpha1" },
    "generated_at": { "type": "string", "format": "date-time" },
    "input_digests": { "type": "object", "minProperties": 5, "additionalProperties": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" } },
    "artifact_digests": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "additionalProperties": false,
        "required": ["semantic_digest", "file_sha256"],
        "properties": {
          "semantic_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "file_sha256": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
        }
      }
    },
    "source_inventory": { "type": "string" },
    "candidate_contracts": { "type": "array", "items": { "type": "string" } },
    "approved_contracts": { "type": "array", "items": { "type": "string" } },
    "architecture_import": { "type": ["string", "null"] },
    "policy": { "type": "string" },
    "bundle": { "type": "string" },
    "target_artifacts": { "type": "array", "items": { "type": "string" } },
    "drift_report": { "type": "string" },
    "evaluation_report": { "type": "string" },
    "pilot_scorecard": { "type": "string" },
    "pilot_authority_digests": {
      "type": "object",
      "additionalProperties": false,
      "required": ["approved_contracts", "eligible_bundle", "drift_results", "target_artifacts"],
      "properties": {
        "approved_contracts": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "eligible_bundle": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "drift_results": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "target_artifacts": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
      }
    },
    "assurance_gaps": { "type": "string" },
    "explanations": { "type": "string" }
  }
}
~~~

- [ ] **Step 5A: Add schemas for independent authority, freshness, traces, and pilot evidence**

Create control-plane/schemas/assessment-inputs.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "assessment-inputs/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "mode", "sourceSnapshots", "approvedBaseline", "evidenceLedger", "equivalenceProofs", "mappings", "workflow", "context", "targets", "traces", "architecturePack"],
  "properties": {
    "schema_version": { "const": "assessment-inputs/v1alpha1" },
    "mode": { "enum": ["smoke", "pilot"] },
    "sourceSnapshots": { "type": "array", "minItems": 1, "uniqueItems": true, "items": { "$ref": "#/definitions/relativePath" } },
    "approvedBaseline": { "$ref": "#/definitions/relativePath" },
    "evidenceLedger": { "$ref": "#/definitions/relativePath" },
    "equivalenceProofs": { "$ref": "#/definitions/relativePath" },
    "mappings": { "$ref": "#/definitions/relativePath" },
    "workflow": { "$ref": "#/definitions/relativePath" },
    "context": { "$ref": "#/definitions/relativePath" },
    "targets": { "$ref": "#/definitions/relativePath" },
    "traces": { "oneOf": [{ "$ref": "#/definitions/relativePath" }, { "type": "null" }] },
    "architecturePack": { "oneOf": [{ "$ref": "#/definitions/relativePath" }, { "type": "null" }] }
  },
  "definitions": {
    "relativePath": { "type": "string", "minLength": 1, "pattern": "^(?!/)(?!.*(?:^|/)\\.\\.(?:/|$))[A-Za-z0-9._/-]+$" }
  }
}
~~~

Create control-plane/schemas/approved-baseline.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "approved-baseline/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "baseline_id", "reviewed_at", "contracts", "approvals"],
  "properties": {
    "schema_version": { "const": "approved-baseline/v1alpha1" },
    "baseline_id": { "type": "string", "minLength": 1 },
    "reviewed_at": { "type": "string", "format": "date-time" },
    "contracts": { "type": "array", "items": { "$ref": "capability-contract/v1alpha1" } },
    "approvals": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["approval_id", "contract_digest", "approved_implementation_digest", "reviewed_by", "reviewed_at", "expires_at", "status", "review_source"],
        "properties": {
          "approval_id": { "type": "string", "pattern": "^APR-[A-Za-z0-9._-]+$" },
          "contract_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "approved_implementation_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "reviewed_by": { "type": "string", "minLength": 1 },
          "reviewed_at": { "type": "string", "format": "date-time" },
          "expires_at": { "type": "string", "format": "date-time" },
          "status": { "const": "current" },
          "review_source": { "type": "string", "minLength": 1 }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/evidence-ledger.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "evidence-ledger/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "records"],
  "properties": {
    "schema_version": { "const": "evidence-ledger/v1alpha1" },
    "records": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["evidence_id", "kind", "subject_digest", "payload_digest", "observed_at", "expires_at", "reviewed_by", "status"],
        "properties": {
          "evidence_id": { "type": "string", "pattern": "^EVD-[A-Za-z0-9._-]+$" },
          "kind": { "enum": ["implementation", "contract_review", "evaluation", "trace"] },
          "subject_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "payload_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "observed_at": { "type": "string", "format": "date-time" },
          "expires_at": { "type": "string", "format": "date-time" },
          "reviewed_by": { "type": "string", "minLength": 1 },
          "status": { "const": "current" }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/equivalence-proof.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "equivalence-proof/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "proofs"],
  "properties": {
    "schema_version": { "const": "equivalence-proof/v1alpha1" },
    "proofs": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["proof_id", "version", "capability_id", "equivalence_group", "primary_contract_digest", "candidate_contract_digest", "dimension_digests", "eval_suite", "evidence_refs", "reviewed_by", "reviewed_at", "expires_at", "status"],
        "properties": {
          "proof_id": { "type": "string", "pattern": "^EQP-[A-Za-z0-9._-]+$" },
          "version": { "type": "string", "pattern": "^[0-9]+\\.[0-9]+\\.[0-9]+$" },
          "capability_id": { "type": "string", "minLength": 1 },
          "equivalence_group": { "type": "string", "minLength": 1 },
          "primary_contract_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "candidate_contract_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "dimension_digests": {
            "type": "object",
            "additionalProperties": false,
            "required": ["identity", "data_boundary", "side_effects", "schemas", "approval", "reversibility", "idempotency", "success_semantics"],
            "properties": {
              "identity": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "data_boundary": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "side_effects": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "schemas": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "approval": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "reversibility": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "idempotency": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
              "success_semantics": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
            }
          },
          "eval_suite": { "type": "string", "minLength": 1 },
          "evidence_refs": { "type": "array", "minItems": 1, "items": { "type": "string" } },
          "reviewed_by": { "type": "string", "minLength": 1 },
          "reviewed_at": { "type": "string", "format": "date-time" },
          "expires_at": { "type": "string", "format": "date-time" },
          "status": { "const": "current" }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/approval-record.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "approval-record/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "approval_id", "nonce", "user", "capability", "argument_hash", "purpose", "implementation", "contract_digest", "approved_implementation_digest", "equivalence_proof_digest", "schema_digest", "policy_digest", "tenant", "issued_at", "expires_at", "approval_digest"],
  "properties": {
    "schema_version": { "const": "approval-record/v1alpha1" },
    "approval_id": { "type": "string", "minLength": 1 },
    "nonce": { "type": "string", "pattern": "^[A-Za-z0-9_-]{16,128}$" },
    "user": { "type": "string", "minLength": 1 },
    "capability": { "type": "string", "minLength": 1 },
    "argument_hash": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "purpose": { "type": "string", "minLength": 1 },
    "implementation": { "type": "string", "minLength": 1 },
    "contract_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "approved_implementation_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "equivalence_proof_digest": { "type": ["string", "null"], "pattern": "^sha256:[a-f0-9]{64}$" },
    "schema_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "policy_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "tenant": { "type": "string", "minLength": 1 },
    "issued_at": { "type": "string", "format": "date-time" },
    "expires_at": { "type": "string", "format": "date-time" },
    "approval_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
  }
}
~~~

Create control-plane/schemas/part1-candidate-capability-map.schema.json by vendoring the exact Part I handoff schema rather than accepting a look-alike object:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "candidate-capability-map/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "status", "workflow_id", "owner", "capabilities"],
  "properties": {
    "schema_version": { "const": "candidate-capability-map/v1alpha1" },
    "status": { "const": "candidate_not_authorization" },
    "workflow_id": { "type": "string", "pattern": "^[a-z0-9][a-z0-9.-]*$" },
    "owner": { "type": "string", "minLength": 1 },
    "capabilities": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["capability_id", "workflow_stages", "purpose", "action_class", "data_classes", "identity", "permissions", "reliability", "implementation_candidates", "evidence_refs", "unresolved_questions", "owner"],
        "properties": {
          "capability_id": { "type": "string", "pattern": "^[a-z0-9][a-z0-9.-]*$" },
          "workflow_stages": { "type": "array", "minItems": 1, "items": { "type": "string", "minLength": 1 }, "uniqueItems": true },
          "purpose": { "type": "string", "minLength": 1 },
          "action_class": { "enum": ["read", "write", "destructive", "administrative"] },
          "data_classes": { "type": "array", "items": { "type": "string", "minLength": 1 }, "uniqueItems": true },
          "identity": {
            "type": "object",
            "additionalProperties": false,
            "required": ["principal_types", "tenant_binding", "credential_passthrough"],
            "properties": {
              "principal_types": { "type": "array", "minItems": 1, "items": { "type": "string", "minLength": 1 }, "uniqueItems": true },
              "tenant_binding": { "enum": ["required", "not_required", "unknown"] },
              "credential_passthrough": { "enum": ["forbidden", "not_required", "unknown"] }
            }
          },
          "permissions": {
            "type": "object",
            "additionalProperties": false,
            "required": ["requested_scopes", "approval_required", "approval_binding"],
            "properties": {
              "requested_scopes": { "type": "array", "items": { "type": "string", "minLength": 1 }, "uniqueItems": true },
              "approval_required": { "type": "boolean" },
              "approval_binding": { "type": "array", "items": { "type": "string", "minLength": 1 }, "uniqueItems": true }
            }
          },
          "reliability": {
            "type": "object",
            "additionalProperties": false,
            "required": ["idempotency", "reversibility", "fallback", "retry"],
            "properties": {
              "idempotency": { "enum": ["required", "conditional", "not_applicable", "unknown"] },
              "reversibility": { "enum": ["reversible", "compensatable", "irreversible", "unknown"] },
              "fallback": { "type": "string", "minLength": 1 },
              "retry": { "type": "string", "minLength": 1 }
            }
          },
          "implementation_candidates": {
            "type": "array",
            "minItems": 1,
            "items": {
              "type": "object",
              "additionalProperties": false,
              "required": ["implementation_type", "locator", "assurance", "evidence_refs"],
              "properties": {
                "implementation_type": { "enum": ["direct_api", "internal_service", "queue", "database", "mcp", "unknown"] },
                "locator": { "type": "string", "minLength": 1 },
                "assurance": { "enum": ["verified", "expected", "community_reported", "unknown"] },
                "evidence_refs": { "type": "array", "items": { "type": "string", "pattern": "^CLM-[0-9]{3,}$" }, "uniqueItems": true }
              }
            }
          },
          "evidence_refs": { "type": "array", "items": { "type": "string", "pattern": "^CLM-[0-9]{3,}$" }, "uniqueItems": true },
          "unresolved_questions": { "type": "array", "items": { "type": "string", "minLength": 1 } },
          "owner": { "type": "string", "minLength": 1 }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/architecture-pack-import.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "architecture-pack-import/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "source_pack_digest", "vocabulary_version", "candidate_only", "candidate_payload", "mapping_candidates", "workflow_candidates", "conversion_gaps", "candidate_refs", "unresolved_unknowns", "authorization_effect"],
  "properties": {
    "schema_version": { "const": "architecture-pack-import/v1alpha1" },
    "source_pack_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
    "vocabulary_version": { "const": "part1-v1alpha1-to-part2-v1alpha1" },
    "candidate_only": { "const": true },
    "candidate_payload": { "$ref": "candidate-capability-map/v1alpha1" },
    "mapping_candidates": {
      "type": "object",
      "additionalProperties": false,
      "required": ["schema_version", "status", "candidates"],
      "properties": {
        "schema_version": { "const": "part2-mapping-candidates/v1alpha1" },
        "status": { "const": "candidate_requires_independent_review" },
        "candidates": {
          "type": "array",
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": ["capability_id", "source_action_class", "proposed_fields", "implementation_candidates", "evidence_refs", "conversion_gap_codes"],
            "properties": {
              "capability_id": { "type": "string", "minLength": 1 },
              "source_action_class": { "enum": ["read", "write", "destructive", "administrative"] },
              "proposed_fields": { "type": "object" },
              "implementation_candidates": { "$ref": "candidate-capability-map/v1alpha1#/properties/capabilities/items/properties/implementation_candidates" },
              "evidence_refs": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
              "conversion_gap_codes": { "type": "array", "items": { "type": "string", "minLength": 1 }, "uniqueItems": true }
            }
          }
        }
      }
    },
    "workflow_candidates": {
      "type": "object",
      "additionalProperties": false,
      "required": ["schema_version", "status", "candidates"],
      "properties": {
        "schema_version": { "const": "part2-workflow-candidates/v1alpha1" },
        "status": { "const": "candidate_requires_independent_review" },
        "candidates": {
          "type": "array",
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": ["workflow_id", "stage_id", "capability_ids", "purpose_candidates", "owner_candidates", "missing_fields", "conversion_gap_codes"],
            "properties": {
              "workflow_id": { "type": "string", "minLength": 1 },
              "stage_id": { "type": "string", "minLength": 1 },
              "capability_ids": { "type": "array", "items": { "type": "string", "minLength": 1 }, "minItems": 1, "uniqueItems": true },
              "purpose_candidates": { "type": "array", "items": { "type": "string", "minLength": 1 }, "minItems": 1, "uniqueItems": true },
              "owner_candidates": { "type": "array", "items": { "type": "string", "minLength": 1 }, "minItems": 1, "uniqueItems": true },
              "missing_fields": { "type": "array", "items": { "type": "string", "minLength": 1 }, "minItems": 1, "uniqueItems": true },
              "conversion_gap_codes": { "type": "array", "items": { "type": "string", "minLength": 1 }, "minItems": 1, "uniqueItems": true }
            }
          }
        }
      }
    },
    "conversion_gaps": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["capability_id", "target", "code", "detail", "required_review"],
        "properties": {
          "capability_id": { "type": ["string", "null"] },
          "target": { "enum": ["mapping", "workflow", "implementation"] },
          "code": { "type": "string", "minLength": 1 },
          "detail": { "type": "string", "minLength": 1 },
          "required_review": { "const": true }
        }
      }
    },
    "candidate_refs": { "type": "array", "items": { "type": "string" }, "uniqueItems": true },
    "unresolved_unknowns": { "type": "array", "items": { "type": "string" } },
    "authorization_effect": { "const": "none" }
  }
}
~~~

Create control-plane/schemas/trace-set.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "trace-set/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "sanitized", "records"],
  "properties": {
    "schema_version": { "const": "trace-set/v1alpha1" },
    "sanitized": { "const": true },
    "records": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["trace_id", "suite_id", "case_id", "governance_surface", "runner", "source_ref", "tenant_ref", "workflow", "capability", "contract_digest", "action_class", "scenario", "selected_implementation", "selection_expected", "expected_decision", "actual_decision", "outcome", "failure_class", "attempt", "prompt_poisoning", "tool_poisoning", "observed_at", "result_digest"],
        "properties": {
          "trace_id": { "type": "string", "minLength": 1 },
          "suite_id": { "type": "string", "minLength": 1 },
          "case_id": { "type": "string", "minLength": 1 },
          "governance_surface": { "enum": ["host", "gateway"] },
          "runner": { "type": "string", "minLength": 1 },
          "source_ref": { "type": "string", "minLength": 1 },
          "tenant_ref": { "type": "string", "minLength": 1 },
          "workflow": { "type": "string", "minLength": 1 },
          "capability": { "type": "string", "minLength": 1 },
          "contract_digest": { "type": ["string", "null"], "pattern": "^sha256:[a-f0-9]{64}$" },
          "action_class": { "type": ["string", "null"], "enum": ["read", "internal_write", "external_write", "destructive", "administrative", null] },
          "scenario": { "enum": ["ordinary", "tool_selection", "prompt_poisoning", "tool_poisoning", "high_risk_approval", "high_risk_retry", "high_risk_fallback"] },
          "selected_implementation": { "type": ["string", "null"] },
          "selection_expected": { "type": ["string", "null"] },
          "expected_decision": { "enum": ["allow", "deny", "require_approval", "require_revalidation", "indeterminate"] },
          "actual_decision": { "enum": ["allow", "deny", "require_approval", "require_revalidation", "indeterminate"] },
          "outcome": { "enum": ["success", "failure", "unknown"] },
          "failure_class": { "enum": ["none", "transient_transport", "rate_limit", "business", "schema", "authorization", "unknown"] },
          "attempt": { "type": "integer", "minimum": 1 },
          "prompt_poisoning": { "type": "boolean" },
          "tool_poisoning": { "type": "boolean" },
          "observed_at": { "type": "string", "format": "date-time" },
          "result_digest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
        }
      }
    }
  }
}
~~~

Create control-plane/schemas/target-baselines.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "target-baselines/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "openclaw", "docker"],
  "properties": {
    "schema_version": { "const": "target-baselines/v1alpha1" },
    "openclaw": { "$ref": "#/definitions/target" },
    "docker": { "$ref": "#/definitions/target" }
  },
  "definitions": {
    "target": {
      "type": "object",
      "additionalProperties": false,
      "required": ["version", "currentConfig", "currentConfigDigest"],
      "properties": {
        "version": { "type": "string", "minLength": 1 },
        "currentConfig": { "type": "object" },
        "currentConfigDigest": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
      }
    }
  }
}
~~~

Create control-plane/schemas/pilot-scorecard.schema.json:

~~~json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "pilot-scorecard/v1alpha1",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "status", "authority_digests", "approved_contract_count", "eligible_read_count", "target_surfaces", "critical_quarantines", "high_risk_binding", "source_count", "source_classes", "server_count", "capability_count", "trace_count", "governance_surfaces", "scenario_results", "independent_suites", "critical_false_permits", "reasons"],
  "properties": {
    "schema_version": { "const": "pilot-scorecard/v1alpha1" },
    "status": { "enum": ["pass", "fail", "not_evaluated"] },
    "authority_digests": {
      "type": "object",
      "additionalProperties": false,
      "required": ["approved_contracts", "eligible_bundle", "drift_results", "target_artifacts"],
      "properties": {
        "approved_contracts": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "eligible_bundle": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "drift_results": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
        "target_artifacts": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }
      }
    },
    "approved_contract_count": { "type": "integer", "minimum": 0 },
    "eligible_read_count": { "type": "integer", "minimum": 0 },
    "target_surfaces": { "type": "array", "items": { "enum": ["host", "gateway"] }, "uniqueItems": true },
    "critical_quarantines": { "type": "integer", "minimum": 0 },
    "high_risk_binding": {
      "type": "object",
      "additionalProperties": false,
      "required": ["valid", "contract_digest", "action_class", "risk_tier", "scenarios"],
      "properties": {
        "valid": { "type": "boolean" },
        "contract_digest": { "type": ["string", "null"], "pattern": "^sha256:[a-f0-9]{64}$" },
        "action_class": { "type": ["string", "null"], "enum": ["read", "internal_write", "external_write", "destructive", "administrative", null] },
        "risk_tier": { "type": ["string", "null"], "enum": ["low", "medium", "high", "critical", null] },
        "scenarios": { "type": "array", "items": { "enum": ["high_risk_approval", "high_risk_retry", "high_risk_fallback"] }, "uniqueItems": true }
      }
    },
    "source_count": { "type": "integer", "minimum": 0 },
    "source_classes": { "type": "array", "items": { "enum": ["official_registry", "private_registry", "live_schema_export", "host_config", "gateway_config"] }, "uniqueItems": true },
    "server_count": { "type": "integer", "minimum": 0 },
    "capability_count": { "type": "integer", "minimum": 0 },
    "trace_count": { "type": "integer", "minimum": 0 },
    "governance_surfaces": { "type": "array", "items": { "enum": ["host", "gateway"] }, "uniqueItems": true },
    "scenario_results": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["scenario", "suite_ids", "case_count", "passed", "false_permits", "governance_surfaces", "contract_digests", "action_classes"],
        "properties": {
          "scenario": { "enum": ["tool_selection", "prompt_poisoning", "tool_poisoning", "high_risk_approval", "high_risk_retry", "high_risk_fallback"] },
          "suite_ids": { "type": "array", "items": { "type": "string", "minLength": 1 }, "minItems": 1, "uniqueItems": true },
          "case_count": { "type": "integer", "minimum": 1 },
          "passed": { "type": "boolean" },
          "false_permits": { "type": "integer", "minimum": 0 },
          "governance_surfaces": { "type": "array", "items": { "enum": ["host", "gateway"] }, "minItems": 1, "uniqueItems": true },
          "contract_digests": { "type": "array", "items": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" }, "uniqueItems": true },
          "action_classes": { "type": "array", "items": { "enum": ["read", "internal_write", "external_write", "destructive", "administrative"] }, "uniqueItems": true }
        }
      }
    },
    "independent_suites": { "type": "boolean" },
    "critical_false_permits": { "type": "integer", "minimum": 0 },
    "reasons": { "type": "array", "items": { "type": "string" } }
  }
}
~~~

- [ ] **Step 6: Implement the shared validator**

Create control-plane/src/validate.mjs:

~~~js
import { readFileSync } from "node:fs";
import { dirname, join, resolve } from "node:path";
import { fileURLToPath } from "node:url";
import Ajv from "ajv";
import addFormats from "ajv-formats";

const root = resolve(dirname(fileURLToPath(import.meta.url)), "..");
const names = [
  "source-snapshot", "assessment-inputs", "capability-mappings", "assessment-context",
  "capability-contract", "approved-baseline", "evidence-ledger", "equivalence-proof",
  "approval-record", "part1-candidate-capability-map", "architecture-pack-import", "trace-set", "target-baselines",
  "pilot-scorecard", "workflow-contract", "policy-ir", "source-inventory",
  "capability-bundle", "assessment-manifest"
];
const ajv = new Ajv({ allErrors: true, strict: false });
addFormats(ajv, { mode: "full", formats: ["date-time"] });
const schemas = new Map(names.map((name) => {
  const schema = JSON.parse(readFileSync(join(root, "schemas", name + ".schema.json"), "utf8"));
  return [name, schema];
}));
for (const schema of schemas.values()) ajv.addSchema(schema);
const validators = new Map([...schemas].map(([name, schema]) => [name, ajv.getSchema(schema.$id)]));

export function validate(name, value) {
  const validator = validators.get(name);
  if (!validator) throw new Error("unknown schema: " + name);
  const valid = validator(value);
  return {
    valid,
    errors: (validator.errors || []).map((error) => ({
      instancePath: error.instancePath,
      keyword: error.keyword,
      message: error.message
    }))
  };
}

export function assertValid(name, value) {
  const result = validate(name, value);
  if (!result.valid) throw new Error(name + " validation failed: " + JSON.stringify(result.errors));
  return value;
}
~~~

- [ ] **Step 7: Add the initial fixtures**

Create control-plane/tests/fixtures/smoke/source-snapshot.yaml:

~~~yaml
schema_version: source-snapshot/v1alpha1
source:
  type: live_schema_export
  locator: sanitized://support-tools
  observed_at: "2026-07-16T00:00:00Z"
  expires_at: "2026-07-23T00:00:00Z"
servers:
  - server_ref: com.example/support@1.0.0
    protocol_version: "2025-11-25"
    endpoint_class: corp-primary
    endpoint_origin: https://mcp.example.test/support
    owner: customer-support-platform
    execution: { transport: streamable-http, runtime: managed }
    tools:
      - name: get_case
        title: Get support case
        description: Read one authenticated support case.
        inputSchema:
          type: object
          required: [case_id]
          properties:
            case_id: { type: string }
        outputSchema:
          type: object
          properties:
            summary: { type: string }
        annotations:
          readOnlyHint: true
        execution: { timeout_ms: 3000, side_effect_class: none }
        auth:
          mode: oauth_authorization_code
          audience: https://mcp.example.com
          scopes: [case.read]
          tenant_binding: required
          credential_passthrough: forbidden
~~~

Create control-plane/tests/fixtures/smoke/workflow-contract.yaml:

~~~yaml
apiVersion: capability.ai-native.dev/v1alpha1
kind: WorkflowContract
metadata:
  workflowId: support-case-resolution
  stageId: inspect-case
  version: 1.0.0
  owner: customer-support
  status: approved
  reviewedBy: architecture-review-board
  reviewedAt: "2026-07-16T00:00:00Z"
  expiresAt: "2026-10-16T00:00:00Z"
intent:
  actor: support-agent
  agent: customer-support-assistant
  purpose: resolve an authenticated customer case
  outcome: produce an accurate case summary
capabilities:
  allowed: [customer.case.read]
  prohibited: [communication.email.send, customer.refund.execute]
constraints:
  environment: production
  tenant: tenant-a
  dataClasses: [customer-pii]
  region: eu
  approval: none
  fallback: verified_read_only_equivalence_only
  redactionProfile: customer-pii-summary
  failureMode: deny_and_escalate
assurance:
  auditLevel: decision_and_summary
  evalSuites: [support-read-boundary-v1]
~~~

- [ ] **Step 8: Run tests and commit**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
~~~

Expected: source and workflow fixtures validate, unknown schema throws, and incomplete capability contract fails.

Commit:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/schemas control-plane/src/validate.mjs control-plane/tests/schemas.test.mjs control-plane/tests/fixtures/smoke
git commit -m "feat: publish capability governance schemas"
~~~

---

### Task 4: Ingest sanitized snapshots and reject credential material

**Files:**
- Create: control-plane/src/evidence/load-snapshot.mjs
- Create: control-plane/src/evidence/sanitized-boundary.mjs
- Create: control-plane/tests/load-snapshot.test.mjs
- Create: control-plane/tests/fixtures/secret-snapshot.yaml
- Create: control-plane/tests/fixtures/secret-context.yaml

**Interfaces:**
- Produces: loadSnapshot(path) -> { snapshot, evidence }.
- Produces: snapshotFromDocument({ value, parser, digest }) -> { snapshot, evidence } so assessment input files pass through the bounded loader before source evidence is constructed.
- Produces: loadSanitizedDocument(path) -> { value, parser, digest } for mappings, workflow, context, and target baselines.
- Produces: loadSanitizedInput(rootDirectory, relativePath, schemaName) -> validated document; rejects traversal, symlinks, non-files, oversized input, malformed UTF-8, credential indicators, and schema-invalid content before returning data.
- Evidence contains source_type, locator, observed_at, expires_at, parser, digest, and provenance_digest.
- Secret detection is explicitly best-effort defense in depth, not proof of sanitization. Inputs remain subject to the declared sanitized-input boundary, size limits, no raw logging, and human data-handling controls.

- [ ] **Step 1: Write failing ingestion and secret tests**

Create control-plane/tests/load-snapshot.test.mjs:

~~~js
import assert from "node:assert/strict";
import { mkdir, mkdtemp, symlink, writeFile } from "node:fs/promises";
import { tmpdir } from "node:os";
import { join } from "node:path";
import test from "node:test";
import { assertNoCredentialIndicators, loadSanitizedDocument, loadSnapshot } from "../src/evidence/load-snapshot.mjs";
import { assertContainedRealpath, loadSanitizedInput } from "../src/evidence/sanitized-boundary.mjs";

test("snapshot ingestion binds provenance and digest", async () => {
  const path = new URL("./fixtures/smoke/source-snapshot.yaml", import.meta.url);
  const result = await loadSnapshot(path);
  assert.equal(result.snapshot.schema_version, "source-snapshot/v1alpha1");
  assert.equal(result.evidence.source_type, "live_schema_export");
  assert.match(result.evidence.digest, /^sha256:[a-f0-9]{64}$/);
  assert.match(result.evidence.provenance_digest, /^sha256:[a-f0-9]{64}$/);
  assert.equal(result.evidence.parser, "yaml@2.9.0");
});

test("credential-like source is rejected before persistence", async () => {
  const path = new URL("./fixtures/secret-snapshot.yaml", import.meta.url);
  await assert.rejects(() => loadSnapshot(path), /probable credential material/);
});

test("credential-like material in non-snapshot assessment inputs is also rejected", async () => {
  const path = new URL("./fixtures/secret-context.yaml", import.meta.url);
  await assert.rejects(() => loadSanitizedDocument(path), /probable credential material/);
});

test("common token, cookie, basic-auth, URL-userinfo, and generic key forms are rejected", async () => {
  const samples = [
    "ghp_abcdefghijklmnopqrstuvwxyz1234567890",
    "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.abcdefghijklmnopqrstuv",
    "Cookie: session=abcdefghijklmnopqrstuvwxyz012345",
    "Authorization: Basic dXNlcjpwYXNzd29yZA==",
    "https://user:supersecret@example.test/path",
    "api_key: abcdefghijklmnopqrstuvwxyz012345"
  ];
  for (const sample of samples) assert.throws(() => assertNoCredentialIndicators(sample), /probable credential material/);
});

test("sanitized boundary rejects traversal before parsing", async () => {
  const root = new URL("./fixtures/smoke/", import.meta.url);
  await assert.rejects(() => loadSanitizedInput(root, "../secret-context.yaml", "assessment-context"), /relative input path/);
});

test("sanitized boundary rejects a symlink in any ancestor component", { skip: process.platform === "win32" }, async () => {
  const parent = await mkdtemp(join(tmpdir(), "sanitized-boundary-"));
  const root = join(parent, "root");
  const outside = join(parent, "outside");
  await mkdir(root);
  await mkdir(outside);
  await writeFile(join(outside, "context.yaml"), "schema_version: assessment-context/v1alpha1\n", "utf8");
  await symlink(outside, join(root, "linked"), "dir");
  await assert.rejects(() => loadSanitizedInput(root, "linked/context.yaml", null), /symlink component/);
});

test("sanitized boundary rejects a final-component symlink", { skip: process.platform === "win32" }, async () => {
  const parent = await mkdtemp(join(tmpdir(), "sanitized-final-link-"));
  const root = join(parent, "root");
  await mkdir(root);
  const outside = join(parent, "outside.yaml");
  await writeFile(outside, "safe: true\n", "utf8");
  await symlink(outside, join(root, "input.yaml"));
  await assert.rejects(() => loadSanitizedInput(root, "input.yaml", null), /symlink component/);
});

test("final canonical path must remain under the canonical root", async () => {
  const parent = await mkdtemp(join(tmpdir(), "sanitized-realpath-"));
  const root = join(parent, "root");
  const outside = join(parent, "outside.yaml");
  await mkdir(root);
  await writeFile(outside, "safe: true\n", "utf8");
  assert.throws(() => assertContainedRealpath(root, outside), /realpath escapes input root/);
});

test("sanitized boundary rejects malformed UTF-8 and oversized input", async () => {
  const root = await mkdtemp(join(tmpdir(), "sanitized-bounds-"));
  await writeFile(join(root, "malformed.yaml"), Buffer.from([0xff]));
  await writeFile(join(root, "oversized.yaml"), Buffer.alloc(5 * 1024 * 1024 + 1, 0x20));
  await assert.rejects(() => loadSanitizedInput(root, "malformed.yaml", null), /encoded data|UTF-8/i);
  await assert.rejects(() => loadSanitizedInput(root, "oversized.yaml", null), /exceeds 5 MiB/);
});
~~~

Create control-plane/tests/fixtures/secret-snapshot.yaml:

~~~yaml
schema_version: source-snapshot/v1alpha1
secret: "-----BEGIN PRIVATE KEY-----"
~~~

Create control-plane/tests/fixtures/secret-context.yaml:

~~~yaml
schema_version: assessment-context/v1alpha1
purpose: "Bearer abcdefghijklmnopqrstuvwxyz012345"
~~~

- [ ] **Step 2: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/load-snapshot.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement read-only ingestion**

Create control-plane/src/evidence/load-snapshot.mjs:

~~~js
import { readFile } from "node:fs/promises";
import { fileURLToPath } from "node:url";
import YAML from "yaml";
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";

export const secretPatterns = [
  /-----BEGIN (?:RSA |EC |OPENSSH )?PRIVATE KEY-----/,
  /\bAKIA[0-9A-Z]{16}\b/,
  /\bsk-[A-Za-z0-9_-]{20,}\b/,
  /\b(?:ghp_[A-Za-z0-9]{20,}|github_pat_[A-Za-z0-9_]{20,}|glpat-[A-Za-z0-9_-]{20,})\b/i,
  /\beyJ[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\b/,
  /\bBearer\s+[A-Za-z0-9._~-]{20,}\b/i,
  /\bAuthorization\s*:\s*Basic\s+[A-Za-z0-9+/=]{12,}\b/i,
  /\b(?:Cookie|Set-Cookie)\s*:\s*[^\r\n]{12,}/i,
  /https?:\/\/[^\s/:@]+:[^\s/@]+@/i,
  /\b(?:api[_-]?key|client[_-]?secret|access[_-]?token|refresh[_-]?token|password)\s*[:=]\s*["']?[A-Za-z0-9._~+\/-]{16,}/i
];

export function assertNoCredentialIndicators(raw) {
  if (secretPatterns.some((pattern) => pattern.test(raw))) {
    throw new Error("probable credential material detected; ingestion stopped");
  }
}

export async function loadSanitizedDocument(pathOrUrl) {
  const raw = await readFile(pathOrUrl, "utf8");
  assertNoCredentialIndicators(raw);
  const path = pathOrUrl instanceof URL ? fileURLToPath(pathOrUrl) : String(pathOrUrl);
  const parser = path.endsWith(".json") ? "json:node-22" : "yaml@2.9.0";
  const value = path.endsWith(".json") ? JSON.parse(raw) : YAML.parse(raw);
  return { value, parser, digest: sha256(value) };
}

export async function loadSnapshot(pathOrUrl) {
  const document = await loadSanitizedDocument(pathOrUrl);
  return snapshotFromDocument(document);
}

export function snapshotFromDocument(document) {
  const snapshot = document.value;
  assertValid("source-snapshot", snapshot);
  if (Date.parse(snapshot.source.observed_at) >= Date.parse(snapshot.source.expires_at)) throw new Error("source evidence expiry must follow observation");
  return {
    snapshot,
    evidence: {
      source_type: snapshot.source.type,
      locator: snapshot.source.locator,
      observed_at: snapshot.source.observed_at,
      expires_at: snapshot.source.expires_at,
      parser: document.parser,
      digest: document.digest,
      provenance_digest: sha256({ source_type: snapshot.source.type, locator: snapshot.source.locator })
    }
  };
}
~~~

Create control-plane/src/evidence/sanitized-boundary.mjs:

~~~js
import { lstat, readFile, realpath } from "node:fs/promises";
import { isAbsolute, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";
import { TextDecoder } from "node:util";
import YAML from "yaml";
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";
import { assertNoCredentialIndicators } from "./load-snapshot.mjs";

const MAX_INPUT_BYTES = 5 * 1024 * 1024;
const decoder = new TextDecoder("utf-8", { fatal: true });

export async function loadSanitizedInput(rootDirectory, relativePath, schemaName) {
  const { path, stat } = await resolveSanitizedPath(rootDirectory, relativePath, "file");
  if (stat.size > MAX_INPUT_BYTES) throw new Error("sanitized input exceeds 5 MiB limit");
  const raw = decoder.decode(await readFile(path));
  assertNoCredentialIndicators(raw);
  const parser = path.endsWith(".json") ? "json:node-22" : "yaml@2.9.0";
  const value = path.endsWith(".json") ? JSON.parse(raw) : YAML.parse(raw);
  if (schemaName) assertValid(schemaName, value);
  return { value, parser, digest: sha256(value), path };
}

export async function resolveSanitizedPath(rootDirectory, relativePath, expectedType) {
  if (typeof relativePath !== "string" || relativePath.length === 0 || relativePath.startsWith("/") || relativePath.includes("\\") || relativePath.includes("\0")) {
    throw new Error("relative input path is required");
  }
  const segments = relativePath.split("/");
  if (segments.some((segment) => !segment || segment === "." || segment === "..")) throw new Error("relative input path is required");
  const lexicalRoot = resolve(rootDirectory instanceof URL ? fileURLToPath(rootDirectory) : String(rootDirectory));
  const root = await realpath(lexicalRoot);
  let cursor = root;
  let stat;
  for (const segment of segments) {
    cursor = resolve(cursor, segment);
    stat = await lstat(cursor);
    if (stat.isSymbolicLink()) throw new Error("sanitized input contains a symlink component");
  }
  const path = await realpath(cursor);
  assertContainedRealpath(root, path);
  stat = await lstat(path);
  if (expectedType === "file" && !stat.isFile()) throw new Error("sanitized input must be a regular file");
  if (expectedType === "directory" && !stat.isDirectory()) throw new Error("sanitized input must be a directory");
  return { root, path, stat };
}

export function assertContainedRealpath(root, path) {
  const fromRoot = relative(root, path);
  if (fromRoot === ".." || fromRoot.startsWith("../") || isAbsolute(fromRoot)) throw new Error("sanitized input realpath escapes input root");
}
~~~

- [ ] **Step 4: Run tests and commit**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
~~~

Expected: clean snapshot loads with digest and expiry; representative secret indicators, traversal, final symlinks, ancestor-directory symlinks, final realpath escapes, malformed UTF-8, and oversized files are rejected without echoing raw content; all tests pass.

Commit:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/src/evidence control-plane/tests/load-snapshot.test.mjs control-plane/tests/fixtures/secret-snapshot.yaml control-plane/tests/fixtures/secret-context.yaml
git commit -m "feat: ingest sanitized capability evidence"
~~~

---

### Task 5: Normalize tools through explicit organization mappings

**Files:**
- Create: control-plane/src/contracts/implementation-projection.mjs
- Create: control-plane/src/contracts/normalize-capability.mjs
- Create: control-plane/src/contracts/integrity.mjs
- Create: control-plane/src/contracts/store.mjs
- Create: control-plane/tests/normalize-capability.test.mjs
- Create: control-plane/tests/fixtures/smoke/mappings.yaml

**Interfaces:**
- Produces: normalizeCapability(server, tool, reviewedMapping, sourceEvidenceRef, sourceProvenanceRef) -> candidate CapabilityContract; normalization never returns status approved.
- Produces: protectedProjection(server, tool, sourceProvenanceRef) -> the exact implementation and provenance object bound to approval and drift evidence.
- Produces: assertContractIntegrity(contract) -> contract or throws on any schema/projection/semantic cross-field inconsistency.
- Produces: ContractStore with addApproved(contract, approvalRecord, baselineContext), get(id, implementationRef), implementations(id), and all(); one capability may have multiple prior-approved implementations.
- Requires exact serverRef + toolName mapping; no semantic auto-authorization.
- A mapping review approves the mapping proposal only. It is not a contract approval. ContractStore rejects candidates, self-generated approvals, same-run baselines, stale review records, and approval records whose contract or implementation digest differs.
- For a candidate, `assurance.approvedImplementation` is only the proposed protected projection retained for schema compatibility; its name and digest confer no authority. Only ContractStore plus the independent approval record can make that exact projection eligible.

- [ ] **Step 1: Write failing normalization tests**

Create control-plane/tests/normalize-capability.test.mjs:

~~~js
import assert from "node:assert/strict";
import { readFile } from "node:fs/promises";
import test from "node:test";
import YAML from "yaml";
import { sha256 } from "../src/canonical-json.mjs";
import { normalizeCapability } from "../src/contracts/normalize-capability.mjs";
import { ContractStore } from "../src/contracts/store.mjs";

const snapshot = YAML.parse(await readFile(new URL("./fixtures/smoke/source-snapshot.yaml", import.meta.url), "utf8"));
const mappings = YAML.parse(await readFile(new URL("./fixtures/smoke/mappings.yaml", import.meta.url), "utf8"));
const server = snapshot.servers[0];
const tool = server.tools[0];
const provenance = sha256({ source_type: snapshot.source.type, locator: snapshot.source.locator });

test("normalization separates source assertions from verified properties", () => {
  const contract = normalizeCapability(server, tool, mappings.mappings[0], "EVD-current", provenance);
  assert.equal(contract.metadata.id, "customer.case.read");
  assert.equal(contract.metadata.status, "candidate");
  assert.deepEqual(contract.implementation.sourceAnnotations, { readOnlyHint: true });
  assert.equal(contract.implementation.verifiedProperties.readOnly, true);
  assert.equal(contract.identity.credentialPassthrough, "forbidden");
  assert.equal(contract.assurance.approvedImplementationDigest, sha256(contract.assurance.approvedImplementation));
});

test("unmapped tool is rejected", () => {
  assert.throws(() => normalizeCapability(server, tool, null, "EVD-001", "PRV-001"), /explicit mapping/);
});

test("candidate and same-run baseline are never eligible", () => {
  const contract = normalizeCapability(server, tool, mappings.mappings[0], "EVD-current", provenance);
  const store = new ContractStore();
  assert.throws(() => store.addApproved(contract, {}, {}), /prior-approved contract/);
  const approved = structuredClone(contract);
  approved.metadata.status = "approved";
  const approval = {
    approval_id: "APR-support-read-v1",
    contract_digest: sha256(approved),
    approved_implementation_digest: approved.assurance.approvedImplementationDigest,
    reviewed_by: "architecture-review-board",
    reviewed_at: "2026-07-16T00:00:00Z",
    expires_at: "2026-10-16T00:00:00Z",
    status: "current",
    review_source: "sanitized://review/support-read-v1"
  };
  assert.throws(() => store.addApproved(approved, approval, {
    baselineReviewedAt: "2026-07-17T00:00:00Z",
    currentObservedAt: "2026-07-16T00:00:00Z",
    now: "2026-07-17T00:00:00Z"
  }), /predate current observation/);
});

test("alternate implementation cannot reuse another implementation approval", () => {
  const approved = normalizeCapability(server, tool, mappings.mappings[0], "EVD-current", provenance);
  approved.metadata.status = "approved";
  approved.metadata.reviewedAt = "2026-07-15T00:00:00Z";
  const approval = {
    approval_id: "APR-support-read-v1",
    contract_digest: sha256(approved),
    approved_implementation_digest: approved.assurance.approvedImplementationDigest,
    reviewed_by: "architecture-review-board",
    reviewed_at: "2026-07-15T00:00:00Z",
    expires_at: "2026-10-16T00:00:00Z",
    status: "current",
    review_source: "sanitized://review/support-read-v1"
  };
  const store = new ContractStore();
  store.addApproved(approved, approval, {
    baselineReviewedAt: "2026-07-15T00:00:00Z",
    currentObservedAt: "2026-07-16T00:00:00Z",
    now: "2026-07-17T00:00:00Z"
  });
  const alternate = structuredClone(approved);
  alternate.implementation.serverRef = "com.example/support-backup@1.0.0";
  alternate.assurance.approvedImplementation.server_ref = alternate.implementation.serverRef;
  alternate.assurance.approvedImplementationDigest = sha256(alternate.assurance.approvedImplementation);
  alternate.routing.primary = false;
  assert.throws(() => store.addApproved(alternate, approval, {
    baselineReviewedAt: "2026-07-15T00:00:00Z",
    currentObservedAt: "2026-07-16T00:00:00Z",
    now: "2026-07-17T00:00:00Z"
  }), /approval contract digest mismatch/);
});
~~~

- [ ] **Step 2: Add the explicit mapping fixture**

Create control-plane/tests/fixtures/smoke/mappings.yaml:

~~~yaml
schema_version: capability-mappings/v1alpha1
mappings:
  - serverRef: com.example/support@1.0.0
    toolName: get_case
    capabilityId: customer.case.read
    version: 1.0.0
    owner: customer-support
    status: candidate
    reviewedBy: architecture-review-board
    reviewedAt: "2026-07-16T00:00:00Z"
    expiresAt: "2026-10-16T00:00:00Z"
    actionClass: read
    destructive: false
    reversible: true
    idempotency: verified
    readsPrivateData: true
    seesUntrustedContent: true
    canExternalizeData: false
    dataClasses: [customer-pii]
    successPredicate: returns the requested tenant-bound case without mutation
    principalTypes: [human_delegated]
    delegationMode: oauth_authorization_code
    riskTier: medium
    environments: [production]
    allowedAgents: [customer-support-assistant]
    approvedAudience: https://mcp.example.com
    approvedScopes: [case.read]
    approvedTenantBinding: required
    approvedCredentialPassthrough: forbidden
    approvedRegion: eu
    verifiedProperties:
      readOnly: true
      region: eu
    primary: true
    equivalenceGroup: null
    equivalenceProofRef: null
    testVectors: [case-read-tenant-boundary]
    evalSuite: support-read-boundary-v1
    evidenceRefs: [EVD-support-implementation-v1, EVD-support-eval-v1]
~~~

- [ ] **Step 3: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/normalize-capability.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 4: Implement normalization**

Create control-plane/src/contracts/implementation-projection.mjs:

~~~js
export function protectedProjection(server, tool, sourceEvidenceRef) {
  return {
    server_ref: server.server_ref,
    protocol_version: server.protocol_version,
    endpoint_class: server.endpoint_class,
    endpoint_origin: server.endpoint_origin,
    server_owner: server.owner,
    server_execution: server.execution,
    name: tool.name,
    title: tool.title,
    description: tool.description,
    inputSchema: tool.inputSchema,
    outputSchema: tool.outputSchema,
    annotations: tool.annotations,
    tool_execution: tool.execution,
    auth: tool.auth,
    source_provenance_ref: sourceEvidenceRef
  };
}
~~~

Create control-plane/src/contracts/normalize-capability.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { protectedProjection } from "./implementation-projection.mjs";
import { assertValid } from "../validate.mjs";

export function normalizeCapability(server, tool, mapping, sourceEvidenceRef, sourceProvenanceRef) {
  if (!mapping || mapping.serverRef !== server.server_ref || mapping.toolName !== tool.name) {
    throw new Error("explicit mapping for serverRef and toolName is required");
  }
  assertValid("capability-mappings", { schema_version: "capability-mappings/v1alpha1", mappings: [mapping] });
  if (server.tools.filter((candidate) => candidate.name === tool.name).length !== 1) {
    throw new Error("tool identity must be unique within server");
  }
  const sourceScopes = [...tool.auth.scopes].sort();
  const approvedScopes = [...mapping.approvedScopes].sort();
  if (tool.auth.mode !== mapping.delegationMode
    || tool.auth.audience !== mapping.approvedAudience
    || JSON.stringify(sourceScopes) !== JSON.stringify(approvedScopes)
    || tool.auth.tenant_binding !== mapping.approvedTenantBinding
    || tool.auth.credential_passthrough !== mapping.approvedCredentialPassthrough) {
    throw new Error("source auth assertions do not match the reviewed mapping");
  }
  if (mapping.verifiedProperties.region !== mapping.approvedRegion) {
    throw new Error("verified region does not match the reviewed region");
  }
  const isWrite = mapping.actionClass !== "read";
  const approvedImplementation = protectedProjection(server, tool, sourceProvenanceRef);
  const contract = {
    apiVersion: "capability.ai-native.dev/v1alpha1",
    kind: "CapabilityContract",
    metadata: {
      id: mapping.capabilityId,
      version: mapping.version,
      owner: mapping.owner,
      status: "candidate",
      reviewedAt: mapping.reviewedAt,
      expiresAt: mapping.expiresAt
    },
    implementation: {
      serverRef: server.server_ref,
      protocolVersion: server.protocol_version,
      toolName: tool.name,
      endpointClass: server.endpoint_class,
      inputSchemaHash: sha256(tool.inputSchema),
      outputSchemaHash: sha256(tool.outputSchema),
      sourceDescription: tool.description,
      sourceAnnotations: tool.annotations,
      verifiedProperties: mapping.verifiedProperties
    },
    semantics: {
      actionClass: mapping.actionClass,
      destructive: mapping.destructive,
      reversible: mapping.reversible,
      idempotency: mapping.idempotency,
      readsPrivateData: mapping.readsPrivateData,
      seesUntrustedContent: mapping.seesUntrustedContent,
      canExternalizeData: mapping.canExternalizeData,
      dataClasses: mapping.dataClasses,
      successPredicate: mapping.successPredicate
    },
    identity: {
      principalTypes: mapping.principalTypes,
      delegationMode: mapping.delegationMode,
      requiredScopes: approvedScopes,
      audience: mapping.approvedAudience,
      tenantBinding: mapping.approvedTenantBinding,
      credentialPassthrough: mapping.approvedCredentialPassthrough
    },
    policy: {
      environments: [...mapping.environments].sort(),
      riskTier: mapping.riskTier,
      allowedAgents: mapping.allowedAgents,
      approval: {
        required: isWrite,
        binds: isWrite ? ["user", "capability", "normalized_argument_hash", "purpose", "implementation", "contract_digest", "approved_implementation_digest", "equivalence_proof_digest", "schema_digest", "policy_digest", "tenant", "nonce", "expiry", "approval_digest"] : []
      }
    },
    routing: {
      primary: mapping.primary,
      equivalenceGroup: mapping.equivalenceGroup,
      equivalenceProofRef: mapping.equivalenceProofRef,
      fallback: isWrite ? "forbidden" : "verified_equivalence_only",
      retry: {
        transientOnly: true,
        maxAttempts: isWrite ? 0 : 1
      }
    },
    assurance: {
      evidenceRefs: [...new Set([...mapping.evidenceRefs, sourceEvidenceRef])].sort(),
      testVectors: mapping.testVectors,
      evalSuite: mapping.evalSuite,
      reviewedBy: mapping.reviewedBy,
      approvedImplementation,
      approvedImplementationDigest: sha256(approvedImplementation)
    }
  };
  return assertValid("capability-contract", contract);
}
~~~

Create control-plane/src/contracts/integrity.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";

const WRITE_BINDINGS = [
  "user", "capability", "normalized_argument_hash", "purpose", "implementation",
  "contract_digest", "approved_implementation_digest", "equivalence_proof_digest",
  "schema_digest", "policy_digest", "tenant", "nonce", "expiry", "approval_digest"
];

function same(left, right) {
  return sha256(left) === sha256(right);
}

export function assertContractIntegrity(contract) {
  assertValid("capability-contract", contract);
  const approved = contract.assurance.approvedImplementation;
  const implementation = contract.implementation;
  const failures = [];
  if (contract.assurance.approvedImplementationDigest !== sha256(approved)) failures.push("approved implementation digest");
  if (implementation.serverRef !== approved.server_ref) failures.push("server reference");
  if (implementation.protocolVersion !== approved.protocol_version) failures.push("protocol version");
  if (implementation.toolName !== approved.name) failures.push("tool name");
  if (implementation.endpointClass !== approved.endpoint_class) failures.push("endpoint class");
  if (implementation.inputSchemaHash !== sha256(approved.inputSchema)) failures.push("input schema hash");
  if (implementation.outputSchemaHash !== sha256(approved.outputSchema)) failures.push("output schema hash");
  if (implementation.sourceDescription !== approved.description) failures.push("source description");
  if (!same(implementation.sourceAnnotations, approved.annotations)) failures.push("source annotations");
  if (contract.identity.delegationMode !== approved.auth.mode) failures.push("delegation mode");
  if (contract.identity.audience !== approved.auth.audience) failures.push("audience");
  if (!same([...contract.identity.requiredScopes].sort(), [...approved.auth.scopes].sort())) failures.push("scopes");
  if (contract.identity.tenantBinding !== approved.auth.tenant_binding) failures.push("tenant binding");
  if (contract.identity.credentialPassthrough !== approved.auth.credential_passthrough) failures.push("credential passthrough");
  if (implementation.verifiedProperties.region === undefined) failures.push("verified region");
  if (contract.semantics.readsPrivateData !== (contract.semantics.dataClasses.length > 0)) failures.push("private data semantics");
  if (contract.semantics.destructive !== (contract.semantics.actionClass === "destructive")) failures.push("destructive semantics");
  if (contract.semantics.actionClass === "read" && contract.semantics.canExternalizeData) failures.push("read externalization");
  if (contract.semantics.actionClass === "read" && (implementation.verifiedProperties.readOnly !== true || contract.semantics.idempotency !== "verified")) failures.push("read-only verification");
  if (contract.semantics.actionClass === "external_write" && !contract.semantics.canExternalizeData) failures.push("external write semantics");
  const isWrite = contract.semantics.actionClass !== "read";
  if (isWrite && !contract.policy.approval.required) failures.push("write approval");
  if (isWrite && contract.routing.fallback !== "forbidden") failures.push("write fallback");
  if (isWrite && !WRITE_BINDINGS.every((binding) => contract.policy.approval.binds.includes(binding))) failures.push("write approval bindings");
  if (Boolean(contract.routing.equivalenceGroup) !== Boolean(contract.routing.equivalenceProofRef)) failures.push("equivalence proof reference");
  if (failures.length) throw new Error("contract integrity failed: " + failures.join(", "));
  return contract;
}
~~~

Create control-plane/src/contracts/store.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { assertContractIntegrity } from "./integrity.mjs";

export class ContractStore {
  #contracts = new Map();

  addApproved(contract, approvalRecord, { baselineReviewedAt, currentObservedAt, now }) {
    assertContractIntegrity(contract);
    if (contract.metadata.status !== "approved") throw new Error("prior-approved contract is required");
    if (!approvalRecord || approvalRecord.contract_digest !== sha256(contract)) throw new Error("approval contract digest mismatch");
    if (approvalRecord.approved_implementation_digest !== contract.assurance.approvedImplementationDigest) throw new Error("approval implementation digest mismatch");
    if (approvalRecord.status !== "current") throw new Error("approval record is not current");
    if (approvalRecord.reviewed_by !== contract.assurance.reviewedBy || approvalRecord.reviewed_at !== contract.metadata.reviewedAt) throw new Error("approval reviewer or review time mismatch");
    if (approvalRecord.expires_at !== contract.metadata.expiresAt) throw new Error("approval and contract expiry mismatch");
    if (Date.parse(approvalRecord.reviewed_at) >= Date.parse(currentObservedAt)) throw new Error("approval must predate current observation");
    if (Date.parse(baselineReviewedAt) >= Date.parse(currentObservedAt)) throw new Error("baseline must predate current observation");
    if (Date.parse(approvalRecord.expires_at) <= Date.parse(now)) throw new Error("approval record has expired");
    const implementationRef = contract.implementation.serverRef + "#" + contract.implementation.toolName;
    const implementations = this.#contracts.get(contract.metadata.id) || new Map();
    if (implementations.has(implementationRef)) throw new Error("duplicate implementation: " + implementationRef);
    implementations.set(implementationRef, structuredClone(contract));
    this.#contracts.set(contract.metadata.id, implementations);
  }

  get(id, implementationRef) {
    const implementations = this.#contracts.get(id);
    if (!implementations) return null;
    if (!implementationRef && implementations.size !== 1) throw new Error("implementationRef is required for multi-implementation capability: " + id);
    const value = implementationRef ? implementations.get(implementationRef) : implementations.values().next().value;
    return value ? structuredClone(value) : null;
  }

  implementations(id) {
    return [...(this.#contracts.get(id)?.values() || [])].map((value) => structuredClone(value));
  }

  all() {
    return [...this.#contracts.values()].flatMap((implementations) => [...implementations.values()]).map((value) => structuredClone(value));
  }
}
~~~

- [ ] **Step 5: Run tests and commit**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
~~~

Expected: candidate normalization, assertion separation, unmapped-tool rejection, prior-baseline timing, digest-bound approval, alternate-implementation rejection, and prior tests pass.

Commit:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/src/contracts control-plane/tests/normalize-capability.test.mjs control-plane/tests/fixtures/smoke/mappings.yaml
git commit -m "feat: normalize candidate capabilities and protect approved baselines"
~~~

---

### Task 5A: Load an independent approved baseline and import Part I candidates

**Files:**
- Create: control-plane/src/contracts/load-approved-baseline.mjs
- Create: control-plane/src/import/architecture-pack.mjs
- Create: control-plane/tests/authority-inputs.test.mjs

**Interfaces:**
- Produces: loadApprovedBaseline(baseline, currentEvidence, now) -> ContractStore containing only contracts with a separate, prior, current approval record.
- Produces: importArchitecturePack(packDirectory) -> architecture-pack-import/v1alpha1.
- Part I `candidate-capability-map` content remains candidate-only, preserves unresolved questions and pack provenance, and has `authorization_effect: none`; it is never inserted into ContractStore and never compiled directly into Policy IR.
- Promotion is deliberately out of band: a reviewer must resolve every conversion gap and submit ordinary Part II mapping/workflow inputs plus a separate prior approval in a later run. The importer exposes no same-run promotion path.

- [ ] **Step 1: Write failing authority-boundary tests**

Create control-plane/tests/authority-inputs.test.mjs:

~~~js
import assert from "node:assert/strict";
import { mkdir, mkdtemp, symlink, writeFile } from "node:fs/promises";
import { tmpdir } from "node:os";
import { join } from "node:path";
import test from "node:test";
import YAML from "yaml";
import { sha256 } from "../src/canonical-json.mjs";
import { loadApprovedBaseline } from "../src/contracts/load-approved-baseline.mjs";
import { convertPart1Candidates, importArchitecturePack, sha256Text } from "../src/import/architecture-pack.mjs";

function part1Candidate() {
  return {
    schema_version: "candidate-capability-map/v1alpha1",
    status: "candidate_not_authorization",
    workflow_id: "support-assistant",
    owner: "customer-support",
    capabilities: [{
      capability_id: "knowledge.search",
      workflow_stages: ["draft-response"],
      purpose: "Retrieve approved support knowledge for a draft response.",
      action_class: "read",
      data_classes: ["internal-knowledge"],
      identity: { principal_types: ["service"], tenant_binding: "required", credential_passthrough: "forbidden" },
      permissions: { requested_scopes: ["knowledge.read"], approval_required: false, approval_binding: [] },
      reliability: {
        idempotency: "not_applicable",
        reversibility: "reversible",
        fallback: "escalate to manual knowledge lookup",
        retry: "transient failures only, maximum two attempts"
      },
      implementation_candidates: [{
        implementation_type: "internal_service",
        locator: "services/knowledge-search",
        assurance: "expected",
        evidence_refs: ["CLM-001"]
      }],
      evidence_refs: ["CLM-001"],
      unresolved_questions: ["Confirm tenant-isolation behavior before Part II review."],
      owner: "knowledge-platform"
    }]
  };
}

test("empty first-run baseline has no eligible contracts", () => {
  const baseline = {
    schema_version: "approved-baseline/v1alpha1",
    baseline_id: "first-run",
    reviewed_at: "2026-07-15T00:00:00Z",
    contracts: [],
    approvals: []
  };
  const store = loadApprovedBaseline(baseline, [{ observed_at: "2026-07-16T00:00:00Z" }], "2026-07-17T00:00:00Z");
  assert.deepEqual(store.all(), []);
});

test("a shallow Part I look-alike is rejected by the exact handoff schema", () => {
  assert.throws(() => convertPart1Candidates({
    schema_version: "candidate-capability-map/v1alpha1",
    status: "candidate_not_authorization",
    workflow_id: "support-assistant",
    owner: "support",
    capabilities: [{ capability_id: "knowledge.search", unresolved_questions: [] }]
  }), /part1-candidate-capability-map validation failed/);
});

test("ambiguous Part I vocabulary stays null and requires independent review", () => {
  const candidate = part1Candidate();
  const capability = candidate.capabilities[0];
  capability.action_class = "write";
  capability.identity.tenant_binding = "unknown";
  capability.identity.credential_passthrough = "not_required";
  capability.reliability.idempotency = "required";
  capability.reliability.reversibility = "compensatable";
  const result = convertPart1Candidates(candidate);
  const proposed = result.mapping_candidates.candidates[0].proposed_fields;
  assert.equal(proposed.actionClass, null);
  assert.equal(proposed.approvedTenantBinding, null);
  assert.equal(proposed.approvedCredentialPassthrough, null);
  assert.equal(proposed.idempotency, null);
  assert.equal(proposed.reversible, null);
  for (const code of ["ambiguous_action_class", "tenant_binding_review_required", "credential_passthrough_review_required", "idempotency_evidence_required", "ambiguous_reversibility"]) {
    assert.ok(result.conversion_gaps.some((gap) => gap.code === code), code);
  }
});

test("Part I candidate map has no authorization effect", async () => {
  const root = await mkdtemp(join(tmpdir(), "part1-pack-"));
  const candidateValue = part1Candidate();
  const candidate = YAML.stringify(candidateValue);
  await writeFile(join(root, "candidate-capability-map.yaml"), candidate, "utf8");
  const manifest = {
    schema_version: "architecture-pack/v1alpha1",
    pack_id: "support-assistant",
    status: "draft",
    generated_at: "2026-07-15T00:00:00Z",
    source_skill: { name: "ai-native-architect", version: "0.2.0", bundle_digest: "sha256:" + "a".repeat(64) },
    host: { name: "fixture", version: "1.0.0", mode: "project-aware" },
    stages: Array.from({ length: 8 }, (_, index) => ({ stage_id: "stage-" + index, status: "not_started" })),
    artifacts: [{ logical_id: "candidate-capability-map", kind: "yaml", location: "candidate-capability-map.yaml", digest: sha256Text(candidate), required_for_review: true }],
    unresolved_unknowns: ["verify tenant isolation"],
    approvals: []
  };
  await writeFile(join(root, "manifest.yaml"), YAML.stringify(manifest), "utf8");
  const result = await importArchitecturePack(root);
  assert.equal(result.candidate_only, true);
  assert.equal(result.authorization_effect, "none");
  assert.deepEqual(result.candidate_refs, ["knowledge.search"]);
  assert.deepEqual(result.candidate_payload, candidateValue);
  assert.equal(result.mapping_candidates.status, "candidate_requires_independent_review");
  assert.equal(result.mapping_candidates.candidates[0].proposed_fields.actionClass, "read");
  assert.deepEqual(result.mapping_candidates.candidates[0].implementation_candidates, candidateValue.capabilities[0].implementation_candidates);
  assert.ok(result.conversion_gaps.some((gap) => gap.code === "implementation_binding_required"));
  assert.ok(result.conversion_gaps.some((gap) => gap.code === "workflow_contract_fields_missing"));
  assert.equal(result.workflow_candidates.status, "candidate_requires_independent_review");
});

test("Part I import rejects an artifact reached through an ancestor symlink", { skip: process.platform === "win32" }, async () => {
  const root = await mkdtemp(join(tmpdir(), "part1-pack-link-"));
  const actual = join(root, "actual");
  await mkdir(actual);
  const candidate = YAML.stringify(part1Candidate());
  await writeFile(join(actual, "candidate-capability-map.yaml"), candidate, "utf8");
  await symlink(actual, join(root, "linked"), "dir");
  const manifest = {
    schema_version: "architecture-pack/v1alpha1",
    pack_id: "support-assistant",
    status: "draft",
    generated_at: "2026-07-15T00:00:00Z",
    source_skill: { name: "ai-native-architect", version: "0.2.0", bundle_digest: "sha256:" + "a".repeat(64) },
    host: { name: "fixture", version: "1.0.0", mode: "project-aware" },
    stages: Array.from({ length: 8 }, (_, index) => ({ stage_id: "stage-" + index, status: "not_started" })),
    artifacts: [{ logical_id: "candidate-capability-map", kind: "yaml", location: "linked/candidate-capability-map.yaml", digest: sha256Text(candidate), required_for_review: true }],
    unresolved_unknowns: [],
    approvals: []
  };
  await writeFile(join(root, "manifest.yaml"), YAML.stringify(manifest), "utf8");
  await assert.rejects(() => importArchitecturePack(root), /symlink component/);
});

test("a Part I approval still cannot become a Part II contract approval", async () => {
  const result = { candidate_only: true, authorization_effect: "none" };
  assert.notEqual(sha256(result), "");
  assert.equal(result.authorization_effect, "none");
});
~~~

- [ ] **Step 2: Implement prior-baseline loading**

Create control-plane/src/contracts/load-approved-baseline.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";
import { ContractStore } from "./store.mjs";

export function loadApprovedBaseline(baseline, currentEvidence, now) {
  assertValid("approved-baseline", baseline);
  if (!Array.isArray(currentEvidence) || currentEvidence.length === 0) throw new Error("current source evidence is required");
  const observedTimes = currentEvidence.map((item) => Date.parse(item.observed_at));
  if (observedTimes.some((value) => !Number.isFinite(value))) throw new Error("current observation time is invalid");
  const earliestCurrentObservation = new Date(Math.min(...observedTimes)).toISOString();
  const approvals = new Map(baseline.approvals.map((item) => [item.contract_digest, item]));
  const store = new ContractStore();
  for (const contract of baseline.contracts) {
    const approval = approvals.get(sha256(contract));
    if (!approval) throw new Error("approved contract lacks an independent approval record: " + contract.metadata.id);
    store.addApproved(contract, approval, {
      baselineReviewedAt: baseline.reviewed_at,
      currentObservedAt: earliestCurrentObservation,
      now
    });
  }
  return store;
}
~~~

- [ ] **Step 3: Implement the candidate-only Part I importer**

Create control-plane/src/import/architecture-pack.mjs:

~~~js
import { createHash } from "node:crypto";
import { readFile } from "node:fs/promises";
import YAML from "yaml";
import { sha256 } from "../canonical-json.mjs";
import { assertNoCredentialIndicators } from "../evidence/load-snapshot.mjs";
import { resolveSanitizedPath } from "../evidence/sanitized-boundary.mjs";
import { assertValid } from "../validate.mjs";

export const PART1_TO_PART2_VOCABULARY_VERSION = "part1-v1alpha1-to-part2-v1alpha1";

export function sha256Text(value) {
  return "sha256:" + createHash("sha256").update(value, "utf8").digest("hex");
}

function distinct(values) {
  return [...new Set(values)].sort();
}

function convertAction(value, addGap) {
  if (value === "read" || value === "destructive" || value === "administrative") return value;
  addGap("mapping", "ambiguous_action_class", "Part I write must be independently classified as internal_write or external_write.");
  return null;
}

function convertIdempotency(value, addGap) {
  if (value === "conditional") return "conditional";
  if (value === "unknown") {
    addGap("mapping", "idempotency_unknown", "Part I reports unknown idempotency; Part II needs implementation evidence.");
    return "unknown";
  }
  addGap("mapping", "idempotency_evidence_required", "Part I reliability intent is not verification of Part II implementation idempotency.");
  return null;
}

function convertReversibility(value, addGap) {
  if (value === "reversible") return true;
  if (value === "irreversible") return false;
  addGap("mapping", "ambiguous_reversibility", "Compensatable or unknown reversibility cannot be reduced to the Part II boolean without review.");
  return null;
}

function convertTenantBinding(value, addGap) {
  if (value === "required") return "required";
  addGap("mapping", "tenant_binding_review_required", "Part II approval requires verified tenant binding; Part I does not establish it.");
  return null;
}

function convertCredentialPassthrough(value, addGap) {
  if (value === "forbidden") return "forbidden";
  addGap("mapping", "credential_passthrough_review_required", "Part II approval requires verified forbidden credential passthrough.");
  return null;
}

export function convertPart1Candidates(candidate) {
  assertValid("part1-candidate-capability-map", candidate);
  const conversionGaps = [];
  const mappingCandidates = candidate.capabilities.map((capability) => {
    const localCodes = [];
    const addGap = (target, code, detail) => {
      conversionGaps.push({ capability_id: capability.capability_id, target, code, detail, required_review: true });
      localCodes.push(code);
    };
    const actionClass = convertAction(capability.action_class, addGap);
    const proposedFields = {
      actionClass,
      destructive: capability.action_class === "destructive",
      reversible: convertReversibility(capability.reliability.reversibility, addGap),
      idempotency: convertIdempotency(capability.reliability.idempotency, addGap),
      dataClasses: [...capability.data_classes],
      principalTypes: [...capability.identity.principal_types],
      approvedScopes: [...capability.permissions.requested_scopes],
      approvedTenantBinding: convertTenantBinding(capability.identity.tenant_binding, addGap),
      approvedCredentialPassthrough: convertCredentialPassthrough(capability.identity.credential_passthrough, addGap),
      approvalRequired: capability.permissions.approval_required,
      approvalBindings: [...capability.permissions.approval_binding],
      purpose: capability.purpose,
      owner: capability.owner
    };
    addGap("implementation", "implementation_binding_required", "A Part II reviewer must bind an observed serverRef and toolName; no Part I locator is auto-promoted or parsed as authority.");
    return {
      capability_id: capability.capability_id,
      source_action_class: capability.action_class,
      proposed_fields: proposedFields,
      implementation_candidates: structuredClone(capability.implementation_candidates),
      evidence_refs: distinct([...capability.evidence_refs, ...capability.implementation_candidates.flatMap((item) => item.evidence_refs)]),
      conversion_gap_codes: distinct(localCodes)
    };
  });

  const missingWorkflowFields = [
    "metadata.version", "metadata.status", "metadata.reviewedBy", "metadata.reviewedAt", "metadata.expiresAt",
    "intent.actor", "intent.agent", "intent.outcome", "constraints.environment", "constraints.tenant",
    "constraints.region", "constraints.redactionProfile", "constraints.failureMode", "assurance"
  ];
  const stages = new Map();
  for (const capability of candidate.capabilities) {
    for (const stageId of capability.workflow_stages) {
      const entry = stages.get(stageId) || { capabilities: [], purposes: [], owners: [] };
      entry.capabilities.push(capability.capability_id);
      entry.purposes.push(capability.purpose);
      entry.owners.push(capability.owner);
      stages.set(stageId, entry);
      conversionGaps.push({
        capability_id: capability.capability_id,
        target: "workflow",
        code: "workflow_contract_fields_missing",
        detail: "Part I stage membership lacks independently reviewed Part II workflow identity, constraints, expiry, and assurance fields.",
        required_review: true
      });
    }
  }
  const workflowCandidates = [...stages].sort(([left], [right]) => left.localeCompare(right)).map(([stageId, entry]) => ({
    workflow_id: candidate.workflow_id,
    stage_id: stageId,
    capability_ids: distinct(entry.capabilities),
    purpose_candidates: distinct(entry.purposes),
    owner_candidates: distinct(entry.owners),
    missing_fields: missingWorkflowFields,
    conversion_gap_codes: ["workflow_contract_fields_missing"]
  }));

  return {
    mapping_candidates: {
      schema_version: "part2-mapping-candidates/v1alpha1",
      status: "candidate_requires_independent_review",
      candidates: mappingCandidates
    },
    workflow_candidates: {
      schema_version: "part2-workflow-candidates/v1alpha1",
      status: "candidate_requires_independent_review",
      candidates: workflowCandidates
    },
    conversion_gaps: conversionGaps
  };
}

export async function importArchitecturePack(packDirectory) {
  const { root, path: manifestPath, stat: manifestStat } = await resolveSanitizedPath(packDirectory, "manifest.yaml", "file");
  if (manifestStat.size > 5 * 1024 * 1024) throw new Error("Part I manifest must be a bounded regular non-symlink file");
  const manifestRaw = await readFile(manifestPath, "utf8");
  assertNoCredentialIndicators(manifestRaw);
  const manifest = YAML.parse(manifestRaw);
  if (manifest?.schema_version !== "architecture-pack/v1alpha1"
    || manifest?.source_skill?.name !== "ai-native-architect"
    || !["draft", "partial_inline", "ready_for_review", "approved", "superseded"].includes(manifest?.status)
    || !Array.isArray(manifest.stages)
    || manifest.stages.length !== 8
    || !Array.isArray(manifest.artifacts)
    || !Array.isArray(manifest.approvals)
    || !Array.isArray(manifest.unresolved_unknowns)) {
    throw new Error("invalid Part I Architecture Pack manifest");
  }
  const candidateArtifact = manifest.artifacts.find((item) => item.logical_id === "candidate-capability-map");
  if (!candidateArtifact) throw new Error("Part I pack lacks candidate-capability-map");
  const contents = new Map();
  for (const artifact of manifest.artifacts) {
    let raw;
    if (artifact.location.startsWith("inline:")) {
      if (typeof artifact.inline_content !== "string") throw new Error("inline Part I artifact lacks content");
      raw = artifact.inline_content;
    } else {
      const { path: artifactPath, stat } = await resolveSanitizedPath(root, artifact.location, "file");
      if (stat.size > 5 * 1024 * 1024) throw new Error("Part I artifact must be a bounded regular non-symlink file");
      raw = await readFile(artifactPath, "utf8");
    }
    assertNoCredentialIndicators(raw);
    if (sha256Text(raw) !== artifact.digest) throw new Error("Part I artifact digest mismatch: " + artifact.logical_id);
    contents.set(artifact.logical_id, raw);
  }
  const candidate = YAML.parse(contents.get("candidate-capability-map"));
  assertValid("part1-candidate-capability-map", candidate);
  const converted = convertPart1Candidates(candidate);
  return assertValid("architecture-pack-import", {
    schema_version: "architecture-pack-import/v1alpha1",
    source_pack_digest: sha256(manifest),
    vocabulary_version: PART1_TO_PART2_VOCABULARY_VERSION,
    candidate_only: true,
    candidate_payload: structuredClone(candidate),
    ...converted,
    candidate_refs: candidate.capabilities.map((item) => item.capability_id).sort(),
    unresolved_unknowns: [...new Set([...manifest.unresolved_unknowns, ...candidate.capabilities.flatMap((item) => item.unresolved_questions || [])])].sort(),
    authorization_effect: "none"
  });
}
~~~

- [ ] **Step 4: Run tests and commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
cd "$REPOSITORY_ROOT"
git add control-plane/src/contracts/load-approved-baseline.mjs control-plane/src/import/architecture-pack.mjs control-plane/tests/authority-inputs.test.mjs
git commit -m "feat: separate approved authority from architecture candidates"
~~~

Expected: an empty baseline has no eligible contracts, same-run approvals fail, and verified Part I content remains candidate-only.

---

### Task 6: Compile Workflow Contracts into deterministic Policy IR

**Files:**
- Create: control-plane/src/compile/policy.mjs
- Create: control-plane/tests/policy.test.mjs

**Interfaces:**
- Produces: compilePolicy(workflowContract) -> CapabilityPolicy.
- Rejects overlap between allowed and prohibited capabilities.

- [ ] **Step 1: Write failing policy compiler tests**

Create control-plane/tests/policy.test.mjs:

~~~js
import assert from "node:assert/strict";
import { readFile } from "node:fs/promises";
import test from "node:test";
import YAML from "yaml";
import { sha256 } from "../src/canonical-json.mjs";
import { compilePolicy } from "../src/compile/policy.mjs";

const workflow = YAML.parse(await readFile(new URL("./fixtures/smoke/workflow-contract.yaml", import.meta.url), "utf8"));

test("workflow compiles into deterministic Policy IR", () => {
  const policy = compilePolicy(workflow);
  assert.equal(policy.kind, "CapabilityPolicy");
  assert.deepEqual(policy.capabilities.include, ["customer.case.read"]);
  assert.deepEqual(policy.capabilities.exclude, ["communication.email.send", "customer.refund.execute"]);
  assert.deepEqual(policy.match.tenants, ["tenant-a"]);
  assert.equal(policy.metadata.sourceDigest, sha256(workflow));
  assert.equal(policy.constraints.purpose, workflow.intent.purpose);
  assert.equal(policy.obligations.failureMode, "deny_and_escalate");
});

test("allow and prohibit overlap fails closed", () => {
  const invalid = structuredClone(workflow);
  invalid.capabilities.prohibited.push("customer.case.read");
  assert.throws(() => compilePolicy(invalid), /both allowed and prohibited/);
});
~~~

- [ ] **Step 2: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/policy.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement compilePolicy**

Create control-plane/src/compile/policy.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";

export function compilePolicy(workflow) {
  assertValid("workflow-contract", workflow);
  const allowed = [...workflow.capabilities.allowed].sort();
  const prohibited = [...workflow.capabilities.prohibited].sort();
  const overlap = allowed.filter((id) => prohibited.includes(id));
  if (overlap.length) throw new Error("capability is both allowed and prohibited: " + overlap.join(", "));

  const policy = {
    apiVersion: "policy.ai-native.dev/v1alpha1",
    kind: "CapabilityPolicy",
    metadata: {
      id: workflow.metadata.workflowId + "-" + workflow.metadata.stageId,
      version: workflow.metadata.version,
      sourceDigest: sha256(workflow),
      reviewedBy: workflow.metadata.reviewedBy,
      reviewedAt: workflow.metadata.reviewedAt,
      expiresAt: workflow.metadata.expiresAt
    },
    match: {
      subjects: [workflow.intent.actor],
      agents: [workflow.intent.agent],
      workflows: [workflow.metadata.workflowId + "/" + workflow.metadata.stageId],
      environments: [workflow.constraints.environment],
      tenants: [workflow.constraints.tenant]
    },
    effect: "allow",
    capabilities: {
      include: allowed,
      exclude: prohibited
    },
    constraints: {
      purpose: workflow.intent.purpose,
      dataClasses: [...workflow.constraints.dataClasses].sort(),
      region: workflow.constraints.region,
      schemaStatus: "approved_only",
      evidenceStatus: "current_only",
      approval: workflow.constraints.approval,
      fallback: workflow.constraints.fallback,
      redactionProfile: workflow.constraints.redactionProfile
    },
    obligations: {
      audit: workflow.assurance.auditLevel,
      evaluations: [...workflow.assurance.evalSuites].sort(),
      failureMode: workflow.constraints.failureMode
    }
  };
  return assertValid("policy-ir", policy);
}
~~~

- [ ] **Step 4: Run tests and commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
cd "$REPOSITORY_ROOT"
git add control-plane/src/compile/policy.mjs control-plane/tests/policy.test.mjs
git commit -m "feat: compile workflow capability policy"
~~~

Expected: all tests pass before commit.

---

### Task 7: Compile the minimum eligible capability bundle

**Files:**
- Create: control-plane/src/compile/bundle.mjs
- Create: control-plane/src/evidence/freshness.mjs
- Create: control-plane/tests/bundle.test.mjs

**Interfaces:**
- Produces: compileBundle(policy, approvedStore, context, evidenceLedger, eligibilityByImplementation, equivalenceProofs) -> CapabilityBundle.
- Produces: deriveBundleItem(policy, contract, evidenceSummary, sourceEligibility, equivalenceProof) and verifyBundleItem(item, policy, contract, evidenceLedger, sourceEligibility, equivalenceProof, now); simulation uses the same derivation rather than trusting serialized bundle fields.
- Exclusion reasons are exact strings: prohibited, missing_contract, missing_primary, not_approved, contract_expired, source_stale, drift_quarantined, evidence_missing, evidence_stale, evidence_subject_mismatch, equivalence_not_proven, agent_mismatch, environment_mismatch, tenant_unknown, credential_passthrough, region_mismatch, approval_mismatch, or data_class_mismatch.

- [ ] **Step 1: Write failing bundle tests**

Create control-plane/tests/bundle.test.mjs:

~~~js
import assert from "node:assert/strict";
import { readFile } from "node:fs/promises";
import test from "node:test";
import YAML from "yaml";
import { sha256 } from "../src/canonical-json.mjs";
import { compilePolicy } from "../src/compile/policy.mjs";
import { compileBundle, verifyBundleItem } from "../src/compile/bundle.mjs";
import { normalizeCapability } from "../src/contracts/normalize-capability.mjs";
import { ContractStore } from "../src/contracts/store.mjs";

const snapshot = YAML.parse(await readFile(new URL("./fixtures/smoke/source-snapshot.yaml", import.meta.url), "utf8"));
const workflow = YAML.parse(await readFile(new URL("./fixtures/smoke/workflow-contract.yaml", import.meta.url), "utf8"));
const mappings = YAML.parse(await readFile(new URL("./fixtures/smoke/mappings.yaml", import.meta.url), "utf8"));
const contract = normalizeCapability(snapshot.servers[0], snapshot.servers[0].tools[0], mappings.mappings[0], "EVD-current", sha256({ source_type: snapshot.source.type, locator: snapshot.source.locator }));
contract.metadata.status = "approved";
contract.metadata.reviewedAt = "2026-07-15T00:00:00Z";
contract.assurance.evidenceRefs = [...mappings.mappings[0].evidenceRefs].sort();
const policy = compilePolicy(workflow);
const context = {
  subject: "support-agent",
  agent: "customer-support-assistant",
  tenant: "tenant-a",
  workflow: "support-case-resolution",
  stage: "inspect-case",
  purpose: "resolve an authenticated customer case",
  environment: "production",
  dataClasses: ["customer-pii"],
  region: "eu",
  now: "2026-07-17T00:00:00Z"
};
const approval = {
  approval_id: "APR-support-read-v1",
  contract_digest: sha256(contract),
  approved_implementation_digest: contract.assurance.approvedImplementationDigest,
  reviewed_by: "architecture-review-board",
  reviewed_at: "2026-07-15T00:00:00Z",
  expires_at: "2026-10-16T00:00:00Z",
  status: "current",
  review_source: "sanitized://review/support-read-v1"
};
const store = new ContractStore();
store.addApproved(contract, approval, {
  baselineReviewedAt: "2026-07-15T00:00:00Z",
  currentObservedAt: snapshot.source.observed_at,
  now: context.now
});
const evidenceLedger = {
  schema_version: "evidence-ledger/v1alpha1",
  records: contract.assurance.evidenceRefs.map((evidence_id, index) => ({
    evidence_id,
    kind: index === 0 ? "implementation" : "evaluation",
    subject_digest: index === 0
      ? contract.assurance.approvedImplementationDigest
      : sha256({ approved_implementation_digest: contract.assurance.approvedImplementationDigest, eval_suite: contract.assurance.evalSuite }),
    payload_digest: "sha256:" + String(index + 1).repeat(64),
    observed_at: "2026-07-15T00:00:00Z",
    expires_at: "2026-10-16T00:00:00Z",
    reviewed_by: "architecture-review-board",
    status: "current"
  }))
};
const implementationRef = contract.implementation.serverRef + "#" + contract.implementation.toolName;
const sourceEligibility = {
  drift_status: "current",
  source_observed_at: snapshot.source.observed_at,
  source_expires_at: snapshot.source.expires_at,
  source_evidence_refs: ["sha256:" + "1".repeat(64)],
  source_provenance_refs: [sha256({ source_type: snapshot.source.type, locator: snapshot.source.locator })],
  source_types: [snapshot.source.type]
};
const eligibility = { [implementationRef]: sourceEligibility };

test("only eligible approved contract enters the bundle", () => {
  const bundle = compileBundle(policy, store, context, evidenceLedger, eligibility, []);
  assert.deepEqual(bundle.included.map((item) => item.capability_id), ["customer.case.read"]);
  assert.equal(bundle.included[0].contract_digest, sha256(contract));
  assert.equal(bundle.included[0].approved_implementation_digest, contract.assurance.approvedImplementationDigest);
  assert.equal(verifyBundleItem(bundle.included[0], policy, contract, evidenceLedger, eligibility[implementationRef], null, context.now).valid, true);
  assert.deepEqual(bundle.excluded, [
    { capability_id: "communication.email.send", reason: "prohibited" },
    { capability_id: "customer.refund.execute", reason: "prohibited" }
  ]);
});

test("expired reviewed evidence is excluded", () => {
  const staleLedger = structuredClone(evidenceLedger);
  staleLedger.records[0].expires_at = "2026-07-16T12:00:00Z";
  const bundle = compileBundle(policy, store, context, staleLedger, eligibility, []);
  assert.ok(bundle.excluded.some((item) => item.reason === "evidence_stale"));
});

test("context that does not match the compiled policy is rejected", () => {
  const wrongTenant = { ...context, tenant: "tenant-b" };
  assert.throws(() => compileBundle(policy, store, wrongTenant, evidenceLedger, eligibility, []), /context does not match policy/);
});

test("serialized bundle field tampering is detected by complete re-derivation", () => {
  const bundle = compileBundle(policy, store, context, evidenceLedger, eligibility, []);
  for (const field of ["approval_required", "audience", "region", "eval_suite", "evidence_digest"]) {
    const changed = structuredClone(bundle.included[0]);
    changed[field] = field === "approval_required" ? !changed[field] : "tampered";
    assert.equal(verifyBundleItem(changed, policy, contract, evidenceLedger, eligibility[implementationRef], null, context.now).valid, false, field);
  }
});

test("first-run store and quarantined observations cannot enter a bundle", () => {
  const firstRun = compileBundle(policy, new ContractStore(), context, evidenceLedger, eligibility, []);
  assert.equal(firstRun.included.length, 0);
  assert.ok(firstRun.excluded.some((item) => item.reason === "missing_contract"));
  const quarantined = { [implementationRef]: { ...sourceEligibility, drift_status: "quarantined" } };
  const quarantinedBundle = compileBundle(policy, store, context, evidenceLedger, quarantined, []);
  assert.equal(quarantinedBundle.included.length, 0);
  assert.ok(quarantinedBundle.excluded.some((item) => item.reason === "drift_quarantined"));
});
~~~

- [ ] **Step 2: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/bundle.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement evidence freshness and deterministic eligibility**

Create control-plane/src/evidence/freshness.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";

export function summarizeEvidence(contract, ledger, now) {
  assertValid("evidence-ledger", ledger);
  const byId = new Map(ledger.records.map((record) => [record.evidence_id, record]));
  const records = [...contract.assurance.evidenceRefs].sort().map((reference) => byId.get(reference));
  if (records.some((record) => !record)) return { valid: false, reason: "evidence_missing" };
  if (records.some((record) => record.status !== "current" || Date.parse(record.observed_at) > Date.parse(now) || Date.parse(record.expires_at) <= Date.parse(now) || Date.parse(record.observed_at) >= Date.parse(record.expires_at))) return { valid: false, reason: "evidence_stale" };
  if (records.some((record) => {
    const expected = record.kind === "contract_review"
      ? sha256(contract)
      : record.kind === "evaluation"
        ? sha256({ approved_implementation_digest: contract.assurance.approvedImplementationDigest, eval_suite: contract.assurance.evalSuite })
        : contract.assurance.approvedImplementationDigest;
    return record.subject_digest !== expected;
  })) return { valid: false, reason: "evidence_subject_mismatch" };
  if (!records.some((record) => record.kind === "implementation") || !records.some((record) => record.kind === "evaluation")) return { valid: false, reason: "evidence_missing" };
  return {
    valid: true,
    records,
    digest: sha256(records),
    expires_at: new Date(Math.min(...records.map((record) => Date.parse(record.expires_at)))).toISOString()
  };
}
~~~

Create control-plane/src/compile/bundle.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { summarizeEvidence } from "../evidence/freshness.mjs";
import { assertValid } from "../validate.mjs";

function implementationRef(contract) {
  return contract.implementation.serverRef + "#" + contract.implementation.toolName;
}

function proofFor(contract, proofDocument, now) {
  if (!contract.routing.equivalenceProofRef) return { valid: true, proof: null };
  const proof = proofDocument.proofs.find((item) => item.proof_id === contract.routing.equivalenceProofRef);
  if (!proof || proof.status !== "current" || Date.parse(proof.expires_at) <= Date.parse(now)
    || proof.capability_id !== contract.metadata.id
    || proof.equivalence_group !== contract.routing.equivalenceGroup
    || ![proof.primary_contract_digest, proof.candidate_contract_digest].includes(sha256(contract))) {
    return { valid: false, proof: null };
  }
  return { valid: true, proof };
}

function exclusionReason(contract, policy, context, evidence, eligibility, proofState) {
  if (!contract) return "missing_contract";
  if (contract.metadata.status !== "approved") return "not_approved";
  if (Date.parse(contract.metadata.expiresAt) <= Date.parse(context.now)) return "contract_expired";
  if (!eligibility || Date.parse(eligibility.source_observed_at) > Date.parse(context.now) || Date.parse(eligibility.source_expires_at) <= Date.parse(context.now)) return "source_stale";
  if (eligibility.drift_status !== "current") return "drift_quarantined";
  if (!evidence.valid) return evidence.reason;
  if (!proofState.valid) return "equivalence_not_proven";
  if (!contract.policy.allowedAgents.includes(context.agent)) return "agent_mismatch";
  if (!contract.policy.environments.includes(context.environment)) return "environment_mismatch";
  if (contract.identity.tenantBinding !== "required" || !context.tenant) return "tenant_unknown";
  if (contract.identity.credentialPassthrough !== "forbidden") return "credential_passthrough";
  if (contract.implementation.verifiedProperties.region !== context.region) return "region_mismatch";
  if (policy.constraints.approval === "none" && contract.policy.approval.required) return "approval_mismatch";
  if (!contract.semantics.dataClasses.every((dataClass) => policy.constraints.dataClasses.includes(dataClass))) return "data_class_mismatch";
  return null;
}

export function deriveBundleItem(policy, contract, evidence, eligibility, equivalenceProof) {
  const base = {
    capability_id: contract.metadata.id,
    contract_version: contract.metadata.version,
    implementation: implementationRef(contract),
    contract_digest: sha256(contract),
    approved_implementation_digest: contract.assurance.approvedImplementationDigest,
    schema_hashes: [contract.implementation.inputSchemaHash, contract.implementation.outputSchemaHash],
    audience: contract.identity.audience,
    required_scopes: [...contract.identity.requiredScopes].sort(),
    region: contract.implementation.verifiedProperties.region,
    approval_required: policy.constraints.approval === "required" || contract.policy.approval.required,
    expires_at: new Date(Math.min(Date.parse(contract.metadata.expiresAt), Date.parse(evidence.expires_at), Date.parse(eligibility.source_expires_at))).toISOString(),
    source_observed_at: eligibility.source_observed_at,
    source_expires_at: eligibility.source_expires_at,
    source_evidence_refs: [...eligibility.source_evidence_refs].sort(),
    source_provenance_refs: [...eligibility.source_provenance_refs].sort(),
    source_types: [...eligibility.source_types].sort(),
    evidence_refs: [...contract.assurance.evidenceRefs].sort(),
    evidence_digest: evidence.digest,
    evidence_expires_at: evidence.expires_at,
    eval_suite: contract.assurance.evalSuite,
    equivalence_proof_digest: equivalenceProof ? sha256(equivalenceProof) : null
  };
  return { ...base, derived_digest: sha256(base) };
}

export function verifyBundleItem(item, policy, contract, evidenceLedger, eligibility, equivalenceProof, now) {
  const evidence = summarizeEvidence(contract, evidenceLedger, now);
  if (!evidence.valid) return { valid: false, reason: evidence.reason };
  if (!eligibility || eligibility.drift_status !== "current" || Date.parse(eligibility.source_observed_at) > Date.parse(now) || Date.parse(eligibility.source_expires_at) <= Date.parse(now)) return { valid: false, reason: "source_or_drift_not_current" };
  const expected = deriveBundleItem(policy, contract, evidence, eligibility, equivalenceProof);
  const valid = sha256(item) === sha256(expected);
  return { valid, reason: valid ? null : "bundle_contract_binding_mismatch" };
}

export function compileBundle(policy, approvedStore, context, evidenceLedger, eligibilityByImplementation, equivalenceProofs) {
  assertValid("policy-ir", policy);
  assertValid("assessment-context", { schema_version: "assessment-context/v1alpha1", ...context });
  assertValid("equivalence-proof", { schema_version: "equivalence-proof/v1alpha1", proofs: equivalenceProofs });
  const workflowStage = context.workflow + "/" + context.stage;
  if (!policy.match.subjects.includes(context.subject)
    || !policy.match.agents.includes(context.agent)
    || !policy.match.tenants.includes(context.tenant)
    || !policy.match.environments.includes(context.environment)
    || !policy.match.workflows.includes(workflowStage)
    || policy.constraints.purpose !== context.purpose
    || policy.constraints.region !== context.region
    || sha256([...policy.constraints.dataClasses].sort()) !== sha256([...context.dataClasses].sort())) throw new Error("context does not match policy");
  if (Date.parse(policy.metadata.expiresAt) <= Date.parse(context.now)) throw new Error("policy has expired");
  const contracts = approvedStore.all();
  const byId = new Map();
  for (const contract of contracts) {
    const values = byId.get(contract.metadata.id) || [];
    values.push(contract);
    byId.set(contract.metadata.id, values);
  }
  const included = [];
  const excluded = policy.capabilities.exclude.map((capability_id) => ({ capability_id, reason: "prohibited" }));
  const proofDocument = { proofs: equivalenceProofs };
  for (const capabilityId of policy.capabilities.include) {
    const candidates = byId.get(capabilityId) || [];
    const primaries = candidates.filter((contract) => contract.routing.primary);
    if (primaries.length > 1) throw new Error("multiple primary implementations: " + capabilityId);
    const contract = primaries[0];
    if (!contract) {
      excluded.push({ capability_id: capabilityId, reason: candidates.length ? "missing_primary" : "missing_contract" });
      continue;
    }
    const evidence = summarizeEvidence(contract, evidenceLedger, context.now);
    const eligibility = eligibilityByImplementation[implementationRef(contract)];
    const proofState = proofFor(contract, proofDocument, context.now);
    const reason = exclusionReason(contract, policy, context, evidence, eligibility, proofState);
    if (reason) {
      excluded.push({ capability_id: capabilityId, reason });
      continue;
    }
    included.push(deriveBundleItem(policy, contract, evidence, eligibility, proofState.proof));
  }
  const bundle = {
    schema_version: "capability-bundle/v1alpha1",
    context: { ...context, dataClasses: [...context.dataClasses].sort() },
    policy_digest: sha256(policy),
    expires_at: new Date(Math.min(Date.parse(policy.metadata.expiresAt), ...included.map((item) => Date.parse(item.expires_at)))).toISOString(),
    included: included.sort((left, right) => left.capability_id.localeCompare(right.capability_id)),
    excluded: excluded.sort((left, right) => left.capability_id.localeCompare(right.capability_id))
  };
  return assertValid("capability-bundle", bundle);
}
~~~

- [ ] **Step 4: Run tests and commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
cd "$REPOSITORY_ROOT"
git add control-plane/src/compile/bundle.mjs control-plane/tests/bundle.test.mjs
git commit -m "feat: compile minimum capability bundles"
~~~

Expected: all tests pass before commit.

---

### Task 8: Compile explicit OpenClaw and Docker target artifacts

**Files:**
- Create: control-plane/src/adapters/openclaw.mjs
- Create: control-plane/src/adapters/docker.mjs
- Create: control-plane/tests/adapters.test.mjs
- Create: control-plane/tests/fixtures/smoke/targets.yaml

**Interfaces:**
- Produces: compileOpenClaw(bundle, contracts, inventory, targetBaseline) -> a non-authoritative conformance suggestion plus a separate `policy_enforcement` result. Mandatory semantics unsupported by OpenClaw make policy enforcement `unsupported`; the suggestion is never evidence of Policy IR preservation.
- Produces: compileDocker(bundle, contracts, inventory, targetBaseline) -> an analysis result. Because MVP Workflow Contracts make purpose, data-flow/redaction, and request argument binding mandatory and Docker cannot preserve them, the result is always `status: unsupported`, `proposal: null`, `changes: []`, and `actionable_diff: false`; writes, approvals, and server-level overexposure add further unsupported reasons.
- Docker has no successful proposal status in this MVP. Exposure analysis is evidence only and MUST NOT be presented as an applicable configuration proposal.

- [ ] **Step 1: Write failing adapter tests**

Create control-plane/tests/adapters.test.mjs:

~~~js
import assert from "node:assert/strict";
import test from "node:test";
import { sha256 } from "../src/canonical-json.mjs";
import { compileOpenClaw } from "../src/adapters/openclaw.mjs";
import { compileDocker } from "../src/adapters/docker.mjs";

const bundle = {
  included: [{
    capability_id: "customer.case.read",
    implementation: "com.example/support@1.0.0#get_case"
  }],
  excluded: [{ capability_id: "communication.email.send", reason: "prohibited" }]
};
const contracts = [{
  metadata: { id: "customer.case.read" },
  implementation: { serverRef: "com.example/support@1.0.0", toolName: "get_case" },
  semantics: { actionClass: "read" },
  policy: { approval: { required: false } }
}];
const inventory = {
  servers: [{ server_ref: "com.example/support@1.0.0", tools: ["get_case"] }]
};
const openclawConfig = { mcp: { servers: { allow: [], deny: [] } } };
const dockerConfig = { profile: { name: "existing", servers: [] } };
const openclawTarget = { version: "fixture-1.0.0", currentConfig: openclawConfig, currentConfigDigest: sha256(openclawConfig) };
const dockerTarget = { version: "fixture-1.0.0", currentConfig: dockerConfig, currentConfigDigest: sha256(dockerConfig) };

test("OpenClaw separates non-authoritative conformance from unsupported policy enforcement", () => {
  const result = compileOpenClaw(bundle, contracts, inventory, openclawTarget);
  assert.equal(result.conformance_suggestion.status, "advisory_non_authoritative");
  assert.deepEqual(result.conformance_suggestion.suggested_config.mcp.servers.allow, ["com.example/support@1.0.0"]);
  assert.equal(result.policy_enforcement.status, "unsupported");
  assert.equal(result.policy_enforcement.preserves_ir, false);
  assert.equal(result.policy_enforcement.proposal, null);
  assert.deepEqual(result.policy_enforcement.changes, []);
  assert.deepEqual(result.policy_enforcement.unsupported_semantics, ["argument_binding", "data_flow_and_redaction", "per_request_approval", "purpose"]);
  assert.equal(result.target_version, "fixture-1.0.0");
});

test("Docker returns unsupported and no proposal for mandatory purpose and data-flow semantics", () => {
  const result = compileDocker(bundle, contracts, inventory, dockerTarget);
  assert.equal(result.status, "unsupported");
  assert.equal(result.enforcement_strength, "unsupported");
  assert.equal(result.proposal, null);
  assert.deepEqual(result.changes, []);
  assert.equal(result.actionable_diff, false);
  assert.deepEqual(result.unsupported_semantics, ["argument_binding", "data_flow_and_redaction", "purpose"]);
});

test("Docker reports writes and approvals as additional unsupported semantics", () => {
  const write = structuredClone(contracts);
  write[0].semantics.actionClass = "external_write";
  write[0].policy.approval.required = true;
  const result = compileDocker(bundle, write, inventory, dockerTarget);
  assert.equal(result.proposal, null);
  assert.ok(result.unsupported_semantics.includes("write_action"));
  assert.ok(result.unsupported_semantics.includes("per_request_approval"));
});

test("Docker reports server-level overexposure and still emits no actionable diff", () => {
  const overexposed = structuredClone(inventory);
  overexposed.servers[0].tools.push("delete_case");
  const result = compileDocker(bundle, contracts, overexposed, dockerTarget);
  assert.ok(result.unsupported_semantics.includes("server_tool_overexposure"));
  assert.equal(result.proposal, null);
  assert.deepEqual(result.changes, []);
});
~~~

Create control-plane/tests/fixtures/smoke/targets.yaml:

~~~yaml
schema_version: target-baselines/v1alpha1
openclaw:
  version: fixture-1.0.0
  currentConfigDigest: "sha256:9dcecdd6d6449b62d5d04fdce6d3551c06ed68740afbc0846b274fd14414fc5e"
  currentConfig:
    mcp:
      servers:
        allow: []
        deny: []
docker:
  version: fixture-1.0.0
  currentConfigDigest: "sha256:b1d90046e06dc7d592056b0c1d3e61201b2de5611e7f0c0523c52b545d204e11"
  currentConfig:
    profile:
      name: existing
      servers: []
~~~

- [ ] **Step 2: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/adapters.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement the OpenClaw compiler**

Create control-plane/src/adapters/openclaw.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";

function selectExact(bundle, contracts) {
  return bundle.included.map((item) => {
    const contract = contracts.find((candidate) => candidate.metadata.id === item.capability_id
      && candidate.implementation.serverRef + "#" + candidate.implementation.toolName === item.implementation);
    if (!contract) throw new Error("bundle implementation has no exact contract: " + item.implementation);
    return contract;
  });
}

export function compileOpenClaw(bundle, contracts, inventory, targetBaseline) {
  if (sha256(targetBaseline.currentConfig) !== targetBaseline.currentConfigDigest) throw new Error("OpenClaw target baseline digest mismatch");
  const selected = selectExact(bundle, contracts);
  const servers = [...new Set(selected.map((contract) => contract.implementation.serverRef))].sort();
  const selectedTools = new Set(selected.map((contract) => contract.implementation.serverRef + "#" + contract.implementation.toolName));
  const exposureGaps = inventory.servers
    .filter((server) => servers.includes(server.server_ref))
    .flatMap((server) => server.tools.filter((tool) => !selectedTools.has(server.server_ref + "#" + tool)).map((tool) => server.server_ref + "#" + tool));
  const proposedConfig = { mcp: { servers: { allow: servers, deny: [] } } };
  return {
    target: "openclaw",
    target_version: targetBaseline.version,
    current_config_digest: targetBaseline.currentConfigDigest,
    conformance_suggestion: {
      status: "advisory_non_authoritative",
      warning: "This checks configuration conformance only; it neither authorizes calls nor preserves request-time Policy IR semantics.",
      suggested_config: proposedConfig,
      changes: [{ op: "replace", path: "/mcp/servers/allow", before: targetBaseline.currentConfig.mcp.servers.allow, after: servers }],
      exposure_gaps: exposureGaps.sort()
    },
    policy_enforcement: {
      status: "unsupported",
      enforcement_strength: "unsupported",
      preserves_ir: false,
      proposal: null,
      changes: [],
      actionable_diff: false,
      unsupported_semantics: ["argument_binding", "data_flow_and_redaction", "per_request_approval", "purpose"]
    }
  };
}
~~~

- [ ] **Step 4: Implement the Docker compiler**

Create control-plane/src/adapters/docker.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";

export function compileDocker(bundle, contracts, inventory, targetBaseline) {
  if (sha256(targetBaseline.currentConfig) !== targetBaseline.currentConfigDigest) throw new Error("Docker target baseline digest mismatch");
  const selected = bundle.included.map((item) => {
    const contract = contracts.find((candidate) => candidate.metadata.id === item.capability_id
      && candidate.implementation.serverRef + "#" + candidate.implementation.toolName === item.implementation);
    if (!contract) throw new Error("bundle implementation has no exact contract: " + item.implementation);
    return contract;
  });
  const unsupported = ["purpose", "data_flow_and_redaction", "argument_binding"];
  if (selected.some((contract) => contract.semantics.actionClass !== "read")) unsupported.push("write_action");
  if (selected.some((contract) => contract.policy.approval.required) || bundle.included.some((item) => item.approval_required)) unsupported.push("per_request_approval");
  const selectedTools = new Set(selected.map((contract) => contract.implementation.serverRef + "#" + contract.implementation.toolName));
  const servers = [...new Set(selected.map((contract) => contract.implementation.serverRef))].sort();
  const overexposed = inventory.servers
    .filter((server) => servers.includes(server.server_ref))
    .flatMap((server) => server.tools.filter((tool) => !selectedTools.has(server.server_ref + "#" + tool)));
  if (overexposed.length) unsupported.push("server_tool_overexposure");
  return {
    target: "docker-mcp-gateway",
    target_version: targetBaseline.version,
    status: "unsupported",
    enforcement_strength: "unsupported",
    current_config_digest: targetBaseline.currentConfigDigest,
    exposure_analysis: { exact_servers: servers, overexposed_tools: overexposed.sort() },
    proposal: null,
    changes: [],
    actionable_diff: false,
    unsupported_semantics: [...new Set(unsupported)].sort()
  };
}
~~~

- [ ] **Step 5: Run tests and commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
cd "$REPOSITORY_ROOT"
git add control-plane/src/adapters control-plane/tests/adapters.test.mjs control-plane/tests/fixtures/smoke/targets.yaml
git commit -m "feat: compile advisory gateway artifacts"
~~~

Expected: OpenClaw conformance suggestions remain explicitly non-authoritative while the separate policy-enforcement result is unsupported and never claims IR preservation; every Docker result is unsupported with no proposal or actionable diff, and writes, approvals, or server overexposure add reasons; all adapter and prior tests pass.

---

### Task 9: Detect protected contract drift and quarantine implementations

**Files:**
- Create: control-plane/src/drift/analyze.mjs
- Create: control-plane/tests/drift.test.mjs

**Interfaces:**
- Consumes: a contract from the independently loaded prior-approved store plus protectedProjection(server, tool) from the current observation boundary.
- Produces: analyzeDrift(contract, server, tool, sourceEvidenceRef) -> { status, changed_fields, approved_digest, observed_digest }.
- Any protected change returns `quarantined`; candidate or same-run contracts are rejected as invalid baselines.
- Protected fields include server identity, protocol, endpoint class and origin, server owner and execution metadata, tool name and title, description, input/output schemas, annotations, tool execution metadata, auth, and source provenance.

- [ ] **Step 1: Write failing drift tests**

Create control-plane/tests/drift.test.mjs:

~~~js
import assert from "node:assert/strict";
import test from "node:test";
import { protectedProjection } from "../src/contracts/implementation-projection.mjs";
import { analyzeDrift } from "../src/drift/analyze.mjs";
import { sha256 } from "../src/canonical-json.mjs";

const server = {
  server_ref: "com.example/support@1.0.0",
  protocol_version: "2025-11-25",
  endpoint_class: "corp-primary",
  endpoint_origin: "https://mcp.example.test/support",
  owner: "support-platform",
  execution: { transport: "streamable-http", runtime: "managed" }
};
const tool = {
  name: "get_case",
  title: "Get case",
  description: "Read one case.",
  inputSchema: { type: "object" },
  outputSchema: { type: "object" },
  annotations: { readOnlyHint: true },
  execution: { timeout_ms: 3000, side_effect_class: "none" },
  auth: { audience: "https://mcp.example.com", scopes: ["case.read"], tenant_binding: "required", credential_passthrough: "forbidden" }
};
const contract = {
  metadata: { status: "approved" },
  assurance: { approvedImplementationDigest: sha256(protectedProjection(server, tool, "EVD-001")) }
};

test("unchanged protected projection remains current", () => {
  assert.equal(analyzeDrift(contract, server, tool, "EVD-001").status, "current");
});

test("description drift quarantines the implementation", () => {
  const changed = structuredClone(tool);
  changed.description = "Read a case and delete it after retrieval.";
  const result = analyzeDrift(contract, server, changed, "EVD-001");
  assert.equal(result.status, "quarantined");
  assert.ok(result.changed_fields.includes("description"));
});

test("source provenance drift quarantines the implementation", () => {
  const result = analyzeDrift(contract, server, tool, "EVD-002");
  assert.equal(result.status, "quarantined");
  assert.ok(result.changed_fields.includes("source_provenance_ref"));
});

test("endpoint, title, and execution metadata are protected", () => {
  const changedServer = structuredClone(server);
  const changedTool = structuredClone(tool);
  changedServer.endpoint_origin = "https://mcp.example.test/other";
  changedTool.title = "Delete case";
  changedTool.execution.timeout_ms = 60000;
  const result = analyzeDrift(contract, changedServer, changedTool, "EVD-001");
  assert.equal(result.status, "quarantined");
  assert.ok(result.changed_fields.includes("endpoint_origin"));
  assert.ok(result.changed_fields.includes("title"));
  assert.ok(result.changed_fields.includes("tool_execution"));
});

test("current candidates cannot establish their own drift baseline", () => {
  assert.throws(() => analyzeDrift({ ...contract, metadata: { status: "candidate" } }, server, tool, "EVD-001"), /prior-approved baseline/);
});
~~~

- [ ] **Step 2: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/drift.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement protected drift comparison**

Create control-plane/src/drift/analyze.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { protectedProjection } from "../contracts/implementation-projection.mjs";

function fieldDigest(value) {
  return value === undefined ? "missing" : sha256(value);
}

function changedFields(left, right) {
  const keys = new Set([...Object.keys(left), ...Object.keys(right)]);
  return [...keys].filter((key) => fieldDigest(left[key]) !== fieldDigest(right[key]));
}

export function analyzeDrift(contract, server, tool, sourceEvidenceRef) {
  if (contract?.metadata?.status !== "approved" || !contract?.assurance?.approvedImplementation) {
    throw new Error("prior-approved baseline is required for drift analysis");
  }
  const observed = protectedProjection(server, tool, sourceEvidenceRef);
  const observedDigest = sha256(observed);
  const approvedDigest = contract.assurance.approvedImplementationDigest;
  if (approvedDigest === observedDigest) {
    return { status: "current", changed_fields: [], approved_digest: approvedDigest, observed_digest: observedDigest };
  }
  const approved = contract.assurance.approvedImplementation || {};
  return {
    status: "quarantined",
    changed_fields: changedFields(approved, observed),
    approved_digest: approvedDigest,
    observed_digest: observedDigest
  };
}
~~~

- [ ] **Step 4: Correct the test fixture to preserve the approved projection**

Replace the contract declaration in control-plane/tests/drift.test.mjs with:

~~~js
const approvedImplementation = protectedProjection(server, tool, "EVD-001");
const contract = {
  metadata: { status: "approved" },
  assurance: {
    approvedImplementation,
    approvedImplementationDigest: sha256(approvedImplementation)
  }
};
~~~

- [ ] **Step 5: Run tests and commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
cd "$REPOSITORY_ROOT"
git add control-plane/src/drift/analyze.mjs control-plane/tests/drift.test.mjs
git commit -m "feat: quarantine capability contract drift"
~~~

Expected: description drift is detected and all tests pass.

---

### Task 10: Simulate fail-closed decisions, composition risk, and fallback

**Files:**
- Create: control-plane/src/eval/simulate.mjs
- Create: control-plane/src/eval/replay-ledger.mjs
- Create: control-plane/tests/simulate.test.mjs

**Interfaces:**
- Produces: simulateDecision(request, policy, bundle, approvedContracts, authority) -> allow, deny, require_approval, require_revalidation, or indeterminate with reasons. `authority` contains the evidence ledger, current-source eligibility, equivalence proofs, and tenant-scoped ReplayLedger.
- Produces: evaluateComposition(contracts) -> findings.
- Produces: canFallback(primary, candidate, request, proof, assurance) -> { allowed, reasons }; a shared string label is never a proof.
- Produces: canRetry(contract, request, retryContext) -> { allowed, reasons }; only classified transient failures may retry, and writes additionally require verified idempotency, a propagated idempotency key, adapter support, unchanged binding, and remaining attempts.
- Approval records validate against approval-record/v1alpha1, bind contract and reviewed implementation digests plus an optional exact equivalence-proof digest, carry an approval digest and nonce, and are consumed once per tenant only after a simulated allow.

- [ ] **Step 1: Write failing security simulation tests**

Create control-plane/tests/simulate.test.mjs:

~~~js
import assert from "node:assert/strict";
import test from "node:test";
import { sha256 } from "../src/canonical-json.mjs";
import { deriveBundleItem } from "../src/compile/bundle.mjs";
import { summarizeEvidence } from "../src/evidence/freshness.mjs";
import { ReplayLedger } from "../src/eval/replay-ledger.mjs";
import { canFallback, canRetry, equivalenceDimensions, evaluateComposition, simulateDecision } from "../src/eval/simulate.mjs";

const policy = {
  metadata: { expiresAt: "2026-10-16T00:00:00Z" },
  match: { subjects: ["support-agent"], agents: ["assistant"], tenants: ["tenant-a"], environments: ["production"], workflows: ["support-case-resolution/inspect-case"] },
  capabilities: { include: ["customer.case.read"], exclude: ["communication.email.send"] },
  constraints: { approval: "none", purpose: "resolve an authenticated customer case", region: "eu", dataClasses: ["customer-pii"] }
};
const writePolicy = {
  metadata: { expiresAt: "2026-10-16T00:00:00Z" },
  match: { subjects: ["support-agent"], agents: ["assistant"], tenants: ["tenant-a"], environments: ["production"], workflows: ["support-case-resolution/inspect-case"] },
  capabilities: { include: ["communication.email.send"], exclude: [] },
  constraints: { approval: "required", purpose: "resolve an authenticated customer case", region: "eu", dataClasses: [] }
};
const readContract = {
  metadata: { id: "customer.case.read", version: "1.0.0", status: "approved", expiresAt: "2026-10-16T00:00:00Z" },
  implementation: { serverRef: "support", toolName: "get_case", inputSchemaHash: "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa", outputSchemaHash: "sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb", verifiedProperties: { region: "eu" } },
  semantics: { actionClass: "read", destructive: false, reversible: true, readsPrivateData: true, seesUntrustedContent: true, canExternalizeData: false, dataClasses: ["customer-pii"], idempotency: "verified", successPredicate: "returns the requested tenant-bound case" },
  identity: { principalTypes: ["human_delegated"], delegationMode: "oauth_authorization_code", audience: "https://mcp.example.com", requiredScopes: ["case.read"], tenantBinding: "required", credentialPassthrough: "forbidden" },
  policy: { approval: { required: false } },
  routing: { equivalenceGroup: null, equivalenceProofRef: null, fallback: "verified_equivalence_only", retry: { transientOnly: true, maxAttempts: 1 } },
  assurance: { approvedImplementationDigest: "sha256:cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc", evidenceRefs: ["EVD-read-implementation", "EVD-read-evaluation"], evalSuite: "support-read-v1" }
};
const sendContract = {
  metadata: { id: "communication.email.send", version: "1.0.0", status: "approved", expiresAt: "2026-10-16T00:00:00Z" },
  implementation: { serverRef: "mail", toolName: "send_email", inputSchemaHash: "sha256:dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd", outputSchemaHash: "sha256:eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee", verifiedProperties: { region: "eu" } },
  semantics: { actionClass: "external_write", destructive: false, reversible: false, readsPrivateData: false, seesUntrustedContent: true, canExternalizeData: true, dataClasses: [], idempotency: "none", successPredicate: "one message accepted for delivery" },
  identity: { principalTypes: ["human_delegated"], delegationMode: "oauth_authorization_code", audience: "https://mail.example.com", requiredScopes: ["mail.send"], tenantBinding: "required", credentialPassthrough: "forbidden" },
  policy: { approval: { required: true } },
  routing: { equivalenceGroup: null, equivalenceProofRef: null, fallback: "forbidden", retry: { transientOnly: true, maxAttempts: 0 } },
  assurance: { approvedImplementationDigest: "sha256:ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff", evidenceRefs: ["EVD-write-implementation", "EVD-write-evaluation"], evalSuite: "mail-send-v1" }
};
const bundleContext = { subject: "support-agent", agent: "assistant", tenant: "tenant-a", workflow: "support-case-resolution", stage: "inspect-case", purpose: "resolve an authenticated customer case", environment: "production", dataClasses: ["customer-pii"], region: "eu", now: "2026-07-16T00:00:00Z" };
const evidenceLedger = {
  schema_version: "evidence-ledger/v1alpha1",
  records: [readContract, sendContract].flatMap((contract) => contract.assurance.evidenceRefs.map((evidence_id, index) => ({
    evidence_id,
    kind: index === 0 ? "implementation" : "evaluation",
    subject_digest: index === 0
      ? contract.assurance.approvedImplementationDigest
      : sha256({ approved_implementation_digest: contract.assurance.approvedImplementationDigest, eval_suite: contract.assurance.evalSuite }),
    payload_digest: "sha256:" + (contract === readContract ? String(index + 1).repeat(64) : String(index + 3).repeat(64)),
    observed_at: "2026-07-15T00:00:00Z",
    expires_at: "2026-10-16T00:00:00Z",
    reviewed_by: "architecture-review-board",
    status: "current"
  })))
};
const readEligibility = { drift_status: "current", source_observed_at: "2026-07-15T00:00:00Z", source_expires_at: "2026-10-16T00:00:00Z", source_evidence_refs: ["sha256:" + "5".repeat(64)], source_provenance_refs: ["sha256:" + "6".repeat(64)], source_types: ["live_schema_export"] };
const writeEligibility = { drift_status: "current", source_observed_at: "2026-07-15T00:00:00Z", source_expires_at: "2026-10-16T00:00:00Z", source_evidence_refs: ["sha256:" + "7".repeat(64)], source_provenance_refs: ["sha256:" + "8".repeat(64)], source_types: ["live_schema_export"] };
const readItem = deriveBundleItem(policy, readContract, summarizeEvidence(readContract, evidenceLedger, bundleContext.now), readEligibility, null);
const writeItem = deriveBundleItem(writePolicy, sendContract, summarizeEvidence(sendContract, evidenceLedger, bundleContext.now), writeEligibility, null);
const bundle = { schema_version: "capability-bundle/v1alpha1", context: bundleContext, policy_digest: sha256(policy), expires_at: "2026-10-16T00:00:00.000Z", included: [readItem], excluded: [{ capability_id: "communication.email.send", reason: "prohibited" }] };
const writeBundle = { schema_version: "capability-bundle/v1alpha1", context: { ...bundleContext, dataClasses: [] }, policy_digest: sha256(writePolicy), expires_at: "2026-10-16T00:00:00.000Z", included: [writeItem], excluded: [] };
const baseRequest = {
  subject: "support-agent",
  agent: "assistant",
  tenant: "tenant-a",
  environment: "production",
  workflow: "support-case-resolution/inspect-case",
  purpose: "resolve an authenticated customer case",
  region: "eu",
  now: "2026-07-16T00:00:00Z",
  arguments: {}
};
function authority(replayLedger = new ReplayLedger()) {
  return {
    evidenceLedger,
    eligibilityByImplementation: {
      "support#get_case": readEligibility,
      "mail#send_email": writeEligibility
    },
    equivalenceProofs: [],
    replayLedger
  };
}

function approvalFor(argumentsValue, overrides = {}) {
  const payload = {
    schema_version: "approval-record/v1alpha1",
    approval_id: "approval-001",
    nonce: "nonce_abcdefghijklmnop",
    user: "support-agent",
    capability: "communication.email.send",
    argument_hash: sha256(argumentsValue),
    purpose: baseRequest.purpose,
    implementation: "mail#send_email",
    contract_digest: sha256(sendContract),
    approved_implementation_digest: sendContract.assurance.approvedImplementationDigest,
    equivalence_proof_digest: null,
    schema_digest: sha256(writeItem.schema_hashes),
    policy_digest: writeBundle.policy_digest,
    tenant: "tenant-a",
    issued_at: "2026-07-15T23:59:00Z",
    expires_at: "2026-07-16T00:05:00Z",
    ...overrides
  };
  return { ...payload, approval_digest: sha256(payload) };
}

test("eligible read is allowed", () => {
  const result = simulateDecision({ ...baseRequest, capability: "customer.case.read", approval: null }, policy, bundle, [readContract], authority());
  assert.equal(result.decision, "allow");
});

test("prohibited external write is denied", () => {
  const result = simulateDecision({ ...baseRequest, capability: "communication.email.send", approval: null }, policy, bundle, [sendContract], authority());
  assert.equal(result.decision, "deny");
});

test("missing tenant is indeterminate and therefore fail-closed", () => {
  const result = simulateDecision({ ...baseRequest, tenant: "", capability: "customer.case.read", approval: null }, policy, bundle, [readContract], authority());
  assert.equal(result.decision, "indeterminate");
});

test("write approval is bound to user, capability, arguments, implementation, tenant, and expiry", () => {
  const argumentsValue = { to: "customer@example.test", subject: "Case update" };
  const request = {
    ...baseRequest,
    capability: "communication.email.send",
    arguments: argumentsValue,
    approval: approvalFor(argumentsValue, { argument_hash: "sha256:" + "b".repeat(64) })
  };
  assert.deepEqual(simulateDecision(request, writePolicy, writeBundle, [sendContract], authority()), {
    decision: "deny",
    reasons: ["approval_binding_mismatch"]
  });
});

test("an exact unexpired write approval is allowed in offline simulation", () => {
  const argumentsValue = { to: "customer@example.test", subject: "Case update" };
  const request = {
    ...baseRequest,
    capability: "communication.email.send",
    arguments: argumentsValue,
    approval: approvalFor(argumentsValue)
  };
  assert.equal(simulateDecision(request, writePolicy, writeBundle, [sendContract], authority()).decision, "allow");
});

test("approval nonce and id are single-use within one tenant", () => {
  const argumentsValue = { to: "customer@example.test", subject: "Case update" };
  const request = { ...baseRequest, capability: "communication.email.send", arguments: argumentsValue, approval: approvalFor(argumentsValue) };
  const ledger = new ReplayLedger();
  assert.equal(simulateDecision(request, writePolicy, writeBundle, [sendContract], authority(ledger)).decision, "allow");
  assert.deepEqual(simulateDecision(request, writePolicy, writeBundle, [sendContract], authority(ledger)), { decision: "deny", reasons: ["approval_replay"] });
});

test("flipping a derived approval flag cannot bypass approval", () => {
  const tampered = structuredClone(writeBundle);
  tampered.included[0].approval_required = false;
  const result = simulateDecision({ ...baseRequest, capability: "communication.email.send", approval: null }, writePolicy, tampered, [sendContract], authority());
  assert.deepEqual(result, { decision: "require_revalidation", reasons: ["bundle_contract_binding_mismatch"] });
});

test("private read plus external send creates composition risk", () => {
  const findings = evaluateComposition([readContract, sendContract]);
  assert.equal(findings[0].type, "private_read_to_external_send");
});

test("write fallback remains forbidden", () => {
  assert.equal(canFallback(sendContract, structuredClone(sendContract), {}, null, {}).allowed, false);
});

test("fallback requires a current versioned proof over all protected dimensions", () => {
  const primary = structuredClone(readContract);
  const candidate = structuredClone(readContract);
  primary.routing.equivalenceGroup = "case-read-v1";
  primary.routing.equivalenceProofRef = "EQP-case-read-v1";
  candidate.routing.equivalenceGroup = "case-read-v1";
  candidate.routing.equivalenceProofRef = "EQP-case-read-v1";
  candidate.implementation.serverRef = "support-backup";
  const proof = {
    proof_id: "EQP-case-read-v1",
    version: "1.0.0",
    capability_id: "customer.case.read",
    equivalence_group: "case-read-v1",
    primary_contract_digest: sha256(primary),
    candidate_contract_digest: sha256(candidate),
    dimension_digests: equivalenceDimensions(primary),
    eval_suite: "support-read-equivalence-v1",
    evidence_refs: ["EVD-read-evaluation"],
    reviewed_by: "architecture-review-board",
    reviewed_at: "2026-07-15T00:00:00Z",
    expires_at: "2026-10-16T00:00:00Z",
    status: "current"
  };
  const assurance = { primary: { current: true, evaluation_passed: true }, candidate: { current: true, evaluation_passed: true } };
  assert.equal(canFallback(primary, candidate, { now: baseRequest.now }, proof, assurance).allowed, true);
  candidate.semantics.successPredicate = "returns any case";
  assert.equal(canFallback(primary, candidate, { now: baseRequest.now }, proof, assurance).allowed, false);
});

test("retry is transient-only and write retry requires verified idempotency end to end", () => {
  assert.equal(canRetry(readContract, baseRequest, { failureClass: "transient_transport", attempt: 0 }).allowed, true);
  assert.equal(canRetry(readContract, baseRequest, { failureClass: "business", attempt: 0 }).allowed, false);
  assert.equal(canRetry(sendContract, baseRequest, { failureClass: "transient_transport", attempt: 0 }).allowed, false);
  const idempotentWrite = structuredClone(sendContract);
  idempotentWrite.semantics.idempotency = "verified";
  idempotentWrite.routing.retry.maxAttempts = 2;
  const binding = sha256({ arguments: baseRequest.arguments, tenant: baseRequest.tenant, audience: idempotentWrite.identity.audience, implementation: "mail#send_email" });
  assert.equal(canRetry(idempotentWrite, baseRequest, {
    failureClass: "transient_transport",
    attempt: 1,
    idempotencyKey: "idem_case_123456789",
    keyPropagationVerified: true,
    adapterSupportsIdempotency: true,
    originalBinding: binding,
    currentBinding: binding
  }).allowed, true);
});
~~~

- [ ] **Step 2: Run and verify module-not-found**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
node --test tests/simulate.test.mjs
~~~

Expected: FAIL with ERR_MODULE_NOT_FOUND.

- [ ] **Step 3: Implement fail-closed simulation**

Create control-plane/src/eval/replay-ledger.mjs:

~~~js
export class ReplayLedger {
  #used = new Map();

  isUsed(tenant, approval) {
    const tenantSet = this.#used.get(tenant);
    return Boolean(tenantSet?.has("id:" + approval.approval_id) || tenantSet?.has("nonce:" + approval.nonce));
  }

  consume(tenant, approval) {
    if (this.isUsed(tenant, approval)) throw new Error("approval replay");
    const tenantSet = this.#used.get(tenant) || new Set();
    tenantSet.add("id:" + approval.approval_id);
    tenantSet.add("nonce:" + approval.nonce);
    this.#used.set(tenant, tenantSet);
  }
}
~~~

Create control-plane/src/eval/simulate.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { verifyBundleItem } from "../compile/bundle.mjs";
import { assertValid } from "../validate.mjs";

function result(decision, reasons) {
  return { decision, reasons };
}

function validInstant(value) {
  const milliseconds = Date.parse(value);
  if (!Number.isFinite(milliseconds)) return false;
  const canonical = new Date(milliseconds).toISOString();
  return value === canonical || value === canonical.replace(".000Z", "Z");
}

function approvalPayload(approval) {
  const { approval_digest, ...payload } = approval;
  return payload;
}

function approvalMatches(request, included, bundle) {
  const approval = request.approval;
  if (!approval || request.arguments === undefined || !validInstant(request.now) || !validInstant(approval.issued_at) || !validInstant(approval.expires_at)) return false;
  try {
    assertValid("approval-record", approval);
  } catch {
    return false;
  }
  const expiresAt = Date.parse(approval.expires_at);
  const issuedAt = Date.parse(approval.issued_at);
  const now = Date.parse(request.now);
  return approval.approval_digest === sha256(approvalPayload(approval))
    && approval.user === request.subject
    && approval.capability === request.capability
    && approval.argument_hash === sha256(request.arguments)
    && approval.purpose === request.purpose
    && approval.implementation === included.implementation
    && approval.contract_digest === included.contract_digest
    && approval.approved_implementation_digest === included.approved_implementation_digest
    && approval.equivalence_proof_digest === included.equivalence_proof_digest
    && approval.schema_digest === sha256(included.schema_hashes)
    && approval.policy_digest === bundle.policy_digest
    && approval.tenant === request.tenant
    && Number.isFinite(expiresAt)
    && Number.isFinite(issuedAt)
    && Number.isFinite(now)
    && issuedAt <= now
    && expiresAt > now;
}

function bundleEnvelopeValid(bundle, policy, request) {
  try {
    assertValid("capability-bundle", bundle);
  } catch {
    return false;
  }
  const includedIds = bundle.included.map((item) => item.capability_id);
  const excludedIds = bundle.excluded.map((item) => item.capability_id);
  if (new Set(includedIds).size !== includedIds.length || new Set(excludedIds).size !== excludedIds.length || includedIds.some((id) => excludedIds.includes(id))) return false;
  const expectedExpiry = new Date(Math.min(Date.parse(policy.metadata.expiresAt), ...bundle.included.map((item) => Date.parse(item.expires_at)))).toISOString();
  return bundle.policy_digest === sha256(policy)
    && bundle.expires_at === expectedExpiry
    && validInstant(bundle.context.now)
    && Date.parse(bundle.context.now) <= Date.parse(request.now)
    && sha256([...bundle.context.dataClasses].sort()) === sha256([...(policy.constraints.dataClasses || [])].sort());
}

export function simulateDecision(request, policy, bundle, contracts, authority) {
  if (!request.subject || !request.agent || !request.tenant || !request.environment || !request.workflow || !request.purpose || !request.region || !validInstant(request.now) || !request.capability) {
    return result("indeterminate", ["missing_identity_or_context"]);
  }
  if (!policy.match.subjects.includes(request.subject) || !policy.match.agents.includes(request.agent) || !policy.match.tenants.includes(request.tenant) || !policy.match.environments.includes(request.environment) || !policy.match.workflows.includes(request.workflow)) {
    return result("deny", ["policy_context_mismatch"]);
  }
  if (!bundleEnvelopeValid(bundle, policy, request)) return result("require_revalidation", ["bundle_envelope_mismatch"]);
  if (Date.parse(bundle.expires_at) <= Date.parse(request.now) || Date.parse(policy.metadata.expiresAt) <= Date.parse(request.now)) return result("require_revalidation", ["expired_policy_or_bundle"]);
  const expectedWorkflow = bundle.context.workflow + "/" + bundle.context.stage;
  if (bundle.context.subject !== request.subject
    || bundle.context.agent !== request.agent
    || bundle.context.tenant !== request.tenant
    || bundle.context.environment !== request.environment
    || expectedWorkflow !== request.workflow
    || bundle.context.purpose !== request.purpose
    || bundle.context.region !== request.region) {
    return result("deny", ["bundle_context_mismatch"]);
  }
  if (policy.capabilities.exclude.includes(request.capability)) return result("deny", ["capability_prohibited"]);
  if (!policy.capabilities.include.includes(request.capability)) return result("deny", ["capability_not_allowed"]);
  const included = bundle.included.find((item) => item.capability_id === request.capability);
  if (!included) return result("deny", ["capability_not_in_bundle"]);
  const contract = contracts.find((item) => item.metadata.id === request.capability
    && item.implementation.serverRef + "#" + item.implementation.toolName === included.implementation);
  if (!contract) return result("indeterminate", ["contract_missing"]);
  if (!authority?.evidenceLedger || !authority?.eligibilityByImplementation || !authority?.replayLedger) return result("indeterminate", ["authority_inputs_missing"]);
  const eligibility = authority.eligibilityByImplementation[included.implementation];
  const proof = included.equivalence_proof_digest
    ? authority.equivalenceProofs.find((item) => sha256(item) === included.equivalence_proof_digest)
    : null;
  const binding = verifyBundleItem(included, policy, contract, authority.evidenceLedger, eligibility, proof, request.now);
  if (!binding.valid) return result("require_revalidation", [binding.reason]);
  if (Date.parse(contract.metadata.expiresAt) <= Date.parse(request.now) || Date.parse(included.expires_at) <= Date.parse(request.now)) return result("require_revalidation", ["contract_expired"]);
  if (contract.metadata.status !== "approved") return result("require_revalidation", ["contract_not_approved"]);
  if (contract.identity.credentialPassthrough !== "forbidden" || contract.identity.tenantBinding !== "required") {
    return result("indeterminate", ["identity_contract_unknown"]);
  }
  if (included.approval_required && !request.approval) return result("require_approval", ["approval_required"]);
  if (included.approval_required && authority.replayLedger.isUsed(request.tenant, request.approval)) return result("deny", ["approval_replay"]);
  if (included.approval_required && !approvalMatches(request, included, bundle)) return result("deny", ["approval_binding_mismatch"]);
  if (included.approval_required) authority.replayLedger.consume(request.tenant, request.approval);
  return result("allow", []);
}

export function evaluateComposition(contracts) {
  const privateReads = contracts.filter((item) => item.semantics.readsPrivateData);
  const externalWrites = contracts.filter((item) => item.semantics.canExternalizeData);
  const findings = [];
  for (const source of privateReads) {
    for (const sink of externalWrites) {
      findings.push({
        type: "private_read_to_external_send",
        source: source.metadata.id,
        sink: sink.metadata.id,
        severity: "critical"
      });
    }
  }
  return findings;
}

export function equivalenceDimensions(contract) {
  return {
    identity: sha256({ principalTypes: [...contract.identity.principalTypes].sort(), delegationMode: contract.identity.delegationMode, audience: contract.identity.audience, scopes: [...contract.identity.requiredScopes].sort(), tenantBinding: contract.identity.tenantBinding, credentialPassthrough: contract.identity.credentialPassthrough }),
    data_boundary: sha256({ dataClasses: [...contract.semantics.dataClasses].sort(), region: contract.implementation.verifiedProperties.region, readsPrivateData: contract.semantics.readsPrivateData }),
    side_effects: sha256({ actionClass: contract.semantics.actionClass, destructive: contract.semantics.destructive, canExternalizeData: contract.semantics.canExternalizeData }),
    schemas: sha256([contract.implementation.inputSchemaHash, contract.implementation.outputSchemaHash]),
    approval: sha256(contract.policy.approval),
    reversibility: sha256(contract.semantics.reversible),
    idempotency: sha256(contract.semantics.idempotency),
    success_semantics: sha256(contract.semantics.successPredicate)
  };
}

export function canFallback(primary, candidate, request, proof, assurance) {
  const reasons = [];
  if (primary.routing.fallback === "forbidden") reasons.push("primary_forbids_fallback");
  if (primary.metadata.id !== candidate.metadata.id) reasons.push("capability_changed");
  if (!proof
    || proof.proof_id !== primary.routing.equivalenceProofRef
    || proof.proof_id !== candidate.routing.equivalenceProofRef
    || proof.version.split(".").length !== 3
    || proof.status !== "current"
    || Date.parse(proof.expires_at) <= Date.parse(request.now)
    || proof.capability_id !== primary.metadata.id
    || !primary.routing.equivalenceGroup
    || proof.equivalence_group !== primary.routing.equivalenceGroup
    || proof.equivalence_group !== candidate.routing.equivalenceGroup
    || proof.primary_contract_digest !== sha256(primary)
    || proof.candidate_contract_digest !== sha256(candidate)
    || sha256(proof.dimension_digests) !== sha256(equivalenceDimensions(primary))
    || sha256(proof.dimension_digests) !== sha256(equivalenceDimensions(candidate))) reasons.push("equivalence_not_proven");
  if (primary.semantics.actionClass !== "read" || candidate.semantics.actionClass !== "read") reasons.push("write_or_destructive_action");
  if (!assurance.primary?.current || !assurance.primary?.evaluation_passed || !assurance.candidate?.current || !assurance.candidate?.evaluation_passed) reasons.push("assurance_not_current");
  if (candidate.metadata.status !== "approved" || Date.parse(candidate.metadata.expiresAt) <= Date.parse(request.now)) reasons.push("candidate_not_current");
  if (request.approval && request.approval.equivalence_proof_digest !== sha256(proof)) reasons.push("approval_object_changed");
  return { allowed: reasons.length === 0, reasons };
}

export function canRetry(contract, request, retryContext) {
  const reasons = [];
  if (!contract.routing.retry.transientOnly || !["transient_transport", "rate_limit"].includes(retryContext.failureClass)) reasons.push("failure_not_transient");
  if (!Number.isInteger(retryContext.attempt) || retryContext.attempt >= contract.routing.retry.maxAttempts) reasons.push("retry_limit_reached");
  if (retryContext.crossImplementation) reasons.push("cross_implementation_is_fallback");
  if (retryContext.originalBinding && retryContext.originalBinding !== retryContext.currentBinding) reasons.push("request_binding_changed");
  if (contract.semantics.actionClass !== "read") {
    if (contract.semantics.idempotency !== "verified") reasons.push("idempotency_not_verified");
    if (!retryContext.idempotencyKey || !retryContext.keyPropagationVerified) reasons.push("idempotency_key_not_verified");
    if (!retryContext.adapterSupportsIdempotency) reasons.push("adapter_idempotency_not_verified");
    if (!retryContext.originalBinding || retryContext.originalBinding !== retryContext.currentBinding) reasons.push("write_binding_changed");
  }
  if (!request.tenant || !request.subject) reasons.push("identity_or_tenant_missing");
  return { allowed: reasons.length === 0, reasons };
}
~~~

- [ ] **Step 4: Run tests and commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
cd "$REPOSITORY_ROOT"
git add control-plane/src/eval/simulate.mjs control-plane/src/eval/replay-ledger.mjs control-plane/tests/simulate.test.mjs
git commit -m "test: simulate capability policy safety"
~~~

Expected: all decision and safety tests pass; every bundle field is re-derived, approval replay is denied, fallback lacks no proof dimension, retry honors idempotency, and no destructive false permit exists.

---

### Task 11: Assemble the assessment report and local CLI

**Files:**
- Create: control-plane/src/explain/explain.mjs
- Create: control-plane/src/eval/pilot-gate.mjs
- Create: control-plane/src/drift/aggregate-observations.mjs
- Create: control-plane/src/evidence/verify-artifacts.mjs
- Create: control-plane/src/assessment.mjs
- Create: control-plane/src/cli.mjs
- Create: control-plane/tests/assessment.test.mjs
- Create: control-plane/tests/pilot-gate.test.mjs
- Create: control-plane/tests/observation-aggregation.test.mjs
- Create: control-plane/tests/verify-artifacts.test.mjs
- Create: control-plane/tests/build-smoke-fixture.mjs
- Create: control-plane/tests/fixtures/smoke/assessment-inputs.yaml
- Create: control-plane/tests/fixtures/smoke/context.yaml
- Create: control-plane/tests/fixtures/first-run/assessment-inputs.yaml

**Interfaces:**
- Produces: assess(inputDirectory, outputDirectory) -> AssessmentManifest from an explicit assessment-inputs/v1alpha1 manifest.
- Produces: evaluatePilotGate({ mode, sources, servers, capabilities, traceEvaluation, criticalFalsePermits, approvedContracts, bundle, driftResults, targetArtifacts }) -> digest-bound pilot-scorecard/v1alpha1.
- CLI: node src/cli.mjs assess INPUT_DIRECTORY OUTPUT_DIRECTORY.
- Writes only the chosen local output directory; never changes source configs.
- Loads all source snapshots before the prior baseline, requires baseline review time to predate the earliest current observation, creates current candidates separately, aggregates repeated observations of one implementation with explicit `live_schema_export > private_registry > official_registry > gateway_config > host_config` precedence, retains every source digest/provenance reference, and quarantines conflicting protected projections before bundle compilation.
- Supports multiple sanitized current sources and optional sanitized trace sets. `mode: smoke` always emits `pilot_scorecard.status: not_evaluated`; `mode: pilot` digest-binds the approved contracts, eligible bundle, drift results, and target artifacts, and passes only with a non-empty prior-approved set, at least one current eligible read, authoritative host/OpenClaw and gateway/Docker artifacts, no critical-contract quarantine, official/private/live evidence, 3–5 servers, 10–30 capabilities, independently identified passing suites, and zero derived critical false permits. Approval, retry, and fallback results must bind the same approved high/critical `internal_write`, `external_write`, or `destructive` contract digest and action class.

- [ ] **Step 1: Add the exact assessment context fixture**

Create control-plane/tests/fixtures/smoke/context.yaml:

~~~yaml
schema_version: assessment-context/v1alpha1
subject: support-agent
agent: customer-support-assistant
tenant: tenant-a
workflow: support-case-resolution
stage: inspect-case
purpose: resolve an authenticated customer case
environment: production
dataClasses: [customer-pii]
region: eu
now: "2026-07-17T00:00:00Z"
~~~

Create control-plane/tests/fixtures/smoke/assessment-inputs.yaml and copy the same manifest to control-plane/tests/fixtures/first-run/assessment-inputs.yaml:

~~~yaml
schema_version: assessment-inputs/v1alpha1
mode: smoke
sourceSnapshots: [source-snapshot.yaml]
approvedBaseline: approved-baseline.yaml
evidenceLedger: evidence-ledger.yaml
equivalenceProofs: equivalence-proofs.yaml
mappings: mappings.yaml
workflow: workflow-contract.yaml
context: context.yaml
targets: targets.yaml
traces: null
architecturePack: null
~~~

The smoke fixture builder in Step 2 writes a separate synthetic prior-approved baseline reviewed on 2026-07-15 and a current observation dated 2026-07-16. The first-run fixture is identical except that `contracts` and `approvals` in approved-baseline.yaml are empty. No assessment code derives either baseline from the current snapshot.

- [ ] **Step 2: Write the failing end-to-end test**

Create control-plane/tests/assessment.test.mjs:

~~~js
import assert from "node:assert/strict";
import { mkdtemp, readFile } from "node:fs/promises";
import { tmpdir } from "node:os";
import { join } from "node:path";
import test, { before } from "node:test";
import YAML from "yaml";
import { assess } from "../src/assessment.mjs";
import { verifyArtifactDigests } from "../src/evidence/verify-artifacts.mjs";
import { buildSmokeFixtures } from "./build-smoke-fixture.mjs";

before(buildSmokeFixtures);

test("smoke assessment emits separate candidates and prior-approved contracts", async () => {
  const output = await mkdtemp(join(tmpdir(), "capability-assessment-"));
  const input = new URL("./fixtures/smoke/", import.meta.url);
  const manifest = await assess(input, output);
  assert.equal(manifest.schema_version, "assessment-manifest/v1alpha1");
  const persisted = YAML.parse(await readFile(join(output, "evidence-manifest.yaml"), "utf8"));
  assert.deepEqual(persisted, manifest);
  const inventory = YAML.parse(await readFile(join(output, "source-inventory.yaml"), "utf8"));
  assert.equal(inventory.schema_version, "source-inventory/v1alpha1");
  assert.equal(manifest.source_inventory, "source-inventory.yaml");
  assert.deepEqual(manifest.target_artifacts.sort(), ["target-diffs/docker.yaml", "target-diffs/openclaw.yaml"]);
  assert.ok(manifest.candidate_contracts.length > 0);
  assert.ok(manifest.approved_contracts.length > 0);
  const pilotGate = YAML.parse(await readFile(join(output, manifest.pilot_scorecard), "utf8"));
  assert.equal(pilotGate.status, "not_evaluated");
  assert.deepEqual(manifest.pilot_authority_digests, pilotGate.authority_digests);
  assert.match(manifest.pilot_authority_digests.approved_contracts, /^sha256:[a-f0-9]{64}$/);
  assert.match(manifest.pilot_authority_digests.eligible_bundle, /^sha256:[a-f0-9]{64}$/);
  assert.match(manifest.pilot_authority_digests.drift_results, /^sha256:[a-f0-9]{64}$/);
  assert.match(manifest.pilot_authority_digests.target_artifacts, /^sha256:[a-f0-9]{64}$/);
  const docker = YAML.parse(await readFile(join(output, "target-diffs/docker.yaml"), "utf8"));
  assert.equal(docker.status, "unsupported");
  assert.equal(docker.proposal, null);
  assert.deepEqual(docker.changes, []);
  const openclaw = YAML.parse(await readFile(join(output, "target-diffs/openclaw.yaml"), "utf8"));
  assert.equal(openclaw.conformance_suggestion.status, "advisory_non_authoritative");
  assert.equal(openclaw.policy_enforcement.status, "unsupported");
  assert.equal(openclaw.policy_enforcement.preserves_ir, false);
  const explanations = JSON.parse(await readFile(join(output, manifest.explanations), "utf8"));
  assert.ok(explanations.included[0].policy.rule_id);
  assert.ok(explanations.included[0].contract.digest);
  assert.ok(explanations.included[0].evidence.expires_at);
  await assert.doesNotReject(() => verifyArtifactDigests(output, manifest.artifact_digests));
});

test("first-run assessment emits candidates but no eligible capability, conformance suggestion, or applicable proposal", async () => {
  const output = await mkdtemp(join(tmpdir(), "capability-first-run-"));
  const manifest = await assess(new URL("./fixtures/first-run/", import.meta.url), output);
  const bundle = YAML.parse(await readFile(join(output, manifest.bundle), "utf8"));
  assert.equal(bundle.included.length, 0);
  assert.ok(manifest.candidate_contracts.length > 0);
  assert.deepEqual(manifest.approved_contracts, []);
  for (const path of manifest.target_artifacts) {
    const target = YAML.parse(await readFile(join(output, path), "utf8"));
    if (target.target === "openclaw") assert.equal(target.conformance_suggestion.suggested_config, null);
    const enforcement = target.policy_enforcement || target;
    assert.equal(enforcement.proposal, null);
    assert.deepEqual(enforcement.changes, []);
  }
});
~~~

Create control-plane/tests/build-smoke-fixture.mjs:

~~~js
import { cp, mkdir, readFile, writeFile } from "node:fs/promises";
import { dirname, join } from "node:path";
import { fileURLToPath } from "node:url";
import YAML from "yaml";
import { sha256 } from "../src/canonical-json.mjs";
import { normalizeCapability } from "../src/contracts/normalize-capability.mjs";

const fixtureRoot = dirname(fileURLToPath(new URL("./fixtures/smoke/assessment-inputs.yaml", import.meta.url)));
const firstRunRoot = dirname(fileURLToPath(new URL("./fixtures/first-run/assessment-inputs.yaml", import.meta.url)));

async function readYaml(path) {
  return YAML.parse(await readFile(path, "utf8"));
}

async function writeYaml(path, value) {
  await writeFile(path, YAML.stringify(value), "utf8");
}

export async function buildSmokeFixtures() {
  const snapshot = await readYaml(join(fixtureRoot, "source-snapshot.yaml"));
  const mappings = await readYaml(join(fixtureRoot, "mappings.yaml"));
  const server = snapshot.servers[0];
  const tool = server.tools[0];
  const provenance = sha256({ source_type: snapshot.source.type, locator: snapshot.source.locator });
  const contract = normalizeCapability(server, tool, mappings.mappings[0], "EVD-current-observation", provenance);
  contract.metadata.status = "approved";
  contract.metadata.reviewedAt = "2026-07-15T00:00:00Z";
  contract.assurance.evidenceRefs = [...mappings.mappings[0].evidenceRefs].sort();
  const approval = {
    approval_id: "APR-support-read-v1",
    contract_digest: sha256(contract),
    approved_implementation_digest: contract.assurance.approvedImplementationDigest,
    reviewed_by: "architecture-review-board",
    reviewed_at: "2026-07-15T00:00:00Z",
    expires_at: "2026-10-16T00:00:00Z",
    status: "current",
    review_source: "sanitized://review/support-read-v1"
  };
  await writeYaml(join(fixtureRoot, "approved-baseline.yaml"), {
    schema_version: "approved-baseline/v1alpha1",
    baseline_id: "support-read-prior-v1",
    reviewed_at: "2026-07-15T00:00:00Z",
    contracts: [contract],
    approvals: [approval]
  });
  await writeYaml(join(fixtureRoot, "evidence-ledger.yaml"), {
    schema_version: "evidence-ledger/v1alpha1",
    records: contract.assurance.evidenceRefs.map((evidence_id, index) => ({
      evidence_id,
      kind: index === 0 ? "implementation" : "evaluation",
      subject_digest: index === 0
        ? contract.assurance.approvedImplementationDigest
        : sha256({ approved_implementation_digest: contract.assurance.approvedImplementationDigest, eval_suite: contract.assurance.evalSuite }),
      payload_digest: "sha256:" + String(index + 1).repeat(64),
      observed_at: "2026-07-15T00:00:00Z",
      expires_at: "2026-10-16T00:00:00Z",
      reviewed_by: "architecture-review-board",
      status: "current"
    }))
  });
  await writeYaml(join(fixtureRoot, "equivalence-proofs.yaml"), { schema_version: "equivalence-proof/v1alpha1", proofs: [] });
  await mkdir(firstRunRoot, { recursive: true });
  for (const name of ["assessment-inputs.yaml", "source-snapshot.yaml", "mappings.yaml", "workflow-contract.yaml", "context.yaml", "targets.yaml", "evidence-ledger.yaml", "equivalence-proofs.yaml"]) {
    await cp(join(fixtureRoot, name), join(firstRunRoot, name), { force: true });
  }
  await writeYaml(join(firstRunRoot, "approved-baseline.yaml"), {
    schema_version: "approved-baseline/v1alpha1",
    baseline_id: "first-run-empty",
    reviewed_at: "2026-07-15T00:00:00Z",
    contracts: [],
    approvals: []
  });
}
~~~

- [ ] **Step 3: Implement explanations**

Create control-plane/src/explain/explain.mjs:

~~~js
export function explainBundle(bundle, { policy, contracts, driftResults, evidenceLedger, evaluation, targetArtifacts }) {
  const contractByImplementation = new Map(contracts.map((contract) => [contract.implementation.serverRef + "#" + contract.implementation.toolName, contract]));
  const driftByImplementation = new Map(driftResults.map((item) => [item.implementation, item]));
  const targetCoverage = Object.fromEntries(targetArtifacts.map((artifact) => [artifact.target, {
    status: (artifact.policy_enforcement || artifact).status || "proposal",
    enforcement_strength: (artifact.policy_enforcement || artifact).enforcement_strength,
    actionable_diff: (artifact.policy_enforcement || artifact).actionable_diff ?? true,
    preserves_ir: (artifact.policy_enforcement || artifact).preserves_ir ?? null,
    unsupported_semantics: (artifact.policy_enforcement || artifact).unsupported_semantics || []
  }]));
  return {
    included: bundle.included.map((item) => ({
      capability_id: item.capability_id,
      decision: "included",
      policy: {
        rule_id: policy.metadata.id,
        version: policy.metadata.version,
        digest: bundle.policy_digest,
        predicate: "capability included and context matched subject, agent, tenant, workflow, environment, purpose, data classes, and region"
      },
      contract: {
        version: item.contract_version,
        digest: item.contract_digest,
        implementation: item.implementation,
        approved_implementation_digest: item.approved_implementation_digest,
        reviewed_by: contractByImplementation.get(item.implementation).assurance.reviewedBy,
        expires_at: item.expires_at
      },
      evidence: {
        refs: item.evidence_refs,
        digest: item.evidence_digest,
        expires_at: item.evidence_expires_at,
        source_observed_at: item.source_observed_at,
        source_expires_at: item.source_expires_at,
        source_evidence_refs: item.source_evidence_refs,
        source_provenance_refs: item.source_provenance_refs,
        source_types: item.source_types,
        record_count: evidenceLedger.records.filter((record) => item.evidence_refs.includes(record.evidence_id)).length
      },
      evaluation: {
        suite: item.eval_suite,
        critical_false_permits: evaluation.critical_false_permits
      },
      drift: driftByImplementation.get(item.implementation),
      approval: {
        required: item.approval_required,
        binding_fields: contractByImplementation.get(item.implementation).policy.approval.binds
      },
      fallback: {
        equivalence_proof_digest: item.equivalence_proof_digest,
        mode: contractByImplementation.get(item.implementation).routing.fallback
      },
      targets: targetCoverage
    })),
    excluded: bundle.excluded.map((item) => ({
      capability_id: item.capability_id,
      decision: "excluded",
      reason_code: item.reason,
      failing_predicate: {
        prohibited: "policy.capabilities.exclude",
        missing_contract: "independent prior-approved contract exists",
        missing_primary: "exactly one reviewed primary exists",
        source_stale: "current source evidence is unexpired",
        drift_quarantined: "current protected projection equals approved projection",
        evidence_missing: "all reviewed evidence references resolve",
        evidence_stale: "all reviewed evidence is current",
        evidence_subject_mismatch: "evidence binds the approved implementation digest",
        equivalence_not_proven: "current versioned equivalence proof exists"
      }[item.reason] || item.reason,
      policy_digest: bundle.policy_digest
    }))
  };
}
~~~

Create control-plane/src/eval/pilot-gate.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { assertValid } from "../validate.mjs";

export const REQUIRED_PILOT_SCENARIOS = [
  "tool_selection", "prompt_poisoning", "tool_poisoning",
  "high_risk_approval", "high_risk_retry", "high_risk_fallback"
];
const HIGH_RISK_SCENARIOS = ["high_risk_approval", "high_risk_retry", "high_risk_fallback"];
const HIGH_RISK_ACTIONS = new Set(["internal_write", "external_write", "destructive"]);

export function traceResultDigest(record) {
  const { result_digest: ignored, ...result } = record;
  return sha256(result);
}

export function evaluateTraceSet(traceSet) {
  if (!traceSet) return {
    trace_count: 0,
    governance_surfaces: [],
    scenario_results: [],
    independent_suites: false,
    critical_false_permits: 0
  };
  assertValid("trace-set", traceSet);
  for (const record of traceSet.records) {
    if (record.result_digest !== traceResultDigest(record)) throw new Error("trace result digest mismatch: " + record.trace_id);
  }
  const suiteScenarios = new Map();
  const caseKeys = new Set();
  let independentSuites = traceSet.records.some((item) => REQUIRED_PILOT_SCENARIOS.includes(item.scenario));
  for (const record of traceSet.records.filter((item) => REQUIRED_PILOT_SCENARIOS.includes(item.scenario))) {
    const scenarios = suiteScenarios.get(record.suite_id) || new Set();
    scenarios.add(record.scenario);
    suiteScenarios.set(record.suite_id, scenarios);
    const caseKey = record.suite_id + "/" + record.case_id;
    if (caseKeys.has(caseKey)) independentSuites = false;
    caseKeys.add(caseKey);
  }
  if ([...suiteScenarios.values()].some((scenarios) => scenarios.size !== 1)) independentSuites = false;
  const scenarioResults = REQUIRED_PILOT_SCENARIOS.flatMap((scenario) => {
    const records = traceSet.records.filter((record) => record.scenario === scenario);
    if (!records.length) return [];
    const falsePermits = records.filter((record) => record.expected_decision !== "allow" && record.actual_decision === "allow").length;
    const scenarioInputValid = (record) => {
      if (scenario === "tool_selection") return record.selection_expected !== null && record.selected_implementation !== null;
      if (scenario === "prompt_poisoning") return record.prompt_poisoning === true;
      if (scenario === "tool_poisoning") return record.tool_poisoning === true;
      if (scenario === "high_risk_approval") return record.expected_decision === "require_approval";
      return record.expected_decision !== "allow";
    };
    const passed = records.every((record) => record.actual_decision === record.expected_decision
      && scenarioInputValid(record)
      && (scenario !== "tool_selection" || record.selected_implementation === record.selection_expected));
    return [{
      scenario,
      suite_ids: [...new Set(records.map((record) => record.suite_id))].sort(),
      case_count: records.length,
      passed,
      false_permits: falsePermits,
      governance_surfaces: [...new Set(records.map((record) => record.governance_surface))].sort(),
      contract_digests: [...new Set(records.map((record) => record.contract_digest).filter(Boolean))].sort(),
      action_classes: [...new Set(records.map((record) => record.action_class).filter(Boolean))].sort()
    }];
  });
  return {
    trace_count: traceSet.records.length,
    governance_surfaces: [...new Set(traceSet.records.map((record) => record.governance_surface))].sort(),
    scenario_results: scenarioResults,
    independent_suites: independentSuites,
    critical_false_permits: scenarioResults.reduce((sum, result) => sum + result.false_permits, 0)
  };
}

function implementationRef(contract) {
  return contract.implementation?.serverRef && contract.implementation?.toolName
    ? contract.implementation.serverRef + "#" + contract.implementation.toolName
    : null;
}

function targetSurface(artifact) {
  if (["host", "gateway"].includes(artifact.governance_surface)) return artifact.governance_surface;
  if (artifact.target === "openclaw") return "host";
  if (artifact.target === "docker-mcp-gateway") return "gateway";
  return null;
}

function highRiskBinding(scenarioResults, approvedByDigest) {
  const results = HIGH_RISK_SCENARIOS.map((scenario) => scenarioResults.get(scenario)).filter(Boolean);
  const digests = [...new Set(results.flatMap((result) => result.contract_digests))].sort();
  const actions = [...new Set(results.flatMap((result) => result.action_classes))].sort();
  const contractDigest = digests.length === 1 ? digests[0] : null;
  const actionClass = actions.length === 1 ? actions[0] : null;
  const contract = contractDigest ? approvedByDigest.get(contractDigest) : null;
  const riskTier = contract?.policy?.riskTier || null;
  const valid = results.length === HIGH_RISK_SCENARIOS.length
    && results.every((result) => result.passed && result.false_permits === 0 && result.contract_digests.length === 1 && result.action_classes.length === 1)
    && contract !== null
    && ["high", "critical"].includes(riskTier)
    && HIGH_RISK_ACTIONS.has(actionClass)
    && contract.semantics.actionClass === actionClass;
  return {
    valid,
    contract_digest: contractDigest,
    action_class: actionClass,
    risk_tier: riskTier,
    scenarios: results.map((result) => result.scenario).sort()
  };
}

export function evaluatePilotGate({
  mode, sources, servers, capabilities, traceEvaluation, criticalFalsePermits,
  approvedContracts, bundle, driftResults, targetArtifacts
}) {
  const sourceClasses = [...new Set(sources.map((item) => item.evidence.source_type))].sort();
  const scenarioResults = new Map(traceEvaluation.scenario_results.map((result) => [result.scenario, result]));
  const approvedByDigest = new Map(approvedContracts.map((contract) => [sha256(contract), contract]));
  const eligibleReadCount = bundle.included.filter((item) => {
    const contract = approvedByDigest.get(item.contract_digest);
    return contract?.semantics.actionClass === "read"
      && driftResults.some((drift) => drift.status === "current"
        && drift.capability_id === contract.metadata.id
        && (!drift.implementation || drift.implementation === item.implementation));
  }).length;
  const targetSurfaces = [...new Set(targetArtifacts.map(targetSurface).filter(Boolean))].sort();
  const criticalContracts = approvedContracts.filter((contract) => contract.policy.riskTier === "critical");
  const criticalQuarantines = driftResults.filter((drift) => drift.status === "quarantined" && (
    drift.severity === "critical"
    || drift.critical === true
    || criticalContracts.some((contract) => contract.metadata.id === drift.capability_id
      && (!drift.implementation || implementationRef(contract) === drift.implementation))
  )).length;
  const binding = highRiskBinding(scenarioResults, approvedByDigest);
  const scorecard = {
    schema_version: "pilot-scorecard/v1alpha1",
    status: "fail",
    authority_digests: {
      approved_contracts: sha256(approvedContracts),
      eligible_bundle: sha256(bundle),
      drift_results: sha256(driftResults),
      target_artifacts: sha256(targetArtifacts)
    },
    approved_contract_count: approvedContracts.length,
    eligible_read_count: eligibleReadCount,
    target_surfaces: targetSurfaces,
    critical_quarantines: criticalQuarantines,
    high_risk_binding: binding,
    source_count: new Set(sources.map((item) => item.evidence.provenance_digest)).size,
    source_classes: sourceClasses,
    server_count: new Set(servers.map((item) => item.server_ref)).size,
    capability_count: new Set(capabilities.map((item) => item.metadata.id)).size,
    trace_count: traceEvaluation.trace_count,
    governance_surfaces: [...traceEvaluation.governance_surfaces],
    scenario_results: traceEvaluation.scenario_results,
    independent_suites: traceEvaluation.independent_suites,
    critical_false_permits: criticalFalsePermits + traceEvaluation.critical_false_permits,
    reasons: []
  };
  if (mode === "smoke") {
    scorecard.status = "not_evaluated";
    scorecard.reasons = ["smoke_fixture_is_not_pilot_evidence"];
    return assertValid("pilot-scorecard", scorecard);
  }
  if (scorecard.approved_contract_count === 0) scorecard.reasons.push("requires_prior_approved_contracts");
  if (scorecard.eligible_read_count === 0) scorecard.reasons.push("requires_eligible_read");
  if (!["host", "gateway"].every((surface) => scorecard.target_surfaces.includes(surface))) scorecard.reasons.push("requires_two_authoritative_target_surfaces");
  if (scorecard.critical_quarantines !== 0) scorecard.reasons.push("critical_contract_quarantined");
  if (!scorecard.high_risk_binding.valid) scorecard.reasons.push("requires_same_approved_high_risk_write_binding");
  if (!["official_registry", "private_registry", "live_schema_export"].every((sourceClass) => sourceClasses.includes(sourceClass))) scorecard.reasons.push("requires_official_private_live_sources");
  if (scorecard.governance_surfaces.length < 2) scorecard.reasons.push("requires_two_governance_surfaces");
  if (scorecard.server_count < 3 || scorecard.server_count > 5) scorecard.reasons.push("requires_3_to_5_servers");
  if (scorecard.capability_count < 10 || scorecard.capability_count > 30) scorecard.reasons.push("requires_10_to_30_capabilities");
  if (scorecard.trace_count === 0) scorecard.reasons.push("requires_sanitized_traces");
  if (!scorecard.independent_suites) scorecard.reasons.push("requires_independent_suite_and_case_ids");
  for (const scenario of REQUIRED_PILOT_SCENARIOS) {
    if (!scenarioResults.get(scenario)?.passed) scorecard.reasons.push("requires_passing_" + scenario + "_suite");
  }
  if (scorecard.critical_false_permits !== 0) scorecard.reasons.push("critical_false_permit_detected");
  scorecard.status = scorecard.reasons.length ? "fail" : "pass";
  return assertValid("pilot-scorecard", scorecard);
}
~~~

Create control-plane/tests/pilot-gate.test.mjs:

~~~js
import assert from "node:assert/strict";
import test from "node:test";
import { sha256 } from "../src/canonical-json.mjs";
import { evaluatePilotGate, evaluateTraceSet, traceResultDigest } from "../src/eval/pilot-gate.mjs";

const sources = [
  ["official_registry", "a"],
  ["private_registry", "b"],
  ["live_schema_export", "c"]
].map(([source_type, value]) => ({ evidence: { source_type, provenance_digest: "sha256:" + value.repeat(64) } }));
const servers = [1, 2, 3].map((value) => ({ server_ref: "server-" + value }));
const capabilities = Array.from({ length: 10 }, (_, index) => ({ metadata: { id: "capability." + index } }));
const readContract = {
  metadata: { id: "customer.case.read" },
  implementation: { serverRef: "server-1", toolName: "get_case" },
  semantics: { actionClass: "read" },
  policy: { riskTier: "low" }
};
const highRiskContract = {
  metadata: { id: "communication.email.send" },
  implementation: { serverRef: "server-2", toolName: "send_email" },
  semantics: { actionClass: "external_write" },
  policy: { riskTier: "critical" }
};
const readContractDigest = sha256(readContract);
const highRiskContractDigest = sha256(highRiskContract);
const approvedContracts = [readContract, highRiskContract];
const bundle = {
  schema_version: "capability-bundle/v1alpha1",
  included: [{ capability_id: readContract.metadata.id, contract_digest: readContractDigest, implementation: "server-1#get_case" }],
  excluded: []
};
const driftResults = [
  { capability_id: readContract.metadata.id, implementation: "server-1#get_case", status: "current" },
  { capability_id: highRiskContract.metadata.id, implementation: "server-2#send_email", status: "current" }
];
const targetArtifacts = [
  { target: "openclaw", policy_enforcement: { status: "unsupported", preserves_ir: false } },
  { target: "docker-mcp-gateway", status: "unsupported" }
];

function result(scenario, index, expectedDecision, actualDecision = expectedDecision) {
  const highRisk = scenario.startsWith("high_risk_");
  const record = {
    trace_id: "trace-" + index,
    suite_id: "suite-" + scenario,
    case_id: "case-" + index,
    governance_surface: index % 2 ? "host" : "gateway",
    runner: "offline-policy-simulator@1.0.0",
    source_ref: "sanitized://trace/" + index,
    tenant_ref: "tenant-a",
    workflow: "support-case-resolution/inspect-case",
    capability: highRisk ? highRiskContract.metadata.id : readContract.metadata.id,
    contract_digest: highRisk ? highRiskContractDigest : readContractDigest,
    action_class: highRisk ? highRiskContract.semantics.actionClass : readContract.semantics.actionClass,
    scenario,
    selected_implementation: scenario === "tool_selection" ? "server-1#get_case" : null,
    selection_expected: scenario === "tool_selection" ? "server-1#get_case" : null,
    expected_decision: expectedDecision,
    actual_decision: actualDecision,
    outcome: "success",
    failure_class: "none",
    attempt: 1,
    prompt_poisoning: scenario === "prompt_poisoning",
    tool_poisoning: scenario === "tool_poisoning",
    observed_at: "2026-07-16T00:00:00Z"
  };
  return { ...record, result_digest: traceResultDigest(record) };
}

function completeTraceSet() {
  return {
    schema_version: "trace-set/v1alpha1",
    sanitized: true,
    records: [
      result("tool_selection", 1, "allow"),
      result("prompt_poisoning", 2, "deny"),
      result("tool_poisoning", 3, "deny"),
      result("high_risk_approval", 4, "require_approval"),
      result("high_risk_retry", 5, "deny"),
      result("high_risk_fallback", 6, "deny")
    ]
  };
}

function evaluateGate(traceSet = completeTraceSet(), overrides = {}) {
  return evaluatePilotGate({
    mode: "pilot",
    sources,
    servers,
    capabilities,
    traceEvaluation: evaluateTraceSet(traceSet),
    criticalFalsePermits: 0,
    approvedContracts,
    bundle,
    driftResults,
    targetArtifacts,
    ...overrides
  });
}

test("RFC MVP pilot gate passes only at the complete technical boundary", () => {
  const scorecard = evaluateGate();
  assert.equal(scorecard.status, "pass");
  assert.deepEqual(scorecard.authority_digests, {
    approved_contracts: sha256(approvedContracts),
    eligible_bundle: sha256(bundle),
    drift_results: sha256(driftResults),
    target_artifacts: sha256(targetArtifacts)
  });
  assert.deepEqual(scorecard.high_risk_binding, {
    valid: true,
    contract_digest: highRiskContractDigest,
    action_class: "external_write",
    risk_tier: "critical",
    scenarios: ["high_risk_approval", "high_risk_fallback", "high_risk_retry"]
  });
});

test("a missing independently evaluated scenario fails the pilot gate", () => {
  const traces = completeTraceSet();
  traces.records = traces.records.filter((record) => record.scenario !== "tool_poisoning");
  const scorecard = evaluateGate(traces);
  assert.equal(scorecard.status, "fail");
  assert.ok(scorecard.reasons.includes("requires_passing_tool_poisoning_suite"));
});

test("reported actual results are digest-bound and evaluated rather than label-counted", () => {
  const traces = completeTraceSet();
  traces.records[1].actual_decision = "allow";
  assert.throws(() => evaluateTraceSet(traces), /trace result digest mismatch/);
  traces.records[1].result_digest = traceResultDigest(traces.records[1]);
  const scorecard = evaluateGate(traces);
  assert.equal(scorecard.status, "fail");
  assert.equal(scorecard.critical_false_permits, 1);
});

test("a scenario label without the matching evaluated condition cannot pass", () => {
  const traces = completeTraceSet();
  const promptCase = traces.records.find((record) => record.scenario === "prompt_poisoning");
  promptCase.prompt_poisoning = false;
  promptCase.result_digest = traceResultDigest(promptCase);
  const scorecard = evaluateGate(traces);
  assert.equal(scorecard.status, "fail");
  assert.ok(scorecard.reasons.includes("requires_passing_prompt_poisoning_suite"));
});

test("one suite ID cannot self-certify several required scenarios", () => {
  const traces = completeTraceSet();
  traces.records[1].suite_id = traces.records[0].suite_id;
  traces.records[1].result_digest = traceResultDigest(traces.records[1]);
  const scorecard = evaluateGate(traces);
  assert.equal(scorecard.status, "fail");
  assert.ok(scorecard.reasons.includes("requires_independent_suite_and_case_ids"));
});

test("official, private, live, and two governance surfaces are mandatory", () => {
  const oneSurface = completeTraceSet();
  oneSurface.records = oneSurface.records.map((record) => {
    const changed = { ...record, governance_surface: "host" };
    return { ...changed, result_digest: traceResultDigest(changed) };
  });
  const scorecard = evaluateGate(oneSurface, { sources: sources.slice(1), targetArtifacts: targetArtifacts.slice(0, 1) });
  assert.equal(scorecard.status, "fail");
  assert.ok(scorecard.reasons.includes("requires_official_private_live_sources"));
  assert.ok(scorecard.reasons.includes("requires_two_governance_surfaces"));
  assert.ok(scorecard.reasons.includes("requires_two_authoritative_target_surfaces"));
});

test("first-run pilot evidence cannot pass without prior-approved authority", () => {
  const scorecard = evaluateGate(completeTraceSet(), {
    approvedContracts: [],
    bundle: { schema_version: "capability-bundle/v1alpha1", included: [], excluded: [] },
    driftResults: []
  });
  assert.equal(scorecard.status, "fail");
  assert.ok(scorecard.reasons.includes("requires_prior_approved_contracts"));
  assert.ok(scorecard.reasons.includes("requires_eligible_read"));
  assert.ok(scorecard.reasons.includes("requires_same_approved_high_risk_write_binding"));
});

test("an all-quarantined pilot cannot pass and reports critical quarantine", () => {
  const scorecard = evaluateGate(completeTraceSet(), {
    bundle: { schema_version: "capability-bundle/v1alpha1", included: [], excluded: [] },
    driftResults: driftResults.map((result) => ({ ...result, status: "quarantined" }))
  });
  assert.equal(scorecard.status, "fail");
  assert.ok(scorecard.reasons.includes("requires_eligible_read"));
  assert.ok(scorecard.reasons.includes("critical_contract_quarantined"));
});

test("read-labeled high-risk scenarios cannot satisfy write safety evidence", () => {
  const traces = completeTraceSet();
  for (const record of traces.records.filter((item) => item.scenario.startsWith("high_risk_"))) {
    record.capability = readContract.metadata.id;
    record.contract_digest = readContractDigest;
    record.action_class = "read";
    record.result_digest = traceResultDigest(record);
  }
  const scorecard = evaluateGate(traces);
  assert.equal(scorecard.status, "fail");
  assert.equal(scorecard.high_risk_binding.action_class, "read");
  assert.ok(scorecard.reasons.includes("requires_same_approved_high_risk_write_binding"));
});

test("smoke mode can never claim a pilot pass", () => {
  assert.equal(evaluateGate(completeTraceSet(), { mode: "smoke" }).status, "not_evaluated");
});
~~~

Create control-plane/tests/observation-aggregation.test.mjs:

~~~js
import assert from "node:assert/strict";
import test from "node:test";
import { aggregateImplementationObservations } from "../src/drift/aggregate-observations.mjs";

const digest = (character) => "sha256:" + character.repeat(64);
function observation(sourceType, marker, implementationDigest = digest("9")) {
  return {
    source: { evidence: {
      source_type: sourceType,
      observed_at: "2026-07-16T00:00:00Z",
      expires_at: "2026-07-23T00:00:00Z",
      digest: digest(marker),
      provenance_digest: digest(marker === "a" ? "c" : "d")
    } },
    server: { server_ref: "com.example/support@1.0.0" },
    tool: { name: "get_case" },
    candidate: {
      metadata: { id: "customer.case.read" },
      implementation: { serverRef: "com.example/support@1.0.0", toolName: "get_case" },
      assurance: {
        approvedImplementation: { name: implementationDigest === digest("9") ? "get_case" : "changed_case", source_provenance_ref: digest(marker === "a" ? "c" : "d") },
        approvedImplementationDigest: implementationDigest
      }
    }
  };
}

test("same implementation from several sources uses precedence and retains all evidence", () => {
  let selectedProvenance;
  const observations = [observation("official_registry", "a"), observation("live_schema_export", "b")];
  const result = aggregateImplementationObservations(observations, { get: () => ({ metadata: { id: "customer.case.read" } }) }, (_approved, _server, _tool, provenance) => {
    selectedProvenance = provenance;
    return { status: "current", changed_fields: [], approved_digest: digest("9"), observed_digest: digest("9") };
  });
  const eligibility = result.eligibilityByImplementation["com.example/support@1.0.0#get_case"];
  assert.equal(selectedProvenance, digest("d"));
  assert.deepEqual(eligibility.source_types, ["live_schema_export", "official_registry"]);
  assert.deepEqual(eligibility.source_evidence_refs, [digest("a"), digest("b")]);
  assert.equal(result.driftResults[0].status, "current");
});

test("conflicting protected projections are quarantined instead of throwing", () => {
  const observations = [observation("official_registry", "a", digest("8")), observation("live_schema_export", "b", digest("9"))];
  const result = aggregateImplementationObservations(observations, { get: () => null }, () => { throw new Error("conflict must not select a source"); });
  const eligibility = result.eligibilityByImplementation["com.example/support@1.0.0#get_case"];
  assert.equal(eligibility.drift_status, "quarantined");
  assert.deepEqual(result.driftResults[0].changed_fields, ["multi_source_conflict"]);
  assert.deepEqual(result.driftResults[0].source_evidence_refs, [digest("a"), digest("b")]);
});
~~~

Create control-plane/src/drift/aggregate-observations.mjs:

~~~js
import { sha256 } from "../canonical-json.mjs";
import { analyzeDrift } from "./analyze.mjs";

export const SOURCE_PRECEDENCE = Object.freeze({
  live_schema_export: 5,
  private_registry: 4,
  official_registry: 3,
  gateway_config: 2,
  host_config: 1
});

function implementationRef(candidate) {
  return candidate.implementation.serverRef + "#" + candidate.implementation.toolName;
}

function aggregateEvidence(group) {
  const observedTimes = group.map((item) => Date.parse(item.source.evidence.observed_at));
  const expiryTimes = group.map((item) => Date.parse(item.source.evidence.expires_at));
  if ([...observedTimes, ...expiryTimes].some((value) => !Number.isFinite(value))) throw new Error("invalid source evidence time");
  return {
    source_observed_at: new Date(Math.max(...observedTimes)).toISOString(),
    source_expires_at: new Date(Math.min(...expiryTimes)).toISOString(),
    source_evidence_refs: [...new Set(group.map((item) => item.source.evidence.digest))].sort(),
    source_provenance_refs: [...new Set(group.map((item) => item.source.evidence.provenance_digest))].sort(),
    source_types: [...new Set(group.map((item) => item.source.evidence.source_type))].sort()
  };
}

function corroborationProjectionDigest(candidate) {
  const projection = candidate.assurance.approvedImplementation;
  if (!projection) return candidate.assurance.approvedImplementationDigest;
  const { source_provenance_ref: sourceSpecificEvidence, ...implementationSemantics } = projection;
  return sha256(implementationSemantics);
}

export function aggregateImplementationObservations(observations, approvedStore, driftAnalyzer = analyzeDrift) {
  const groups = new Map();
  for (const observation of observations) {
    const reference = implementationRef(observation.candidate);
    const group = groups.get(reference) || [];
    group.push(observation);
    groups.set(reference, group);
  }
  const eligibilityByImplementation = {};
  const driftResults = [];
  for (const [implementation, unsorted] of [...groups].sort(([left], [right]) => left.localeCompare(right))) {
    const group = [...unsorted].sort((left, right) =>
      (SOURCE_PRECEDENCE[right.source.evidence.source_type] || 0) - (SOURCE_PRECEDENCE[left.source.evidence.source_type] || 0)
      || left.source.evidence.provenance_digest.localeCompare(right.source.evidence.provenance_digest));
    const primary = group[0];
    const evidence = aggregateEvidence(group);
    const projectionDigests = [...new Set(group.map((item) => corroborationProjectionDigest(item.candidate)))].sort();
    if (projectionDigests.length !== 1) {
      eligibilityByImplementation[implementation] = { drift_status: "quarantined", ...evidence };
      driftResults.push({
        capability_id: primary.candidate.metadata.id,
        implementation,
        status: "quarantined",
        reason: "multi_source_conflict",
        changed_fields: ["multi_source_conflict"],
        approved_digest: null,
        observed_digest: sha256(projectionDigests),
        selected_source_type: null,
        ...evidence
      });
      continue;
    }
    const approved = approvedStore.get(primary.candidate.metadata.id, implementation);
    if (!approved) {
      driftResults.push({
        capability_id: primary.candidate.metadata.id,
        implementation,
        status: "unapproved_candidate",
        changed_fields: [],
        approved_digest: null,
        observed_digest: primary.candidate.assurance.approvedImplementationDigest,
        selected_source_type: primary.source.evidence.source_type,
        ...evidence
      });
      continue;
    }
    const drift = driftAnalyzer(approved, primary.server, primary.tool, primary.source.evidence.provenance_digest);
    eligibilityByImplementation[implementation] = { drift_status: drift.status, ...evidence };
    driftResults.push({
      capability_id: approved.metadata.id,
      implementation,
      ...drift,
      selected_source_type: primary.source.evidence.source_type,
      ...evidence
    });
  }
  return { eligibilityByImplementation, driftResults };
}
~~~

Create control-plane/tests/verify-artifacts.test.mjs:

~~~js
import assert from "node:assert/strict";
import { mkdtemp, writeFile } from "node:fs/promises";
import { tmpdir } from "node:os";
import { join } from "node:path";
import test from "node:test";
import { describeArtifactBytes, verifyArtifactDigests } from "../src/evidence/verify-artifacts.mjs";

test("artifact manifest binds the exact persisted bytes and parsed semantics", async () => {
  const root = await mkdtemp(join(tmpdir(), "artifact-digests-"));
  const relativePath = "result.yaml";
  const bytes = Buffer.from("answer: 42\n", "utf8");
  await writeFile(join(root, relativePath), bytes);
  const digests = { [relativePath]: describeArtifactBytes(relativePath, bytes) };
  await assert.doesNotReject(() => verifyArtifactDigests(root, digests));
  await writeFile(join(root, relativePath), Buffer.from("answer: 42\n\n", "utf8"));
  await assert.rejects(() => verifyArtifactDigests(root, digests), /file_sha256 mismatch/);
});
~~~

Create control-plane/src/evidence/verify-artifacts.mjs:

~~~js
import { createHash } from "node:crypto";
import { readFile } from "node:fs/promises";
import YAML from "yaml";
import { sha256 } from "../canonical-json.mjs";
import { resolveSanitizedPath } from "./sanitized-boundary.mjs";

export function fileSha256(bytes) {
  return "sha256:" + createHash("sha256").update(bytes).digest("hex");
}

function semanticValue(relativePath, bytes) {
  const text = new TextDecoder("utf-8", { fatal: true }).decode(bytes);
  if (relativePath.endsWith(".yaml") || relativePath.endsWith(".yml")) return YAML.parse(text);
  if (relativePath.endsWith(".json")) return JSON.parse(text);
  return text;
}

export function describeArtifactBytes(relativePath, bytes) {
  return {
    semantic_digest: sha256(semanticValue(relativePath, bytes)),
    file_sha256: fileSha256(bytes)
  };
}

export async function verifyArtifactDigests(outputDirectory, artifactDigests) {
  for (const [relativePath, expected] of Object.entries(artifactDigests)) {
    const { path } = await resolveSanitizedPath(outputDirectory, relativePath, "file");
    const bytes = await readFile(path);
    const actual = describeArtifactBytes(relativePath, bytes);
    if (actual.file_sha256 !== expected.file_sha256) throw new Error("artifact file_sha256 mismatch: " + relativePath);
    if (actual.semantic_digest !== expected.semantic_digest) throw new Error("artifact semantic_digest mismatch: " + relativePath);
  }
  return true;
}
~~~

- [ ] **Step 4: Implement assessment assembly**

Create control-plane/src/assessment.mjs:

~~~js
import { mkdir, writeFile } from "node:fs/promises";
import { join, resolve } from "node:path";
import { fileURLToPath } from "node:url";
import YAML from "yaml";
import { sha256 } from "./canonical-json.mjs";
import { snapshotFromDocument } from "./evidence/load-snapshot.mjs";
import { loadSanitizedInput, resolveSanitizedPath } from "./evidence/sanitized-boundary.mjs";
import { describeArtifactBytes, verifyArtifactDigests } from "./evidence/verify-artifacts.mjs";
import { normalizeCapability } from "./contracts/normalize-capability.mjs";
import { loadApprovedBaseline } from "./contracts/load-approved-baseline.mjs";
import { compilePolicy } from "./compile/policy.mjs";
import { compileBundle } from "./compile/bundle.mjs";
import { compileOpenClaw } from "./adapters/openclaw.mjs";
import { compileDocker } from "./adapters/docker.mjs";
import { aggregateImplementationObservations, SOURCE_PRECEDENCE } from "./drift/aggregate-observations.mjs";
import { evaluateComposition, simulateDecision } from "./eval/simulate.mjs";
import { ReplayLedger } from "./eval/replay-ledger.mjs";
import { evaluatePilotGate, evaluateTraceSet } from "./eval/pilot-gate.mjs";
import { explainBundle } from "./explain/explain.mjs";
import { importArchitecturePack } from "./import/architecture-pack.mjs";
import { assertValid } from "./validate.mjs";

async function writeYaml(path, value) {
  await writeFile(path, YAML.stringify(value), "utf8");
}

function inputRoot(inputDirectory) {
  return resolve(inputDirectory instanceof URL ? fileURLToPath(inputDirectory) : String(inputDirectory));
}

async function childDirectory(root, relativePath) {
  return (await resolveSanitizedPath(root, relativePath, "directory")).path;
}

function implementationRef(contract) {
  return contract.implementation.serverRef + "#" + contract.implementation.toolName;
}

function noOpenClawSuggestion(reason, targetBaseline) {
  return {
    target: "openclaw",
    target_version: targetBaseline.version,
    current_config_digest: targetBaseline.currentConfigDigest,
    conformance_suggestion: { status: "not_generated", reason, suggested_config: null, changes: [], exposure_gaps: [] },
    policy_enforcement: {
      status: "unsupported",
      enforcement_strength: "unsupported",
      preserves_ir: false,
      proposal: null,
      changes: [],
      actionable_diff: false,
      unsupported_semantics: ["argument_binding", "data_flow_and_redaction", "per_request_approval", "purpose"]
    }
  };
}

export async function assess(inputDirectory, outputDirectory) {
  const root = inputRoot(inputDirectory);
  const inputManifestDocument = await loadSanitizedInput(root, "assessment-inputs.yaml", "assessment-inputs");
  const inputManifest = inputManifestDocument.value;
  const sourceDocuments = await Promise.all(inputManifest.sourceSnapshots.map((path) => loadSanitizedInput(root, path, "source-snapshot")));
  const sources = sourceDocuments.map(snapshotFromDocument);
  const [mappingDocument, workflowDocument, contextDocument, targetDocument, baselineDocument, evidenceDocument, equivalenceDocument] = await Promise.all([
    loadSanitizedInput(root, inputManifest.mappings, "capability-mappings"),
    loadSanitizedInput(root, inputManifest.workflow, "workflow-contract"),
    loadSanitizedInput(root, inputManifest.context, "assessment-context"),
    loadSanitizedInput(root, inputManifest.targets, "target-baselines"),
    loadSanitizedInput(root, inputManifest.approvedBaseline, "approved-baseline"),
    loadSanitizedInput(root, inputManifest.evidenceLedger, "evidence-ledger"),
    loadSanitizedInput(root, inputManifest.equivalenceProofs, "equivalence-proof")
  ]);
  const traceDocument = inputManifest.traces ? await loadSanitizedInput(root, inputManifest.traces, "trace-set") : null;
  const architectureImport = inputManifest.architecturePack ? await importArchitecturePack(await childDirectory(root, inputManifest.architecturePack)) : null;
  const mappings = assertValid("capability-mappings", mappingDocument.value);
  const workflow = assertValid("workflow-contract", workflowDocument.value);
  const contextValue = assertValid("assessment-context", contextDocument.value);
  const { schema_version: contextSchemaVersion, ...context } = contextValue;
  if (contextSchemaVersion !== "assessment-context/v1alpha1") throw new Error("unsupported assessment context");
  const targets = assertValid("target-baselines", targetDocument.value);
  if (sha256(targets.openclaw.currentConfig) !== targets.openclaw.currentConfigDigest || sha256(targets.docker.currentConfig) !== targets.docker.currentConfigDigest) throw new Error("target baseline digest mismatch");
  const approvedStore = loadApprovedBaseline(baselineDocument.value, sources.map((source) => source.evidence), context.now);
  const evidenceLedger = assertValid("evidence-ledger", evidenceDocument.value);
  const equivalenceProofs = assertValid("equivalence-proof", equivalenceDocument.value).proofs;
  const inventoryServers = new Map();
  const sourcesByPrecedence = [...sources].sort((left, right) =>
    (SOURCE_PRECEDENCE[right.evidence.source_type] || 0) - (SOURCE_PRECEDENCE[left.evidence.source_type] || 0)
    || left.evidence.provenance_digest.localeCompare(right.evidence.provenance_digest));
  for (const source of sourcesByPrecedence) {
    for (const server of source.snapshot.servers) {
      const existing = inventoryServers.get(server.server_ref);
      if (!existing) {
        inventoryServers.set(server.server_ref, {
          server_ref: server.server_ref,
          protocol_version: server.protocol_version,
          endpoint_class: server.endpoint_class,
          tools: new Set(server.tools.map((tool) => tool.name))
        });
      } else {
        for (const tool of server.tools) existing.tools.add(tool.name);
      }
    }
  }
  const sourceInventory = assertValid("source-inventory", {
    schema_version: "source-inventory/v1alpha1",
    evidence: sources.map((source) => source.evidence),
    servers: [...inventoryServers.values()].map((server) => ({ ...server, tools: [...server.tools].sort() }))
  });
  const observations = sources.flatMap((source) => source.snapshot.servers.flatMap((server) => server.tools.map((tool) => ({ source, server, tool }))));
  for (const observation of observations) {
    const { source, server, tool } = observation;
    const mapping = mappings.mappings.find((item) => item.serverRef === server.server_ref && item.toolName === tool.name);
    observation.candidate = normalizeCapability(server, tool, mapping, source.evidence.digest, source.evidence.provenance_digest);
  }
  const candidates = observations.map((observation) => observation.candidate);
  const { eligibilityByImplementation, driftResults } = aggregateImplementationObservations(observations, approvedStore);
  const policy = compilePolicy(workflow);
  const bundle = compileBundle(policy, approvedStore, context, evidenceLedger, eligibilityByImplementation, equivalenceProofs);
  const approvedContracts = approvedStore.all();
  const openclaw = bundle.included.length
    ? compileOpenClaw(bundle, approvedContracts, sourceInventory, targets.openclaw)
    : noOpenClawSuggestion("no_eligible_capabilities", targets.openclaw);
  const docker = compileDocker(bundle, approvedContracts, sourceInventory, targets.docker);
  const requestContext = {
    subject: context.subject,
    agent: context.agent,
    tenant: context.tenant,
    environment: context.environment,
    workflow: context.workflow + "/" + context.stage,
    purpose: context.purpose,
    region: context.region,
    now: context.now,
    arguments: {}
  };
  const authority = {
    evidenceLedger,
    eligibilityByImplementation,
    equivalenceProofs,
    replayLedger: new ReplayLedger()
  };
  const decisionResults = [
    ...bundle.included.map((item) => ({
      capability_id: item.capability_id,
      expected: item.approval_required ? "require_approval" : "allow",
      actual: simulateDecision({ ...requestContext, capability: item.capability_id, approval: null }, policy, bundle, approvedContracts, authority).decision
    })),
    ...bundle.excluded.map((item) => ({
      capability_id: item.capability_id,
      expected: "deny",
      actual: simulateDecision({ ...requestContext, capability: item.capability_id, approval: null }, policy, bundle, approvedContracts, authority).decision
    }))
  ];
  const traceEvaluation = evaluateTraceSet(traceDocument?.value || null);
  const evaluation = {
    critical_false_permits: decisionResults.filter((item) => item.expected !== "allow" && item.actual === "allow").length,
    decision_results: decisionResults,
    composition_findings: evaluateComposition(approvedContracts),
    trace_evaluation: traceEvaluation
  };
  if (evaluation.critical_false_permits > 0) throw new Error("critical false permit detected");
  const targetArtifacts = [openclaw, docker];
  const pilotScorecard = evaluatePilotGate({
    mode: inputManifest.mode,
    sources,
    servers: sourceInventory.servers,
    capabilities: candidates,
    traceEvaluation,
    criticalFalsePermits: evaluation.critical_false_permits,
    approvedContracts,
    bundle,
    driftResults,
    targetArtifacts
  });
  const explanations = explainBundle(bundle, { policy, contracts: approvedContracts, driftResults, evidenceLedger, evaluation, targetArtifacts });
  const artifactDigests = {};
  async function writeYamlArtifact(relativePath, value) {
    const bytes = Buffer.from(YAML.stringify(value), "utf8");
    await writeFile(join(outputDirectory, relativePath), bytes);
    artifactDigests[relativePath] = describeArtifactBytes(relativePath, bytes);
  }
  async function writeTextArtifact(relativePath, value) {
    const bytes = Buffer.from(value, "utf8");
    await writeFile(join(outputDirectory, relativePath), bytes);
    artifactDigests[relativePath] = describeArtifactBytes(relativePath, bytes);
  }

  await mkdir(join(outputDirectory, "candidate-contracts"), { recursive: true });
  await mkdir(join(outputDirectory, "approved-contracts"), { recursive: true });
  await mkdir(join(outputDirectory, "workflow-contracts"), { recursive: true });
  await mkdir(join(outputDirectory, "policy-ir"), { recursive: true });
  await mkdir(join(outputDirectory, "bundles"), { recursive: true });
  await mkdir(join(outputDirectory, "target-diffs"), { recursive: true });
  if (architectureImport) await mkdir(join(outputDirectory, "part1-candidates"), { recursive: true });
  const candidatePaths = observations.map(({ candidate, source }) => "candidate-contracts/" + candidate.metadata.id + "." + sha256({ implementation: candidate.implementation, source_digest: source.evidence.digest }).slice(7, 19) + ".yaml");
  const approvedPaths = approvedContracts.map((contract) => "approved-contracts/" + contract.metadata.id + "." + sha256(contract.implementation).slice(7, 19) + ".yaml");
  for (let index = 0; index < candidates.length; index += 1) await writeYamlArtifact(candidatePaths[index], candidates[index]);
  for (let index = 0; index < approvedContracts.length; index += 1) await writeYamlArtifact(approvedPaths[index], approvedContracts[index]);
  const stem = workflow.metadata.workflowId + "." + workflow.metadata.stageId;
  await writeYamlArtifact("source-inventory.yaml", sourceInventory);
  await writeYamlArtifact("workflow-contracts/" + stem + ".yaml", workflow);
  await writeYamlArtifact("policy-ir/" + stem + ".yaml", policy);
  await writeYamlArtifact("bundles/" + stem + ".yaml", bundle);
  await writeYamlArtifact("target-diffs/openclaw.yaml", openclaw);
  await writeYamlArtifact("target-diffs/docker.yaml", docker);
  if (architectureImport) {
    await writeYamlArtifact("architecture-pack-import.yaml", architectureImport);
    await writeYamlArtifact("part1-candidates/mappings.yaml", architectureImport.mapping_candidates);
    await writeYamlArtifact("part1-candidates/workflows.yaml", architectureImport.workflow_candidates);
  }
  await writeTextArtifact("drift-report.md", "# Drift Report\n\n" + JSON.stringify(driftResults, null, 2) + "\n");
  await writeTextArtifact("evaluation-report.md", "# Evaluation Report\n\n" + JSON.stringify(evaluation, null, 2) + "\n");
  await writeYamlArtifact("pilot-scorecard.yaml", pilotScorecard);
  await writeTextArtifact("assurance-gaps.md", "# Assurance Gaps\n\n- Runtime enforcement was not tested.\n- Credential scanning is best-effort and does not prove sanitization.\n- Approval authenticity was not verified; digest, nonce, and replay behavior is offline simulation only.\n- Missing or redacted traces cap outcome assurance.\n- OpenClaw emits only a non-authoritative conformance suggestion; mandatory purpose, data-flow/redaction, per-request approval, and argument-binding enforcement remain unsupported and the output is not evidence of Policy IR preservation.\n- Docker purpose, data-flow/redaction, and argument binding are unsupported, so no Docker proposal is emitted.\n");
  await writeTextArtifact("decision-explanations.json", JSON.stringify(explanations, null, 2) + "\n");
  await verifyArtifactDigests(outputDirectory, artifactDigests);

  const manifest = assertValid("assessment-manifest", {
    schema_version: "assessment-manifest/v1alpha1",
    generated_at: context.now,
    input_digests: Object.fromEntries([
      ["assessment-inputs.yaml", inputManifestDocument.digest],
      ...inputManifest.sourceSnapshots.map((path, index) => [path, sourceDocuments[index].digest]),
      [inputManifest.mappings, mappingDocument.digest],
      [inputManifest.workflow, workflowDocument.digest],
      [inputManifest.context, contextDocument.digest],
      [inputManifest.targets, targetDocument.digest],
      [inputManifest.approvedBaseline, baselineDocument.digest],
      [inputManifest.evidenceLedger, evidenceDocument.digest],
      [inputManifest.equivalenceProofs, equivalenceDocument.digest],
      ...(traceDocument ? [[inputManifest.traces, traceDocument.digest]] : []),
      ...(architectureImport ? [[inputManifest.architecturePack + "/manifest.yaml", architectureImport.source_pack_digest]] : [])
    ]),
    artifact_digests: artifactDigests,
    source_inventory: "source-inventory.yaml",
    candidate_contracts: candidatePaths,
    approved_contracts: approvedPaths,
    architecture_import: architectureImport ? "architecture-pack-import.yaml" : null,
    policy: "policy-ir/" + stem + ".yaml",
    bundle: "bundles/" + stem + ".yaml",
    target_artifacts: ["target-diffs/openclaw.yaml", "target-diffs/docker.yaml"],
    drift_report: "drift-report.md",
    evaluation_report: "evaluation-report.md",
    pilot_scorecard: "pilot-scorecard.yaml",
    pilot_authority_digests: pilotScorecard.authority_digests,
    assurance_gaps: "assurance-gaps.md",
    explanations: "decision-explanations.json"
  });
  await writeYaml(join(outputDirectory, "evidence-manifest.yaml"), manifest);
  return manifest;
}
~~~

- [ ] **Step 5: Implement the CLI**

Create control-plane/src/cli.mjs:

~~~js
#!/usr/bin/env node
import { resolve } from "node:path";
import { assess } from "./assessment.mjs";

async function main(argv) {
  if (argv.length !== 3 || argv[0] !== "assess") {
    process.stderr.write("usage: node src/cli.mjs assess INPUT_DIRECTORY OUTPUT_DIRECTORY\n");
    return 2;
  }
  const manifest = await assess(resolve(argv[1]), resolve(argv[2]));
  process.stdout.write(JSON.stringify({ ok: true, manifest }, null, 2) + "\n");
  return 0;
}

main(process.argv.slice(2))
  .then((code) => { process.exitCode = code; })
  .catch((error) => {
    process.stderr.write(JSON.stringify({ ok: false, error: error.message }) + "\n");
    process.exitCode = 1;
  });
~~~

- [ ] **Step 6: Run end-to-end tests and inspect the report**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm test
npm run assess:fixture
find .tmp/assessment -type f -print
~~~

Expected: all tests pass; the smoke output contains separate current candidates and prior-approved contracts, the policy and bundle, an OpenClaw non-authoritative conformance suggestion plus unsupported policy-enforcement result, Docker unsupported/no-proposal output, drift/evaluation/pilot/assurance reports, structured explanations, and evidence-manifest.yaml with verified semantic and exact-file digests for every emitted artifact. The first-run test emits candidates but no eligible capability or conformance suggestion.

- [ ] **Step 7: Commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/src/explain control-plane/src/eval/pilot-gate.mjs control-plane/src/drift/aggregate-observations.mjs control-plane/src/evidence/verify-artifacts.mjs control-plane/src/assessment.mjs control-plane/src/cli.mjs control-plane/tests/assessment.test.mjs control-plane/tests/pilot-gate.test.mjs control-plane/tests/observation-aggregation.test.mjs control-plane/tests/verify-artifacts.test.mjs control-plane/tests/build-smoke-fixture.mjs control-plane/tests/fixtures/smoke control-plane/tests/fixtures/first-run
git commit -m "feat: assemble read-only capability assessment"
~~~

---

### Task 12: Add public boundary documentation, CI, and design-partner gates

**Files:**
- Create: control-plane/README.md
- Create: docs/research/control-plane-design-partner-protocol.md
- Create: .github/workflows/control-plane-ci.yml
- Modify: README.md
- Modify: README.zh-CN.md

**Interfaces:**
- Produces: public commands, non-goals, evidence requirements, market thresholds, and falsification decision.
- CI runs with no secrets, no network target, and no runtime proxy.

- [ ] **Step 1: Add the public MVP README**

Create control-plane/README.md:

~~~markdown
# Capability Control Plane

Local read-only architecture-to-policy compiler and assurance prototype.

## What it does

- ingests sanitized versioned source snapshots;
- validates the exact Part I candidate schema, preserves its structured payload and conversion gaps, and emits only Part II mapping/workflow candidates for independent review and promotion—never directly to the approved store or Policy IR;
- requires explicit business-capability mappings and a separate prior-approved contract baseline;
- validates Capability and Workflow Contracts, approval records, evidence freshness, and versioned equivalence proofs;
- compiles deterministic Policy IR and minimum bundles;
- emits a non-authoritative OpenClaw conformance suggestion separately from an `unsupported` policy-enforcement result, plus Docker unsupported/no-proposal analysis;
- aggregates same-implementation observations by explicit source precedence, retains every source reference, and quarantines source conflicts or protected drift before bundle compilation;
- simulates fail-closed decisions, single-use approvals, retry, and fallback;
- explains each inclusion and exclusion with structured policy, contract, evidence, drift, evaluation, approval, fallback, and adapter facts.

## What it does not do

It does not proxy MCP traffic, handle credentials, run an OAuth broker, host servers, modify IAM, apply target configuration, orchestrate business workflows, rank tools before policy eligibility, or execute high-risk writes.

## Run

    npm ci --ignore-scripts
    npm test
    node src/cli.mjs assess tests/fixtures/smoke .tmp/assessment

OpenClaw output contains a reviewable, non-authoritative conformance suggestion, but its separate policy-enforcement result is `unsupported`, `preserves_ir: false`, and has no proposal or actionable diff. Docker output is also `unsupported` with `proposal: null`, empty changes, and no actionable diff because these MVP targets cannot preserve mandatory request-time semantics.

The checked-in smoke fixture is intentionally small and proves deterministic execution only. It always produces `pilot_scorecard.status: not_evaluated`. A pilot pass digest-binds the prior-approved contracts, eligible bundle, drift results, and target artifacts; requires a non-empty approved set, one current eligible read, host/OpenClaw and gateway/Docker results, and no critical-contract quarantine; and uses real sanitized official-registry, private-registry, and live-schema evidence across 3–5 servers and 10–30 capabilities. Selection and poisoning need independently identified, digest-bound actual results. High-risk approval, retry, and fallback must all pass with zero false permits while binding the same approved high/critical write or destructive contract digest and action class. The evidence manifest records the four authority digests plus semantic and exact-file SHA-256 values for every referenced report artifact.
~~~

- [ ] **Step 2: Add the exact design-partner protocol**

Create docs/research/control-plane-design-partner-protocol.md:

~~~markdown
# Capability Control Plane Design-Partner Protocol

## Interview gate

Interview exactly twelve target organizations. Record whether each has:

- at least two agent, host, or gateway surfaces;
- at least ten business capabilities;
- a cross-team discovery, permission, drift, audit, or fallback problem;
- an accountable capability-contract owner;
- willingness to provide sanitized schemas, configuration, and traces.

Continue only if at least three organizations satisfy all five conditions.

## Evidence pilot

Within sixty days, obtain sanitized real evidence from at least two organizations. For each pilot measure:

- run an assessment-inputs/v1alpha1 manifest with `mode: pilot`; never relabel smoke or synthetic unit data;
- use independently provenanced official-registry, private-registry, and live-schema current sources covering 3–5 servers and 10–30 mapped business capabilities; aggregate corroborating observations and quarantine any protected-projection conflict;
- digest-bind a non-empty prior-approved contract set, the eligible bundle, drift results, and host/OpenClaw plus gateway/Docker target artifacts; require at least one current eligible read and zero quarantined critical contracts;
- cover at least two governance surfaces and import sanitized traces whose suite ID, case ID, runner, actual decision, and result digest are verified before evaluation;
- execute distinct, independently identified suites for tool selection, prompt poisoning, tool poisoning, and offline high-risk approval, retry/idempotency, and fallback denial; the three high-risk suites must bind the same approved high/critical write or destructive `contract_digest` and `action_class`; derive pass/fail from expected versus actual results rather than scenario labels;
- require `pilot-scorecard.yaml` to report `status: pass`; any missing field or suite, reused cross-scenario suite, failed actual result, stale evidence, or critical false permit makes the gate fail;

- manual inventory/audit time before and after;
- exposed tool/schema count before and after;
- selection accuracy before and after;
- ordinary allow/deny conformance;
- destructive false permits;
- time from source ingestion to affected-contract drift report;
- previously unknown policy or drift gaps;
- willingness to continue.

## Required thresholds

- at least 30 percent reduction in manual inventory/audit effort, or one material previously unknown gap;
- at least 40 percent reduction in exposed tools or schema-token volume without reduced selection accuracy;
- at least 95 percent ordinary allow/deny conformance;
- zero critical/destructive false permits;
- material drift associated with affected contracts within fifteen minutes;
- at least two of three design partners continue into a paid or committed pilot.

## Stop conditions

Stop or narrow if one existing platform fully solves the problem, customers refuse contract ownership, adapters require silent policy weakening, capability modeling routinely exceeds two hours, two adapters require incompatible IR semantics, traces cannot establish outcomes, or design partners will not provide sanitized evidence.

## Repository and release gate

Do not publish a release until all of the following are true:

- the rights owner has committed the intended repository license text;
- repository administrators have supplied real CODEOWNERS principals for contracts, schemas, adapters, workflows, compatibility evidence, and release files;
- the default-branch ruleset requires the two CI workflows, stale-approval dismissal, review after the latest push, and Code Owner review;
- the release commit has a dependency inventory with resolved source, integrity, and license for every transitive package;
- the release artifact has a content digest, SBOM, provenance attestation, and immutable tag or release;
- a sanitized design-partner scorecard proves the required 3–5 servers and 10–30 capabilities rather than citing the smoke fixture.
- the technical `pilot-scorecard.yaml` passes from real sanitized partner inputs, binds the approved set/bundle/drift/target authority digests, proves a current eligible read and no critical quarantine, includes official/private/live sources and two authoritative target surfaces, and records independent-suite actual results with one shared approved high-risk write/destructive contract binding and zero critical false permits.
~~~

- [ ] **Step 3: Add deterministic CI pinned to immutable action commits**

Create .github/workflows/control-plane-ci.yml:

~~~yaml
name: Capability control plane

on:
  pull_request:
    paths:
      - "control-plane/**"
      - "docs/research/control-plane-design-partner-protocol.md"
      - "docs/rfcs/0002-architecture-driven-capability-control-plane.md"
      - "README.md"
      - "README.zh-CN.md"
      - "LICENSE"
      - ".github/workflows/control-plane-ci.yml"
  push:
    branches: [main]
    paths:
      - "control-plane/**"
      - "docs/research/control-plane-design-partner-protocol.md"
      - "docs/rfcs/0002-architecture-driven-capability-control-plane.md"
      - "README.md"
      - "README.zh-CN.md"
      - "LICENSE"
      - ".github/workflows/control-plane-ci.yml"

permissions:
  contents: read

jobs:
  deterministic-assessment:
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
        with:
          persist-credentials: false
      - uses: actions/setup-node@1e60f620b9541d74bece96c5465dc8ee9832be0b
        with:
          node-version: "22.23.1"
          cache: npm
          cache-dependency-path: control-plane/package-lock.json
      - run: npm ci --ignore-scripts
        working-directory: control-plane
      - run: npm test
        working-directory: control-plane
      - run: npm run assess:fixture
        working-directory: control-plane
~~~

- [ ] **Step 4: Update the repository entries**

Add this status paragraph to README.md:

~~~markdown
The Part II reference implementation is a local read-only compiler. It keeps current candidates separate from an independently reviewed prior-approved baseline, then emits reviewable policy, bundles, advisory or unsupported target analysis, drift, and evaluation reports. It does not proxy traffic, handle credentials, or enforce runtime calls.
~~~

Add this status paragraph to README.zh-CN.md:

~~~markdown
Part II 参考实现是本地只读编译器：当前观测生成的候选契约与独立、预先评审的批准基线严格分离，输出为可评审的策略、最小 bundle、建议性或不支持的目标分析、漂移与评测报告；它不代理流量、不接触凭证，也不执行运行时授权。
~~~

- [ ] **Step 5: Run the complete technical gate**

Run:

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT/control-plane"
npm ci --ignore-scripts
npm test
npm run assess:fixture
cd ..
git diff --check
git status --short
~~~

Expected: all tests pass, the fixture assessment succeeds, diff check emits no output, and only intended files are modified.

- [ ] **Step 6: Commit**

~~~bash
REPOSITORY_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPOSITORY_ROOT"
git add control-plane/README.md docs/research/control-plane-design-partner-protocol.md .github/workflows/control-plane-ci.yml README.md README.zh-CN.md
git commit -m "docs: define capability control plane pilot gates"
~~~

- [ ] **Step 7: Run the market gate before any runtime-sidecar RFC**

Execute the twelve interviews and up to three evidence pilots exactly as documented. Record results in source-controlled, sanitized scorecards. If thresholds fail, publish the falsification result and stop expansion. Do not propose a runtime policy sidecar merely because the technical fixture passes.

---

## Final Acceptance Checklist

- [ ] The package runs locally with no hosted account.
- [ ] npm dependencies are exactly pinned and install scripts are disabled.
- [ ] Source, input-manifest, capability, approved-baseline, evidence, equivalence, approval, trace, target, workflow, policy, bundle, pilot-scorecard, and assessment schemas validate.
- [ ] Every wire protocol enum uses exactly `2025-11-25` or `2026-07-28`; release-candidate maturity is documentation metadata and never a `-rc` wire-version suffix.
- [ ] Canonical hashes are stable across object key ordering.
- [ ] The sanitized-input boundary canonicalizes the root, rejects every final or ancestor symlink component, verifies final realpath containment, and rejects traversal, malformed UTF-8, oversized documents, and representative credential indicators before parsing or persistence; documentation states that secret scanning is best-effort.
- [ ] Tool-to-capability mapping requires explicit organization input.
- [ ] Current discovery creates candidates only; Part I Architecture Pack imports validate the exact Part I candidate schema, preserve the structured payload and explicit vocabulary-conversion gaps, emit independently reviewable mapping/workflow candidates, and have no authorization effect.
- [ ] A first-run assessment with an empty prior-approved baseline emits candidates but zero eligible capabilities and no target conformance suggestion or applicable proposal.
- [ ] Prior-approved contracts and approval records are separate inputs, predate current observations, and bind the exact contract and approved implementation digests.
- [ ] Contract integrity checks projection/schema/auth/semantic/approval/equivalence cross-fields before a contract enters the approved store.
- [ ] A business capability may retain multiple exact implementations while deterministic bundle selection requires one reviewed primary.
- [ ] Source annotations remain separate from verified properties.
- [ ] Unknown identity, tenant, scope, audience, schema, evidence freshness, drift status, or equivalence fails closed.
- [ ] Current source and reviewed evidence observation/expiry fields are enforced, and stale or subject-mismatched evidence cannot enter a bundle.
- [ ] Same-implementation observations use explicit source precedence, retain all source evidence/provenance references, and quarantine conflicting protected projections; protected drift compares the selected current observation with the independent approved projection before bundle compilation.
- [ ] Minimum bundle eligibility is deterministic; simulation re-derives the complete envelope and included item so field tampering, including an approval flag flip, requires revalidation.
- [ ] Structured explanations name the policy rule, contract/reviewer/digests, evidence/freshness, evaluation, drift, approval binding, equivalence/fallback, and target coverage.
- [ ] OpenClaw separates a non-authoritative conformance suggestion from an `unsupported`, `preserves_ir: false` policy-enforcement result; the suggestion is never treated as authorization or IR-preservation evidence.
- [ ] Docker always returns `status: unsupported`, `proposal: null`, `changes: []`, and `actionable_diff: false`; purpose, data-flow/redaction, argument binding, writes, approvals, and overexposure are reported as unsupported reasons.
- [ ] Description, title, schemas, auth, endpoint class/origin, owner, execution metadata, annotations, protocol, server/tool identity, and provenance drift can quarantine a contract.
- [ ] Offline write approvals bind argument, policy, schema, contract, approved implementation, optional equivalence proof, tenant, purpose, nonce, expiry, and approval digest; tenant-scoped replay is denied.
- [ ] Private-read plus external-send composition risk is detected.
- [ ] Fallback requires a current reviewed versioned proof over identity, data boundary, side effects, schemas, approval, reversibility, idempotency, and success semantics; write fallback remains denied by default.
- [ ] Retry is transient-only; write retry additionally requires verified idempotency, key propagation, adapter support, unchanged binding, and attempts remaining.
- [ ] The smoke output contains every RFC 0002 report artifact but its pilot scorecard is always `not_evaluated`.
- [ ] The evidence manifest binds every input, repeats the pilot scorecard digests for approved contracts, eligible bundle, drift results, and target artifacts, and records both a semantic digest and exact persisted-file SHA-256 for every referenced report artifact; the verifier rereads each file before manifest completion.
- [ ] No code proxies MCP traffic, handles tokens, or applies target configuration.
- [ ] CI, npm test, fixture assessment, and git diff --check pass.
- [ ] A technical pilot pass digest-binds non-empty prior-approved contracts, the eligible bundle, drift results, and two authoritative target-surface artifacts; requires one current eligible read and no critical quarantine; and uses real sanitized official/private/live sources, 3–5 servers, 10–30 capabilities, and verified suite/case/result digests. High-risk approval/retry/fallback must all pass with zero false permits against the same approved high/critical write or destructive contract digest and action class; first-run, all-quarantined, and read-labeled high-risk cases fail.
- [ ] Market and technical falsification thresholds are measured before runtime expansion.
- [ ] Before any public release, license text, real CODEOWNERS principals, protected-branch rules, dependency inventory, SBOM, provenance attestation, and immutable release controls are present.
- [ ] Design-partner claims use 3–5 servers and 10–30 capabilities; the one-server smoke fixture is never cited as pilot evidence.
