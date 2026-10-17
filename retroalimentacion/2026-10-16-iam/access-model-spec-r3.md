# Access model, round 3: fixes after the adversarial review (16/10/2026)

This file amends `access-model-spec.md` and `access-model-spec-r2.md`, in the same folder. Where they differ, this one wins.

**Origin.** An adversarial review of `feature/access-model` in both repos: 6 lenses, 2 skeptics per finding, 33 findings confirmed by both. The review is in the scratchpad (`review.json`). Below are the fixes, by repo, each with the finding it closes.

**Precedence:** dictionary > own skills > code. Dictionary sections cited:
- §5.4: suspensions; "El sistema impide que dos bloqueos de la misma persona se crucen en el tiempo, incluso en el pasado".
- §5.7: user_roles; a brand may limit any non-global role.
- §5.8.5: an expired access is closed by the system with `caducidad`.
- §5.9: `baja` revokes every assignment with `baja_cuenta` in the same operation; last-superadmin protection.
- §2.6: dates and "today".

## Backend (`mm-acc-be`)

**B1. A rehire by linking closes the `baja`.**
- When an access grant links an existing account that has an open `baja` in force, the same operation closes it now, closed by the requester, because the account has an active employee again. Reactivation already does the same.
- Suspensions with another reason are not touched.
- Test it.

**B2. A closure always happens now.**
- `POST /suspensions/{id}/closure` takes no body: remove `CloseAccessSuspensionResource` and `closedAt` from the command.
- An IN_FORCE suspension ends now. A SCHEDULED one is cancelled now.
- The `BUSINESS_RULE_VIOLATION` path for a client `closedAt` disappears. A client moment could falsify the history of who had access.

**B3. Linking another person's account requires a SUPERADMIN.**
- A non-SUPERADMIN that names an existing account other than its own gets `403 ACCOUNT_LINK_REQUIRES_SUPERADMIN`. This includes an account with no employee and no role anywhere.
- `409 EMPLOYEE_ACCOUNT_CONFLICT` keeps the cases where the employee or the account is already linked in that company.
- `ACCESS_DENIED` remains for a caller that does not administer the employee's company or is not linked to an employee.
- Document the precedence in the controller's OpenAPI.

**B4. The branch filter of a company's employee list never drops.**
- `GET /companies/{id}/employees?branchId=` with a branch that is unknown or belongs to another company must not drop the filter.
- Follow conventions §6.1 for filters. If they are silent, an unknown filter value gives an empty result, never the unfiltered list.

**B5. `LocationType` leaves Jackson.** Move the Jackson annotation out of `organization/domain/model/valueobjects/LocationType` (rule 2.1). The mapping belongs to the interface layer.

**B6. The users export follows the list.**
- `GET /users` with xlsx (`Accept`) honors `companyId` with the same rules and rows as the list.
- The state column says Activo or Bloqueado from the suspension in force, not the old flag (S18, S33).

**B7. The last superadmin, over time and under concurrency** (S2, S6, S7, S27).
- An operation that removes, or will remove, the SUPERADMIN capability of an account is allowed only if another account stays durably SUPERADMIN. The operations are revocation, a suspension now or scheduled, and an exit that closes the account.
- Durably means: a SUPERADMIN assignment not revoked and without expiry, and no suspension in force or scheduled that is not cancelled.
- Serialize those operations with a transaction-scoped advisory lock, or an equivalent row lock, so two concurrent requests on two different superadmins cannot both pass.
- Test both the time case and the race, if a race test is feasible.

**B8. Expiry (`caducidad`) per §5.8.5.**
- A scheduled process (the first `@Scheduled` of the project: `@EnableScheduling`, its interval in configuration) runs inside `SystemAccessScope`. It closes every assignment past its `expires_at` with `revoked_at = expires_at` and the reason `caducidad`.
- Granting closes only the expired assignment with the same key (role and scope) before inserting, never every expired assignment of the account. This fixes the RLS UPDATE failure (500) when an ADMIN_EMPRESA grants to an account with an expired SYSTEM assignment (S4, S10, S17, S30).
- Every "in force" computation uses `revoked_at IS NULL AND (expires_at IS NULL OR expires_at > now)`, so access stops exactly at expiry without waiting for the process. That covers authorities, the scope, `/me`, the lists, the guards and the bootstrap seed check (S9).

**B9. A suspension starts now or later** (S5, S8, S28).
- `POST /users/{id}/suspensions` with a `startsAt` before the moment of the request answers `422 ACCESS_SUSPENSION_START_IN_PAST`.
- An absent `startsAt` means now.
- §5.4 keeps history true: a block that did not happen cannot be recorded.

**B10. An ADMIN_EMPRESA limited to a brand does not administer the company** (S1).
- Only an ADMIN_EMPRESA assignment in force **without a brand** counts as administering a company. This applies to:
  - account administration (password reset, suspensions);
  - exits and returns that close or reopen an account;
  - access grants;
  - employee writes;
  - `AccessScope.administeredCompanyIds`;
  - `/me` `administered`;
  - `/users?companyId`.
- A brand-limited ADMIN_EMPRESA keeps only what `GrantPolicy` already gives it: granting inside its brand.
- Test the takeover: a brand-limited administrator resetting the password of a brand-less administrator of the same company → 403.

**B11. Concurrency on accounts and employees** (S11, S12, S13).
- Operations that read and then change an account take a pessimistic lock on that account's `users` row. These are the access grant, the exit and the return, opening and closing a suspension, granting and revoking a role, and the password reset.
- Linking and the exit lock the `employees` row they change.
- If the V28 exclusion constraint is still violated by a race, the answer is `409 ACCESS_SUSPENSION_CONFLICT`, not 500. Use the existing pattern for translating constraint violations, if there is one.

**B12. Each exit and return rule has its own code** (S25). Replace `BUSINESS_RULE_VIOLATION` on termination and reactivation with these 422 codes:

| Operation | Rule | Code |
|---|---|---|
| Termination | the employee is already inactive | `EMPLOYEE_ALREADY_TERMINATED` |
| Termination | the date is before the hire date | `EMPLOYEE_TERMINATION_BEFORE_HIRE` |
| Termination | the date is after today (unchanged) | `EMPLOYEE_TERMINATION_DATE_IN_FUTURE` |
| Reactivation | the employee is already active | `EMPLOYEE_ALREADY_ACTIVE` |
| Reactivation | the new hire date is before the termination date | `EMPLOYEE_REHIRE_BEFORE_TERMINATION` |

- If `EmployeeAffiliationPolicy.validateTermination` or `validateRehire` holds another distinct rule, give it its own `EMPLOYEE_…` code and report it.
- Do not change the codes of employee create and update; the frontend reads them from your code.

**Not changed, on purpose:**
- **S3.** An ADMIN_EMPRESA records exits in its own company. The roles the exit revokes are only those in that company (`baja_colaborador`). Closing the account still requires administering it.
- **Dictionary behavior.** A manual `baja` revokes in the same operation, as §5.9 says. An exit ends a security block in force before it opens the `baja`. That last point is an open question for the user.

## Frontend (`mm-acc-fe`)

- **F1. The account flag is `hasAccount` (S14, S19, critical).**
  - The backend sends `hasAccount` on the employee resource, and the frontend reads `hasAccess`, so no employee ever shows access.
  - Read `hasAccount` across the resource, the assembler, the entity name if it mirrors it, the views and the mocks.
- **F2. Employee errors use the real codes (S15).** Map employee create and update errors by the codes the backend really sends; read `EmployeeCommandServiceImpl` and its policies in `mm-acc-be` at HEAD.
- **F3. "Today" follows the backend's zone order (S16, S21, S32).** The exit and return dialogs compute today in the employee's branch zone, then the company zone (`companies[].timeZone`), then `America/Lima`, which is the backend's order.
- **F4. A company created during the session can be administered (S20).**
  - A global account administers every company, including one created during the session.
  - The roles dialog's company list must not be a stale sign-in snapshot: reload `/me` when it opens, or derive it for a global account from the companies the scope options can load. Choose and justify.
- **F5. Stale responses (S23).** A late response for one account's roles or suspensions must never show under another account's dialog.
- **F6. The grant button never sticks (S24).** It must not stay disabled when the role changes while the branches are loading.
- **F7. Exit and return codes (S25).** Map the five codes of B12, one message each.
- **F8. Moments in Lima (S26).** `lastLoginAt` and any other moment of an account shown in IAM display in `America/Lima` as `dd/mm/aaaa hh:mm` (§2.6).
- **F9. A company administrator sees scheduled blocks (S29).** From the employees view, an ADMIN_EMPRESA opens the account's suspensions dialog: list, Cancelar for SCHEDULED, Levantar for IN_FORCE, Bloquear. Use the same dialog the users screen uses.
- **F10. The session follows a change of one's own roles (S31).** After a grant or revocation on the signed-in account, reload the session (`/me`) so menus and guards follow.
- **F11. No account actions where the backend always refuses (P1, P2).**
  - Do not offer them on an inactive employee.
  - Do not offer them on the signed-in user's own row, in either the users or the employees view.
  - Do not offer them on the SYSTEM account.
- **F12. Suspension requests (B2, B9).**
  - The closure sends no body.
  - Opening a block "from today" sends no `startsAt`; a future day sends the start of that day in Lima.
  - Map `ACCESS_SUSPENSION_START_IN_PAST`.
- **F13. Mocks follow all of the above** (`e2e/support/api.ts`), with e2e for:
  - an employee with account, on the real field;
  - an ADMIN_EMPRESA cancelling a scheduled block from the employees view;
  - no actions on one's own row;
  - an exit code message.
