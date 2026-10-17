# Access model, round 5: closing the separation of duties, final review and real-integration probe (16/10/2026)

This file amends `access-model-spec.md`, `-r2.md`, `-r3.md` and `-r4.md`, all in the same folder. Where they differ, this one wins.

**Origin.** On 16/10/2026 the user approved recommendations A to G ("Hazlo"). These come from three sources:
- the final adversarial review, in the scratchpad file `review-final.json`;
- the real-integration probe, in the scratchpad folder `probe\`;
- an analysis against NIST SP 800-53 (AC-5, AC-6(5), PS-4), ISO/IEC 27001:2022 (A.5.3, A.5.18, A.8.2) and ANSI INCITS 359 (RBAC static separation of duty).

**Remove what is replaced.** No shims.

## Policy decisions (A to G)

**A. ADMIN_EMPRESA is a reserved role.**
- Only a SUPERADMIN grants or revokes it.
- An ADMIN_ACCESOS session gets 403 `ACCESS_DENIED`, the role-policy refusal, as for SUPERADMIN and ADMIN_ACCESOS.
- ADMIN_ACCESOS keeps granting the operational roles: OPERACIONES, CONTABILIDAD, ASESOR_VENTAS, TALLER and any non-reserved role.

**B. The accounts of company administrators are protected too.**
- A protected account is one holding SUPERADMIN, ADMIN_ACCESOS **or ADMIN_EMPRESA** in force.
- ADMIN_ACCESOS never acts on one: no linking, no granting or revoking any of its roles, no suspending or closing its suspensions, no resetting its password. Each attempt answers 403 `ACCOUNT_MANAGED_BY_SUPERADMIN`.
- Why: without this, ADMIN_ACCESOS resets a company administrator's password, signs in as them, invents an employee and gives it access. One person would defeat the separation of duties.

**C. An exit always removes access.**
- Remove the round 4 rule that "an exit closing a protected account needs a SUPERADMIN". Termination no longer answers `ACCOUNT_MANAGED_BY_SUPERADMIN`.
- The exit always revokes the roles in that company. If it is the last active employee, it also closes the account: `baja`, `baja_cuenta`, sessions closed, and the security track untouched.
- The only remaining stop is the last-superadmin guard (422 `LAST_ACTIVE_SUPERADMIN`).
- Why: PS-4 and A.5.18 ask to remove access as soon as the person leaves. A wrongful exit is reversible by a return; a leaver who keeps access is the real risk.
- Remove the elevation fact "is the account protected" from the exit path if nothing else needs it. Keep "another active employee elsewhere".

**D. Static separation of duty.**
- One account never holds ADMIN_ACCESOS and ADMIN_EMPRESA in force at the same time.
- Granting one to an account that holds the other answers 422 `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`.
- Validate it in the application, and test it. The database cannot see two rows at once without a trigger; if you judge a trigger worth it, justify it.

**E. A privileged role goes only to an account with its own password.**
- SUPERADMIN, ADMIN_ACCESOS and ADMIN_EMPRESA (the roles that make an account protected, which extends the approved text consistently with B) are granted only to an account with `must_change_password = false`.
- Otherwise the grant answers 422 `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`.
- So whoever handed out a temporary password can never end up holding a privileged account.
- The bootstrap superadmin is created by the application, not granted, so it is unaffected.

**F. Nobody records their own exit or return.**
- An exit or a return of an employee linked to the caller's own account answers 403 `OWN_ACCOUNT_FORBIDDEN` (see T2).
- This is the AC-5 two-person rule applied to the person's own lifecycle.

**G. A return is never dated in the future.**
- A reactivation with `hireDate` after today answers 422 `EMPLOYEE_REHIRE_DATE_IN_FUTURE`.
- "Today" is computed in the employee's zone by the same rule as the exit: branch, then company country, then America/Lima.

## Technical fixes (final review and probe)

**T1. No roles dialog on one's own account (frontend).**
- After an access grant that linked the caller's own account, do not offer "Asignar roles" and do not open the roles dialog.
- `AccountDialogs.manageRoles` does nothing for one's own account.
- The roles dialog never offers grant or revoke on one's own account.

**T2. One code per cause for "forbidden", in place of the catch-all.** Map both new codes to 403 in `ErrorResponseAssembler`.
- `403 OWN_ACCOUNT_FORBIDDEN`: the caller acts on their own account. That covers grant, revoke, suspend, close, reset, linking oneself when not allowed (T8), and the own exit or return (F).
- `403 ACTOR_EMPLOYEE_REQUIRED`: the caller's account is not linked to an active employee, so it cannot act (D4-6).
- `403 ACCESS_DENIED` stays for "your role cannot do this", including a reserved role.
- The frontend maps all three per operation, with exact Spanish messages, and removes the texts that named causes that no longer exist.

**T3. Lifting a block in a world with two tracks (frontend).**
- The lift and cancel confirmations name the block's reason.
- If another period stays in force, the confirmation says the account stays blocked by it, for example "seguirá bloqueada por baja".
- Drop the sentence about withdrawn roles.

**T4. The zone of an employee's dates comes from the backend.**
- `EmployeeResource` gains `timeZone`: the IANA zone that governs that employee's dates (branch, then company country, then America/Lima). It is the same rule the termination and the return use. It is resolved in batch for lists, never with a query per row.
- The frontend exit and return dialogs use it, and stop resolving the zone from the branches store.

**T5. A refusal that means the screen is stale reloads it (frontend).**
- These codes reload the affected list or dialog: `ACCESS_SUSPENSION_CLOSED`, `ROLE_ASSIGNMENT_INACTIVE`, `ACCOUNT_MANAGED_BY_SUPERADMIN`, `EMPLOYEE_ALREADY_TERMINATED`, `EMPLOYEE_ALREADY_ACTIVE`, and any other "already changed" code.
- That way the buttons reflect the real state, as `EmployeesStore` already does in some places.

**T6. Lock, then read (backend).** `findByIdForUpdate` (and any other `…ForUpdate` read) returns the row as read **after** the lock, not a copy cached earlier in the transaction: refresh or bypass the persistence context. Add a test with a requester that is also the target.

**T7. Dead code (backend).** Remove the members the locking reads replaced, if nothing uses them: `ExternalOrganizationService.fetchEmployeeByPublicId`, the facade method `fetchEmployeeSummaryByPublicId` and its query handler, and `AccountReach.userId`. Check usages first.

**T8. Self-link only at bootstrap (backend).**
- A SUPERADMIN may link its own account only while it has no active employee (dictionary §5.2 step 6).
- Otherwise the self-link answers 403 `OWN_ACCOUNT_FORBIDDEN`.
- Test that an already-linked SUPERADMIN cannot self-link to an employee of another company.

**T9. Mocks follow the real backend (frontend, from the review and the probe).** In `e2e/support/api.ts`:
- An ADMIN_EMPRESA session gets 403 on `GET /users/{id}/role-assignments` and `/suspensions`.
- Permission is checked before existence: 403 `ACCESS_DENIED` before `*_NOT_FOUND` for callers without the role.
- An unknown company id answers 404 `COMPANY_NOT_FOUND`, never `companies[0]`.
- `/me` `companies[].name`, `UserRoleResource.companyName` and `RoleAssignmentResource.companyName` carry the **trade name**, as the backend does.
- Every A to G and T rule is mirrored.

**P1. One company name in selectors (frontend).** Company selectors show the **trade name**, as IAM does with the backend's data. The legal name stays where the company itself is shown or edited.

**P2. Types follow the real resources (frontend).**
- `CompanyResource` has `publicId`, not `id`; remove the `publicId ?? id` fallback.
- `GET /roles` answers a plain array; fix the type and the assembler.
- `UpdateEmployeeRequest.hireDate` is required.

**P3. Emails are trimmed (backend).** Contract §4 says the access-grant email is normalized with trim. Trim before validation, so `"  ana@x.ky "` is accepted as `ana@x.ky`; lower-casing already works. Apply the same normalization wherever an email enters (sign-in, if it does not already).

**P4. Codes that still fall to the generic message (frontend).** Map them with real messages:
- role grant: `VALIDATION_ERROR`;
- termination: `VALIDATION_ERROR`, `BUSINESS_RULE_VIOLATION` (date), `ACCESS_SUSPENSION_CONFLICT`;
- reactivation: `BUSINESS_RULE_VIOLATION`, `VALIDATION_ERROR`;
- employee save: `VALIDATION_ERROR`;
- sign-in: `VALIDATION_ERROR`;
- plus every new code of this round.

## Not changed, recorded

- **Menu "Roles y permisos".** It is a placeholder page, still shown to ADMIN_EMPRESA. Who creates company roles is decided with the permission matrix, which is due before step 5 according to the implementation order's adjustment 3.
- **The 401 on cold load.** It comes from `GET /me` while restoring the session; that is the standard pattern.
- **`GET /users/{id}`.** It is not served and nothing calls it; it is removed from the r4 table in the docs.
- **A block scheduled before a promotion stays.** It is visible and a SUPERADMIN can cancel it.
- **Password reset shows the temporary password to the administrator.** The ideal is a one-time link sent to the person's email, which needs mail infrastructure; it is recorded as pending.

## Tests to add (all as `erp_mm_app` on the backend)

ADMIN_ACCESOS:
- Granting or revoking ADMIN_EMPRESA → 403 `ACCESS_DENIED`.
- Acting on an account holding ADMIN_EMPRESA → 403 `ACCOUNT_MANAGED_BY_SUPERADMIN`: reset, suspend, link, grant CONTABILIDAD to it.

Exits and returns:
- An ADMIN_EMPRESA's exit closing an ADMIN_ACCESOS account → 200, with the account closed (C).
- Own exit or return → 403 `OWN_ACCOUNT_FORBIDDEN`.
- Return dated tomorrow → 422 `EMPLOYEE_REHIRE_DATE_IN_FUTURE`.

Grants:
- ADMIN_EMPRESA to an ADMIN_ACCESOS holder, and the reverse → 422 `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`.
- SUPERADMIN, ADMIN_ACCESOS or ADMIN_EMPRESA to an account with a temporary password → 422 `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`, and → 201 after it changed its password.

Own account and linking:
- A self-act → 403 `OWN_ACCOUNT_FORBIDDEN`.
- Acting while not linked to an employee → 403 `ACTOR_EMPLOYEE_REQUIRED`.
- A linked SUPERADMIN self-linking → 403 `OWN_ACCOUNT_FORBIDDEN`.

Other:
- `EmployeeResource.timeZone` for a branch in another zone.
- Email with spaces accepted.
- A lock-then-read test.
