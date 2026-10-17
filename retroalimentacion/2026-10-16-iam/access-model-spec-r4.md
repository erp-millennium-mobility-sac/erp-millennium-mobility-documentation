# Access model, round 4: access administrator (TI) and the security track (16/10/2026)

This file amends `access-model-spec.md`, `-r2.md` and `-r3.md`, in the same folder. Where they differ, this one wins.

**Origin.** The user approved "las recomendadas end to end" on 16/10/2026. The decisions are D4-1 to D4-9 below.

**What is replaced.** Where they replace earlier work of rounds 1–3, remove what they replace: no shims, no unused codes or members.
- The cross-company link rule and its code `ACCOUNT_LINK_REQUIRES_SUPERADMIN` go.
- Account administration by ADMIN_EMPRESA goes.
- The brand-less ADMIN_EMPRESA special case of r3 B10, if it becomes redundant, goes.
- `activeSuspensionId` goes.

## Decisions

**D4-1. A single access administrator for the whole group.**
- New system role:
  - code `ADMIN_ACCESOS`;
  - name "Administrador de accesos";
  - description "Da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas, en todas las empresas del grupo";
  - `scope_level = SYSTEM`, `is_system = true`, `company_id` NULL.
- Seed it in a new migration (the next free number), the same way the existing system roles were seeded.
- It is global: like SUPERADMIN, it sees every company (RLS `SYSTEM`).

**D4-2. Three limits on ADMIN_ACCESOS.**
- It never grants roles to itself, and never acts on its own account.
- It never grants `SUPERADMIN` or `ADMIN_ACCESOS`; only a SUPERADMIN does.
- It never acts on a **protected account**: one that holds `SUPERADMIN` or `ADMIN_ACCESOS` in force. Acting means:
  - linking it to an employee;
  - granting it roles or revoking its roles;
  - suspending it or closing its suspensions;
  - resetting its password.
- Refusals on a protected account answer `403 ACCOUNT_MANAGED_BY_SUPERADMIN`, mapped to 403 in `ErrorResponseAssembler`. It replaces `ACCOUNT_LINK_REQUIRES_SUPERADMIN`, which is removed.
- Granting a reserved role, or acting on oneself, answers `403 ACCESS_DENIED`.

**D4-3. An explicit exception to dictionary §5.5 rule 2.** ADMIN_ACCESOS assigns roles whose permissions it does not hold, within D4-2. The text goes into the dictionary proposal; the code follows it.

**D4-4. ADMIN_EMPRESA keeps its people and loses accounts (segregation of duties).**
- It creates and edits the employees of its company and records their exits and returns.
- It does not give access, grant or revoke roles, suspend, close suspensions or reset passwords.
- An exit or a return keeps its automatic consequences:
  - the exit revokes the roles in that company (`baja_colaborador`);
  - the exit of the last active employee closes the account (`baja`, `baja_cuenta`, sessions closed);
  - the return closes the `baja`.
- An exit that would close a protected account requires a SUPERADMIN: `403 ACCOUNT_MANAGED_BY_SUPERADMIN`. The last-superadmin guard stays.
- ADMIN_ACCESOS does **not** create or edit employees, and does not record exits or returns. One person must not be able to invent an employee and give it access.

**D4-5. SUPERADMIN keeps everything.**
- It does all that ADMIN_ACCESOS does, without D4-2's limits, except acting on its own account and on SYSTEM. It also keeps the company and catalog writes.
- No "last ADMIN_ACCESOS" guard is needed: SUPERADMIN can always manage access.

**D4-6. Every actor is an employee.** ADMIN_ACCESOS and ADMIN_EMPRESA act only while their account is linked to an active employee, as grantors already must (§5.2, §5.7). The first SUPERADMIN self-links first, as today.

**D4-7. The permission matrix is due before step 7.** ADMIN_ACCESOS sees every company. Today that covers companies, branches, brands, locations and employees, which it needs. Before step 7 (purchases), the permission matrix must restrict what a global role reads. This is recorded in the implementation order, with no code now.

**D4-8. ADMIN_EMPRESA is never limited to a brand.**
- Granting `ADMIN_EMPRESA` with a `commercialBrandId` answers `422 ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`.
- The database cannot see the role in one row (dictionary §5.7 says such rules live in the application), so it is validated in the application and tested.
- Other company and branch roles may still be limited to a brand (§5.7: "un asesor vea solo Lima Bikes").

**D4-9. Security blocks have their own timeline.**

What changes:
- Periods of the same track never overlap, even in the past.
  - The employment track holds `vacaciones`, `licencia` and `baja`.
  - The security track holds `seguridad`.
  - A `seguridad` period may overlap an employment period.
- **Constraint.** Replace the V28 exclusion with one that compares the track. V28 is unmerged, so edit it in place and recreate the verify DB; or alter it in a new migration. Choose and justify. For example, an `EXCLUDE USING gist (user_id WITH =, <track expression> WITH =, period WITH &&) WHERE (<not cancelled>)`. Make sure `btree_gist` supports the operator class used.
- **Exit that closes the account:**
  - it ends now the `vacaciones`/`licencia` in force and cancels the scheduled ones;
  - it never touches `seguridad`;
  - it opens `baja` from now, unless an open `baja` is already in force.
- **Return and link** (r3 B1) close only the `baja`. A `seguridad` in force keeps the account blocked until someone closes it.

What stays the same:
- A manual overlapping suspension answers `409 ACCESS_SUSPENSION_CONFLICT` only within the same track.
- An account is suspended when **any** period is in force:
  - sign-in answers `403 ACCOUNT_SUSPENDED`;
  - sessions close when any period starts;
  - the last-superadmin guard counts every period, as r3 B7 defined.

Resource change:
- `UserResource` loses `activeSuspensionId`, which is ambiguous with two tracks, and keeps `isSuspended`.
- Actions on blocks go through the suspensions dialog, which lists every period.

## Backend: who may do what (the policy to implement)

**Legend:**
- **P** = a protected account (holds SUPERADMIN or ADMIN_ACCESOS in force).
- **own** = the caller's own account.
- **SYS** = the SYSTEM account.
- **ADMIN_EMPRESA (C)** = ADMIN_EMPRESA of company C, linked to an active employee.

| Operation | SUPERADMIN | ADMIN_ACCESOS | ADMIN_EMPRESA (C) | Any other role |
|---|---|---|---|---|
| `POST /access-grants` (create or link) | yes; linking P too | yes in any company, except linking P (403 `ACCOUNT_MANAGED_BY_SUPERADMIN`) | 403 | 403 |
| Grant a role | any role, to anyone but SYS | any role but SUPERADMIN/ADMIN_ACCESOS; never to own or P | 403 | 403 |
| Revoke an assignment | any (last-superadmin guard) | never a SUPERADMIN/ADMIN_ACCESOS assignment, never own, never on P | 403 | 403 |
| Open or close a suspension | not own, not SYS (guard) | not own, SYS or P | 403 | 403 |
| Password reset | not own, not SYS | not own, SYS or P | 403 | 403 |
| `GET /users` (paged), export, `/users/{id}`, role assignments, suspensions | yes | yes | 403 | 403 |
| `GET /users?companyId` | yes | yes | yes, read-only, for C | 403 |
| Create or edit employees | yes | 403 | yes, in C | 403 |
| Termination, reactivation | yes | 403 | yes, in C; an exit closing P needs SUPERADMIN | 403 |
| Writes on companies, branches, brands, locations | yes | 403 | 403 | 403 |
| Reads of organization data | all | all (global) | its scope | its scope |

**How it fits the scope model:**
- **`AccessScope.global` means sees every company.** It is true for any SYSTEM-scope role in force (SUPERADMIN, ADMIN_ACCESOS) and gives the RLS `SYSTEM` scope.
- **Administering a company is separate.** It covers employee writes, exits and returns, `/me` `administered` and `/users?companyId`. It belongs to SUPERADMIN and to the ADMIN_EMPRESA of that company only. ADMIN_ACCESOS administers no company: its `/me` `administered` is false everywhere.
- **The model must say these apart.** Today `global` probably implies administering everything; separate the two (for example, a flag for SUPERADMIN).
- **Elevations.** Keep `SystemAccessScope` elevations only where a company-scoped caller still needs facts beyond its scope: an ADMIN_EMPRESA's exit checks "another active employee elsewhere" and "is the account protected". Remove the ones that only served account administration by ADMIN_EMPRESA.

**Regression tests**, all connected as `erp_mm_app`:

ADMIN_ACCESOS:
- Grants CONTABILIDAD in Peru → 201.
- Grants SUPERADMIN, or ADMIN_ACCESOS, or anything to itself → 403.
- Resets the password of a SUPERADMIN, or of another ADMIN_ACCESOS → 403 `ACCOUNT_MANAGED_BY_SUPERADMIN`.
- Links an existing account of another company → 200; links a protected account → 403 `ACCOUNT_MANAGED_BY_SUPERADMIN`.
- Creating or editing an employee, or recording an exit → 403.
- Not linked to an employee → 403 on a grant.

ADMIN_EMPRESA:
- An access grant, a role grant, a revocation, a suspension, a closure or a reset → 403.
- Creating an employee in its company → 201.
- An exit closing a normal account → 200; an exit closing an ADMIN_ACCESOS account → 403 `ACCOUNT_MANAGED_BY_SUPERADMIN`, and the same exit by a SUPERADMIN → 200.
- `GET /users?companyId` of its company → 200.

Granting ADMIN_EMPRESA with a brand → 422 `ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`.

The security track, in order:
1. With a `seguridad` in force, an exit keeps it open and opens `baja`.
2. A return closes the `baja`, and the `seguridad` is still in force.
3. Sign-in answers 403 `ACCOUNT_SUSPENDED`.
4. A `vacaciones` overlapping a `seguridad` is accepted; a `vacaciones` overlapping a `licencia` → 409.

## Clarifications added while building (16/10/2026)

**C4-1. Nobody acts on their own account, SUPERADMIN included.**
- No granting or revoking one's own roles, no suspending oneself, no resetting one's own password. Changing one's own password goes through its own endpoint.
- D4-5 wins over the table's "any role, to anyone but SYS".
- Self-acts answer 403 `ACCESS_DENIED`.

**C4-2. The `baja` belongs to the exit.**
- `POST /users/{id}/suspensions` accepts only `vacaciones`, `licencia` and `seguridad`; `baja` answers 400 `VALIDATION_ERROR`, like an unknown reason.
- A `baja` opens only through the exit of the last active employee. It closes only through a return or a link (r3 B1); a manual closure of a `baja` answers 422 `ACCESS_SUSPENSION_BAJA_RESERVED`.
- So the exit that closes the account:
  - cancels every scheduled employment-track period (`vacaciones`, `licencia`, and any legacy `baja`);
  - ends the `vacaciones` or `licencia` in force;
  - keeps an open `baja` in force;
  - never touches `seguridad`.
- Frontend: the block dialog does not offer `baja`, and offers no close action on a `baja` row; it shows that the return closes it.

## Frontend

**Capabilities:**
- `managesAccess` = the session holds SUPERADMIN or ADMIN_ACCESOS. Derive it once in `IamStore` from the session roles; there is no new backend field.
- `administers(companyId)` stays as it is (`/me` `administered`).

**Users screen:**
- Route and menu for SUPERADMIN and ADMIN_ACCESOS.
- Account actions follow the table above. For an ADMIN_ACCESOS session, hide them on protected accounts: the account's roles include SUPERADMIN or ADMIN_ACCESOS.
- Never on one's own row or on SYSTEM.

**Employees screen:**
- For ADMIN_EMPRESA:
  - create, edit, exit and return where administered;
  - the account columns (access, roles) are read-only, with no account actions.
- For ADMIN_ACCESOS:
  - the company selector lists every company;
  - employees are read-only, with no create, edit, exit or return;
  - account actions on non-protected accounts.
- For SUPERADMIN: everything.

**Roles dialog:**
- An ADMIN_ACCESOS session is never offered SUPERADMIN or ADMIN_ACCESOS.
- The brand field is not offered when the role is ADMIN_EMPRESA.

**Blocks:**
- No `activeSuspensionId`. The row's block action opens the suspensions dialog, which lists every period: two can be in force, one per track.
- "Levantar" and "Cancelar" act per period.

**Error codes:**
- Map `ACCOUNT_MANAGED_BY_SUPERADMIN` and `ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`.
- Remove `ACCOUNT_LINK_REQUIRES_SUPERADMIN`.

**e2e mocks follow all of the above, with these scenarios:**
- an ADMIN_ACCESOS session giving access across companies and granting roles;
- no actions on a SUPERADMIN row;
- an ADMIN_EMPRESA session with read-only account columns and employee writes;
- a security block that survives an exit and a return.
