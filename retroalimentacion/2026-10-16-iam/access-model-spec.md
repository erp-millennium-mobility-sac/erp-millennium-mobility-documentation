# Access model, step 3 — implementation contract

Source of truth: data dictionary v1.17 §5.2–§5.9, §5.12, §5.13, D-32, D-38, D-41, D-42, D-43; context map §6.2 and §11 (decisions 2 and 4); API conventions §2.2, §3.2, §4.2, §6.1, §7, §9.2, §13 row 16, §14.10, §14.15, §14.17. Where this contract and the dictionary differ, the dictionary wins, except for the decisions marked **[DECISION]**, which the product owner approved on 15/10/2026 and will be recorded in the dictionary (v1.18), the context map and the conventions.

We are in development: no compatibility shims, no feature flags, no dead endpoints kept "for later". Remove what is replaced.

## 0. Root causes this fixes (evidence from the audit of 14/10/2026)

- A. "Grant access" mixed find-or-create account + link + **replace roles** (`provisionEmployeeUser` → `User.syncRoles`). Account takeover in two steps: link a foreign account by email, then reset its password through the employee.
- B. `user_roles` has no scope (dictionary §5.13); `user_roles_active_uq (user_id, role_id)` even forbids the same role in two companies (contradicts §5.7, Tere example).
- C. Authorization derived from "the caller's employee" (`OrganizationSecurityService` → `findByUserId`, single result) instead of from role assignments (§5.7: the assignment says on what a person acts; the employee is only a precondition to grant it).
- D. Frontend built ahead of the backend contract (`PUT /users/{id}/roles`, `/employees/{id}/suspensions` do not exist).

## 1. Context boundaries **[DECISION D1]**

- **`iam` depends on `organization`; `organization` never depends on `iam`** (map §4/§6.2 proposal, decision 4 resolved). `iam/package-info.java`: `allowedDependencies = {"organization::acl"}`. `organization/package-info.java`: `allowedDependencies = {"location::acl"}`. Delete `organization/.../outboundservices/acl/ExternalIamService` and every use of `IamContextFacade` from organization; delete `IamContextFacade` members nobody uses afterwards (or the whole facade if empty).
- **Termination and reactivation are orchestrated by `iam`** (map decision 2, option A): `iam` validates (last superadmin, permissions), asks `organization` to record the fact, then revokes roles / locks the account in the same transaction, returning `Result` (no exceptions for expected failures, rule 3.1). Routes keep their paths (`POST /employees/{id}/termination`, `POST /employees/{id}/reactivation`); the controllers live in `iam` (routes carry no context prefix).
- **Grant access lives in `iam`** (map §6.2): `POST /access-grants`.
- **How `organization` knows the caller's scope without depending on `iam`:** a port in the shared kernel, implemented by `iam` — same pattern as `CurrentUserAuditorAware` (iam implements Spring's `AuditorAware`, every context uses it without a code dependency on iam).
  - `shared` (application layer, e.g. `ky.millennium.mobility.erp.shared.application.security`): `interface CurrentAccessScope { AccessScope current(); }` and `record AccessScope(Long userId, boolean global, Set<Long> companyIds, Set<Long> administeredCompanyIds)`:
    - `global`: the account holds an active SYSTEM-scope role (SUPERADMIN).
    - `companyIds`: companies where the account holds any active COMPANY- or BRANCH-scope assignment.
    - `administeredCompanyIds`: companies where it holds an active `ADMIN_EMPRESA` assignment.
    - An unauthenticated or suspended account gets an empty scope (never throws).
  - `iam` implements it (infrastructure), reading the current user's active, non-expired assignments.

## 2. Data model (Flyway; next free number is V26; follow V15/V20 naming and header-comment style; grant new tables to `${app_role}` like V19)

**V26 `organization` ownership keys**
- `branches`: `UNIQUE (id, company_id)` (`branches_id_company_uq`).
- `commercial_brands`: `UNIQUE (id, company_id)` (`commercial_brands_id_company_uq`).
- `employees`: `UNIQUE (company_id, user_id)` (a user links to at most one employee per company, §5.3); FK `(branch_id, company_id) → branches (id, company_id)` (the branch of an employee is of its company).
- `locations`: `UNIQUE (address_id)` (an address is never shared, §4.6, LOC-R29); FK `(branch_id, company_id) → branches (id, company_id)`.

**V27 `iam` scoped role assignments** (dictionary §5.7)
- `user_roles`: add `company_id BIGINT NULL REFERENCES companies`, `branch_id BIGINT NULL`, `commercial_brand_id BIGINT NULL`.
- CHECK scope coherence: `SYSTEM` → company, branch, brand all NULL; `COMPANY` → company NOT NULL, branch NULL; `BRANCH` → company and branch NOT NULL. A global (SYSTEM) role cannot have a brand (implied).
- FKs `(branch_id, company_id) → branches (id, company_id)`, `(commercial_brand_id, company_id) → commercial_brands (id, company_id)`.
- Replace `user_roles_active_uq` with `UNIQUE NULLS NOT DISTINCT (user_id, role_id, company_id, branch_id, commercial_brand_id) WHERE revoked_at IS NULL` (as a unique index with `NULLS NOT DISTINCT`), so the same role can be held on two companies but never twice on the same scope.
- `revocation_reason` CHECK: `retiro`, `caducidad`, `baja_colaborador`, `baja_cuenta` (D-41), keeping the existing "revoked ⇔ reason" check. Before adding it, `UPDATE … SET revocation_reason = 'retiro' WHERE revocation_reason NOT IN (…)` (dev data only has none, but the migration must be safe).
- Existing COMPANY/BRANCH assignments without scope: dev databases have none (verified 15/10/2026: only the bootstrap SUPERADMIN). The migration fails loudly (`RAISE EXCEPTION`) if any active COMPANY/BRANCH assignment lacks a company, instead of guessing.

**V28 `iam` access suspensions** (dictionary §5.4)
- `user_access_suspensions`: `id BIGSERIAL PK`, `public_id UUID NOT NULL DEFAULT uuidv7() UNIQUE`, `user_id BIGINT NOT NULL REFERENCES users`, `period TSTZRANGE NOT NULL` (lower bound required, `[start, end)`, open end = still suspended), `reason VARCHAR(20) NOT NULL CHECK IN ('vacaciones','licencia','baja','seguridad')`, `notes TEXT NULL`, `created_by BIGINT NOT NULL REFERENCES users`, `created_at TIMESTAMPTZ NOT NULL`, `closed_by BIGINT NULL REFERENCES users`, `closed_at TIMESTAMPTZ NULL`.
- No two periods of a user overlap: `CONSTRAINT … UNIQUE (user_id, period WITHOUT OVERLAPS)` (PostgreSQL 18 temporal constraint; needs `CREATE EXTENSION IF NOT EXISTS btree_gist`, a trusted extension). If the temporal UNIQUE is not usable, an equivalent `EXCLUDE USING gist (user_id WITH =, period WITH &&)`.
- CHECK `closed_at IS NULL OR closed_by IS NOT NULL` and closing sets the period's upper bound.
- A user is active iff no period contains `now()` (dictionary: no stored flag).

## 3. IAM domain and rules

- `RoleAssignment` gets its scope (`companyId`, `branchId`, `commercialBrandId`, internal ids) and `expiresAt`. `RevocationReason` enum with the four D-41 values (wire values in Spanish, lower case).
- `User` operations: `grant(role, scope, grantedBy, now, expiresAt)` (scope must match `role.scopeLevel`; an active duplicate on the same scope is rejected), `revoke(assignment, reason, now)`, `revokeActiveInCompany(companyId, reason, now)` (COMPANY and BRANCH assignments of that company), `revokeAllActive(reason, now)`. **Delete `syncRoles`.**
- `AccessSuspension` aggregate (own repository): open (start may be future), close (sets end and closer). Opening one with reason `baja` revokes every active assignment with `baja_cuenta` in the same transaction (§5.9).
- **SYSTEM account** (`system@millennium.ky`): cannot sign in (rejected by identity, same 401 as bad credentials), cannot be linked to an employee, cannot receive roles, cannot be suspended or have its password reset.
- **Last active SUPERADMIN** (§5.9): no operation may leave zero active, non-suspended SUPERADMIN assignments — revoke, suspension, termination. One guard in iam used by all of them.
- **Grant policy [DECISION D3]** (interim until `permissions`/`role_permissions` exist; replaces the "hold the same role" rule of §5.13, which contradicts §5.2 step 8 where the company admin grants roles it does not hold):
  - `SUPERADMIN` (global) may grant any role on any scope.
  - `ADMIN_EMPRESA` on company C may grant COMPANY- and BRANCH-scope roles **other than `ADMIN_EMPRESA`**, only with scope inside C (branch and brand of C). `SUPERADMIN` and `ADMIN_EMPRESA` are granted only by a SUPERADMIN (§5.2 step 7).
  - Nobody else grants.
  - Revoking an assignment requires being able to grant it.
  - The grantor must be linked to an active employee (§5.2: the first superadmin cannot grant until linked).
  - The target must have an active employee in the scope's company for COMPANY/BRANCH roles, or in some company for SYSTEM roles (§5.3, §5.7).
  - Role codes are resolved by `roleId` (public UUID) against the catalog; unknown → 404 `ROLE_NOT_FOUND`.
- **Who administers an account** (password reset, suspensions; and the account side of termination): a SUPERADMIN; or an `ADMIN_EMPRESA` of **every** company where the account has an active employee, provided the account holds no active role outside those companies and no SYSTEM-scope role. Nobody suspends or resets their own account through these admin operations (own password change already exists).

## 4. Grant access **[DECISION D2]** — `POST /api/v1/access-grants`

Body `{ "employeeId": UUID, "email": string }`. Transactional; links through the organization facade.
- Employee must exist and be active; if it already has an account → 409 `EMPLOYEE_ACCOUNT_CONFLICT`.
- Caller: SUPERADMIN, or ADMIN_EMPRESA of the employee's company; caller must be linked to an active employee — **except self-link**: an account without employee may link *itself* (email = own email) to an employee (§5.2 step 6, first superadmin).
- Email normalized (trim, lower case).
- Email not registered → create the account (system-generated temporary password, must change; `created_by` = caller) → **201** `{ userId, email, created: true, temporaryPassword }` (shown once).
- Email registered:
  - SYSTEM → 422 `BUSINESS_RULE_VIOLATION`.
  - already linked to another employee of the same company → 409 `EMPLOYEE_ACCOUNT_CONFLICT` (and the V26 unique).
  - linked to an active employee of another company, or holding a SYSTEM-scope role → only a SUPERADMIN may link it (**[DECISION D3]**: one company per country (D-5) and per-country tax ID types (D-44) make a document match between companies impossible, so cross-company identity is confirmed by the global administrator); otherwise 403 `ACCESS_DENIED`.
  - otherwise (no links): only self-link or SUPERADMIN.
  - → **200** `{ userId, email, created: false, temporaryPassword: null }`. **Never changes roles or password.**
- Roles are granted afterwards with role assignments.

## 5. IAM REST (convention routes §2.2)

- `GET /api/v1/users/{id}/role-assignments` — active and past assignments (history), newest first.
- `POST /api/v1/users/{id}/role-assignments` — body `{ roleId, companyId?, branchId?, commercialBrandId?, expiresAt? }` → 201 the assignment. Errors: 403 policy, 404 role/company/branch/brand, 409 `ROLE_ASSIGNMENT_CONFLICT` (active duplicate), 422 scope incoherent / target without active employee.
- `POST /api/v1/role-assignments/{id}/revocation` — reason `retiro` → 200. 422 last superadmin.
- `RoleAssignmentResource`: `id, roleId, roleCode, roleName, scopeLevel, companyId, companyName, branchId, branchName, commercialBrandId, commercialBrandName, grantedAt, expiresAt, revokedAt, revocationReason` (names resolved in batch through the organization facade).
- `GET /api/v1/users/{id}/suspensions`; `POST /api/v1/users/{id}/suspensions` — body `{ startsAt?, endsAt?, reason, notes? }` (startsAt default now; future allowed; endsAt optional) → 201; `POST /api/v1/suspensions/{id}/closure` — body `{ closedAt? }` → 200. Overlap → 409 `ACCESS_SUSPENSION_CONFLICT`. A suspension that starts now closes all sessions of the user.
- `POST /api/v1/users/{id}/password-reset` → 200 `{ temporaryPassword }` once; must change; closes all sessions of the user. **[DECISION]** (not in the dictionary today; recorded in v1.18).
- `GET /api/v1/users` — SUPERADMIN: all; ADMIN_EMPRESA: only accounts with an active employee in one of its administered companies. Filter `companyId` (required when the caller administers more than one company, conventions §14.10): each row then carries `employeeId` (the account's employee in that company) and the roles active **in that company**. `UserResource`: `id, email, mustChangePassword, lastLoginAt, isSuspended, activeSuspensionId, employeeId?, roles[]` (roles as small objects `{assignmentId, roleCode, roleName, scopeLevel, branchId?, branchName?, commercialBrandId?, commercialBrandName?}`). Paged per §6.2 with `page`; full array without it; export unchanged in spirit (no ids, D-51).
- **Delete `POST /api/v1/users`** (account without employee, D-38; conventions §13 row 16).
- `GET /api/v1/authentication/me` (and the sign-in response): `{ id, email, mustChangePassword, roles: [codes active anywhere], global: boolean, companies: [{ id, name }] }` — `companies`: all active companies when global; otherwise the companies of `CurrentAccessScope.companyIds`.
- Sign-in of a suspended account → 403 `ACCOUNT_SUSPENDED`; `SessionUserRefreshFilter` also treats a currently suspended account as signed out (so future-start suspensions take effect by themselves).
- Termination `POST /api/v1/employees/{id}/termination` (iam controller) — body `{ terminationDate, terminationReason? }`:
  - caller: SUPERADMIN, or ADMIN_EMPRESA of the employee's company who administers the account (if any) per §3.
  - `terminationDate` not after "today" of the employee (branch time zone, else the company country's, else `America/Lima`, D-40) — **[DECISION]**: the exit is recorded when it happens; a future date would strip roles before the person leaves.
  - last superadmin guard; organization records the exit (Result); revoke the account's active roles **in that company** with `baja_colaborador`; if the account has no other active employee → open a `baja` suspension now, revoke remaining roles with `baja_cuenta`, close sessions (§5.3, §5.9). → 200 the employee resource from organization (or a minimal termination resource).
- Reactivation `POST /api/v1/employees/{id}/reactivation` (iam controller) — organization reactivates; if the account has an open `baja` suspension, close it (dictionary §5.4 "Si la persona regresa, se cierra ese período"). Roles are not restored (§5.9).

## 6. Organization

- New facade members (organization::acl) for iam, returning acl records only: employee access summary by public id (`id, publicId, companyId, branchId, active, userId`), employees of a user, active-employee company ids of a user, `linkUser(employeeId, userId)` (Result), `terminate(employeePublicId, date, reason)` (Result, validates dates and "today"), `reactivate(employeePublicId, hireDate)` (Result), company/branch/brand public id → internal id with ownership checks (branch/brand belongs to company), names in batch, all active companies (for `/me`), employee ids by user ids within a company.
- Remove from organization: `/employees/{id}/access`, `/employees/{id}/password-reset`, the termination/reactivation controllers (moved to iam), `ExternalIamService`, account fields of `EmployeeResource` (keep `hasAccount: boolean` from `employees.user_id`). Metrics `withAccess` stays (counted from `user_id`).
- **Scope enforcement with `CurrentAccessScope`** (conventions §9.2): every read of companies, branches, brands, locations and employees — list, export and by id, nested routes included — returns only what the scope allows (global → all). A company outside the scope, or a resource whose company is outside it, answers **404** (`<RESOURCE>_NOT_FOUND`), not 403. `OrganizationSecurityService` is rewritten on the port (no `findByUserId`).
- Writes: companies, brands, branches, locations stay SUPERADMIN-only (D-43/§5.2). Employees (create, update profile): SUPERADMIN, or ADMIN_EMPRESA of the employee's company (`administeredCompanyIds`).

## 7. Tests that must exist (regression of every finding)

1. Account takeover: ADMIN_EMPRESA grants access with a SUPERADMIN's email → 403; with SYSTEM's email → 422; nothing changes in the target account (roles, password).
2. Access grant never changes roles or password of an existing account.
3. Role code with spaces / unknown role → 404; ADMIN_EMPRESA granting SUPERADMIN or ADMIN_EMPRESA → 403; granting outside its company → 403.
4. Last superadmin: revoke, suspend, terminate → 422.
5. Two companies: the same role on two companies coexists; terminating in one keeps the other's roles; terminating in the last one locks the account (`baja`) and revokes the rest (`baja_cuenta`).
6. An account with employees in two companies can use the employee screens of both (no single-result lookups).
7. Cross-company reads → 404 for companies, branches, brands, locations, employees (list, export, by id).
8. `/me` lists the companies of the scope; global lists all active companies.
9. Suspension: overlapping → 409; suspended account cannot sign in (403 `ACCOUNT_SUSPENDED`) and its open sessions stop working; future-start suspension takes effect by itself.
10. First superadmin without employee cannot grant until self-linked.
11. Modularity: `ModularityTests` green with iam → organization only.

## 8. Out of scope of this iteration (explicit, not forgotten)

- Row Level Security (implementation order: defense in depth after the application-level filter, needs `erp_mm_app` per environment and `set_config` infrastructure).
- Public sign-up with profile (D-32): needs conventions §14.16 decided; sign-up stays off by default.
- Rehire history (reactivation erases the previous period): needs a dictionary design.
- Permission matrix (`permissions`, `role_permissions`): pending data.
- `employees.document_type_id` same-country key: needs `employees.country_id`; separate change.
