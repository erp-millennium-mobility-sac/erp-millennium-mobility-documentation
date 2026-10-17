# Access model, round 2: what changes after the first build (16/10/2026)

This addendum amends `access-model-spec.md` (same folder). Where they differ, this one wins. It comes from the two
build reports of round 1 and from the precedence rule of the project: dictionary > own skills > code.

Repos and branches: backend `mm-acc-be` and frontend `mm-acc-fe`, both on `feature/access-model`.

## R1. Row Level Security is in scope (overrides §8 of the contract)

**Why.** The backend skill forbids company/branch roles without RLS:
- rule 5.7 of `backend-module/references/architecture-rules.md`, together with rules 5.1–5.3;
- `SKILL.md` §4, "La frontera entre empresas la defiende la base", and its checklist.

Round 1 already granted COMPANY and BRANCH roles, so the database layer must exist before this merges.

**What it needs.**
- **Accounts:**
  - The local cluster now has `erp_mm_owner` and `erp_mm_app`, created by `scripts/database/create-roles.sql`, with local passwords `erp_mm_owner_dev` and `erp_mm_app_dev`.
  - `erp_mm_app` owns nothing, cannot bypass RLS and receives the grants of V19.
  - Tests must connect as `erp_mm_app` (rule 5.3: the application never uses the owner) and migrate as the owner.
- **New migration** (next free number after V28; follow the header-comment style of V15/V19/V20):
  - `ENABLE` and `FORCE ROW LEVEL SECURITY` on every table with `company_id`: `branches`, `commercial_brands`, `employees`, `locations` and `user_roles`, plus `companies` on its own `id`. Check whether any other table has `company_id`.
  - Policies for SELECT, INSERT and UPDATE; the app account has no DELETE on business tables.
  - The policies read transaction-local settings, for example `app.access_scope` ('SYSTEM' or 'SCOPED'), `app.company_ids` and `app.user_id`, with `current_setting(name, true)`. **A missing or empty setting sees nothing** (fail closed).
  - `user_roles` also lets an account see its own rows (`user_id` = `app.user_id`) and the rows without a company, so the scope can be computed from them.
  - The migration re-applies, idempotently, the grants and default privileges of V19 for `${app_role}`. Databases migrated before the role existed (V19 granted nothing then) get them that way.
- **Central `set_config`:**
  - One central point (rule 5.2), inside every transaction, with `set_config(…, true)`. Never at connection open.
  - The identity comes from `CurrentAccessScope`: global → SYSTEM; otherwise the company ids of the scope.
  - SYSTEM explicitly for the bootstrap/startup handlers and any scheduled or automatic process.
  - No session → nothing visible. Sign-in reads only tables without `company_id`, plus the account's own `user_roles` rows through `app.user_id`. Decide how sign-in sets `app.user_id` once the password matched.
  - Choose and justify the mechanism, for example a `JpaTransactionManager` subclass or a transaction-begin hook that runs on the transaction's connection. `JdbcTemplate` inside the same transaction must see the same settings.
- **Elevation for cross-company checks:**
  - These checks must see beyond the caller's companies, through one narrow, explicit elevation to SYSTEM: account administration ("ADMIN_EMPRESA of every company of the account"), the cross-company link rule of access grants, "has the account another active employee" on termination and reactivation, and the last-superadmin count.
  - Use it only inside the check and restore the caller's scope afterwards.
  - List every elevation point in the report, and test that an ADMIN_EMPRESA of A **cannot** reset the password of, suspend, or link an account that also has an employee or role in B (403), while connected as `erp_mm_app`.
- **Tests:**
  - Connected as `erp_mm_app`, the RLS tests show:
    - without settings, zero rows;
    - with scope A, only A's rows, and inserting a B row fails;
    - with SYSTEM, all rows.
  - The application, booted as `erp_mm_app` with an ADMIN_EMPRESA of A, gets 404 for B and does not list it.
  - **The whole suite must pass connected as `erp_mm_app`**, which is the production topology: any path that forgets the central setting must fail in tests, not in production.
- **Verify DB:** recreate `erp_mm_db_verify` with `OWNER erp_mm_owner`; `btree_gist` is a trusted extension. Never touch `erp_mm_db`, which is the user's.
- **Test command:** `DATABASE_NAME=erp_mm_db_verify DATABASE_USER=erp_mm_app DATABASE_PASSWORD=erp_mm_app_dev FLYWAY_USER=erp_mm_owner FLYWAY_PASSWORD=erp_mm_owner_dev ./mvnw -o test`. Document in the backend README how to run the tests with the two accounts.
- **Session cookie:** round 1 saw the cookie named `SESSION`, not `MM_SESSION`, in MockMvc. Find whether the configured name applies at runtime with Spring Session JDBC. Fix the configuration only if it really does not.

## R2. Suspensions: cancelling a scheduled one, and the exit never collides

**Cancelling:**
- `POST /suspensions/{id}/closure` on a period that **has not started** cancels it. Its period stays as scheduled, `closed_at` and `closed_by` are recorded, and it stops counting for overlaps.
- On a period **in force**, closure ends it at `closedAt` (default now), as today.
- On a period that ended or was already closed or cancelled → 422 `ACCESS_SUSPENSION_CLOSED`.
- `ACCESS_SUSPENSION_NOT_STARTED` disappears.

**Database (edit V28 in place):** it is not merged anywhere, and the verify DB is being recreated anyway.
- Replace `UNIQUE (user_id, period WITHOUT OVERLAPS)` with `EXCLUDE USING gist (user_id WITH =, period WITH &&) WHERE (<not cancelled>)`.
- "Cancelled" means `closed_at IS NOT NULL AND closed_at <= lower(period)`.
- Add whatever CHECK keeps `closed_at` consistent with the period.

**Resource:**
- Replace `inForce` with `status`: `SCHEDULED | IN_FORCE | ENDED | CANCELLED`, computed at read time.
- Keep `id, startsAt, endsAt, reason, notes, createdBy, createdAt, closedBy, closedAt`.

**Exit that closes the account:**
- It already ends the suspension in force now and opens `baja` from now. It must also **cancel the suspensions scheduled ahead**, recorded as closed by the requester, instead of answering 409.
- A manual `POST /users/{id}/suspensions` that overlaps still answers 409 `ACCESS_SUSPENSION_CONFLICT`.

## R3. Contract additions and fixes

- **`GET /api/v1/authentication/me` and sign-in:** `companies: [{id, name, timeZone, administered}]`.
  - `timeZone` is the IANA zone of the company's country (`countries.default_timezone`, through the organization facade).
  - `administered` is whether the account administers that company (in `administeredCompanyIds`, or global).
- **`GET /api/v1/users`:**
  - Without `companyId`: SUPERADMIN only, paged as today.
  - With `companyId`: SUPERADMIN, or ADMIN_EMPRESA of that company. Answers a plain array of the accounts that have an employee in that company. A company outside the caller's scope answers 404 `COMPANY_NOT_FOUND`; inside the scope but not administered answers 403.
  - `UserResource` = `{id, email, mustChangePassword, lastLoginAt, isSuspended, activeSuspensionId, employeeId, roles}`:
    - `isSuspended` and `activeSuspensionId` refer to a suspension in force now.
    - `employeeId` is the public id of the account's employee in `companyId`, and null without `companyId`.
    - `roles` are the assignments in force: `[{assignmentId, roleId, roleCode, roleName, scopeLevel, companyId, companyName, branchId, branchName, commercialBrandId, commercialBrandName, expiresAt}]`. With `companyId`, the SYSTEM-scope assignments are included as well as the assignments on that company. Without it, all assignments in force.
  - Everything is resolved in batch, with no query per row (skill §4).
- **Cross-company link:** an existing account whose employees are in another company can be linked only by a SUPERADMIN. The error becomes `403 ACCOUNT_LINK_REQUIRES_SUPERADMIN` (map it to 403 in `ErrorResponseAssembler`, like `ACCOUNT_SUSPENDED`), so the screen can say why. A caller without permission still answers `ACCESS_DENIED`.
- **Revocation** takes no body; the reason is always `retiro`.
- **Section 6 of the contract, unchanged and now due:**
  - Scope filter with 404 on every read of companies, branches, brands, locations and employees: list, export, by id and nested routes.
  - Creating and updating employees requires administering the company.
  - Regression tests: item 6 (the 404 half) and item 7.

## R4. Error codes the frontend maps (exactly as the backend builds them)

All 422 unless noted:

| Operation | Codes |
|---|---|
| Access grant | `EMPLOYEE_NOT_FOUND` 404, `ACCESS_DENIED` 403, `ACCOUNT_LINK_REQUIRES_SUPERADMIN` 403, `EMPLOYEE_INACTIVE`, `EMPLOYEE_ACCOUNT_CONFLICT` 409, `BUSINESS_RULE_VIOLATION` (SYSTEM account), `VALIDATION_ERROR` 400 |
| Role grant | `USER_NOT_FOUND` / `ROLE_NOT_FOUND` / `COMPANY_NOT_FOUND` / `BRANCH_NOT_FOUND` / `COMMERCIALBRAND_NOT_FOUND` 404, `ACCESS_DENIED` 403, `ROLE_ASSIGNMENT_CONFLICT` 409, `ROLE_ASSIGNMENT_SCOPE_MISMATCH`, `ROLE_ASSIGNMENT_EMPLOYEE_REQUIRED`, `BUSINESS_RULE_VIOLATION` (SYSTEM target, expiry not later) |
| Revocation | `ROLE_ASSIGNMENT_NOT_FOUND` 404, `ACCESS_DENIED` 403, `ROLE_ASSIGNMENT_INACTIVE`, `LAST_ACTIVE_SUPERADMIN` |
| Suspension open | `USER_NOT_FOUND` 404, `ACCESS_DENIED` 403 (own account, or not administered), `ACCESS_SUSPENSION_CONFLICT` 409, `LAST_ACTIVE_SUPERADMIN`, `BUSINESS_RULE_VIOLATION` (end before start, SYSTEM), `VALIDATION_ERROR` 400 |
| Suspension closure | `ACCESS_SUSPENSION_NOT_FOUND` 404, `ACCESS_DENIED` 403, `ACCESS_SUSPENSION_CLOSED` |
| Password reset | `USER_NOT_FOUND` 404, `ACCESS_DENIED` 403, `BUSINESS_RULE_VIOLATION` (SYSTEM) |
| Termination | `EMPLOYEE_NOT_FOUND` 404, `ACCESS_DENIED` 403, `EMPLOYEE_TERMINATION_DATE_IN_FUTURE`, `LAST_ACTIVE_SUPERADMIN`, `BUSINESS_RULE_VIOLATION` (date rules). No more 409 |
| Reactivation | `EMPLOYEE_NOT_FOUND` 404, `ACCESS_DENIED` 403, `BUSINESS_RULE_VIOLATION` |
| Termination and reactivation answer | `{id, isActive, hireDate, terminationDate, terminationReason}` |

## R5. Frontend reconciliation

- **Error codes:** map R4 exactly, one message per code. Drop the messages that listed every possible cause.
- **Suspensions:**
  - Read `status` instead of computing where a period stands (remove what `b221e78` added for that, if `status` makes it redundant).
  - The action reads "Cancelar" for SCHEDULED and "Levantar" for IN_FORCE, both through `closure`; ENDED and CANCELLED offer none.
  - Show who blocked (`createdBy` is an account id; show it only if there is a cheap way to name it, otherwise leave it out).
- **Revocation** sends no body.
- **Company accounts (`GET /users?companyId`):** `UserRoleResource` gains `roleId`, `companyId`, `companyName` and `expiresAt`. SYSTEM roles appear, labeled as reaching the whole system, so a SUPERADMIN no longer shows "Sin roles".
- **`administered`:**
  - Request accounts and show the account column and actions only for companies where the account is administered.
  - Employee create and edit only where administered.
  - A company that is in scope but not administered shows the employees read-only, without error toasts. This fixes the mixed-role case 8 of the round 1 report.
- **Time zones (dictionary §2.6, row "Hoy"):**
  - An account and a suspension do not belong to a company, so their days stay in `America/Lima`, as today. Name it once as the dictionary rule, not as a scattered literal.
  - A role's expiry uses the zone of its branch for a BRANCH role (the branch carries `timezone`; take it from the scope options), and otherwise the zone of its company (`companies[].timeZone` of the session).
- **Termination and reactivation:** no change needed; the frontend already reloads.
- **e2e mocks:** follow R2–R4 exactly.
