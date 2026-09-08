---
name: opc
description: Configure and govern an Oracle Database project with optional APEXlang apps, enforcing project preferences before delegating to DB and APEX skills.
---

# Oracle Preference Controller (OPC)

Use OPC as the project entry point for Oracle database and APEX work. It owns project setup, preference enforcement, reusable personal-rule selection, and safe routing; it does not replace the configured `/db` or `/apex` expertise.

## Non-negotiables

- Find the nearest `oracle_project_config.toml` by walking upward from the current directory. Its directory is the project root. Before project, database, or APEX work, initialize a project in the current directory if none exists. Managing the personal global rule library is the only exception and does not require a project config.
- A personal global rule library is optional and is never a project base config. Offer its rules for explicit selection during new-project setup; do not silently apply them or change an existing project because the library changed.
- Read valid config and apply relevant project rules before delegating. Project preferences override dependency defaults, never safety/security constraints.
- Never store passwords, tokens, client secrets, certificates, private keys, or SAML material in either config. Store aliases, IDs, paths, names, and non-secret object references only.
- TOML has no `null`: omit unassigned optional keys. Omitted preferences inherit the relevant dependency default.
- Do not silently select an ambiguous application, workspace, connection, path, authorization model, or destructive action. Ask.
- A one-off task instruction is not a persisted rule unless the user explicitly asks to save/change a rule.

## Personal global rule library

Use the personal file `$CODEX_HOME/oracle-preference-controller/global_config.toml`; when `CODEX_HOME` is unset, use `~/.codex/oracle-preference-controller/global_config.toml`. This file belongs to the current user, outside every project and outside source control. Do not put its path in a project config or create it in a repository.

The library contains reusable **rule templates only**, not a second copy of a project configuration. In particular, it must not contain connection aliases, schemas, environments, workspaces, application keys or IDs, filesystem paths, project/app prefixes, authentication settings, authorization mappings, or other typed project preferences. Those facts are project-specific and must still be selected and verified per project.

Use this shape, preserving unknown keys when safely editing an existing file:

```toml
global_config_version = 1

[[rules]]
name = "prefer_package_apis"
scope = "database" # all | database | apex | shared
description = "Prefer project-prefixed package APIs over standalone database routines."

[[rules]]
name = "document_security_exceptions"
scope = "all"
description = "Document the reason and approver for every security exception."
```

- A global rule has the same `name`, `scope`, and `description` contract as a project `[[rules]]` entry. Names must be unique within the library. Its scope may only be `all`, `database`, `apex`, or `shared`; it cannot name an application that may not exist in another project.
- Create, add, edit, or remove a global rule only when the user explicitly asks to manage their global rules; this can be done without a current project. Show the exact global-file change and obtain confirmation immediately before writing it. Do not create an empty global file merely because it is absent.
- When a user requests new-project setup and the library exists and validates, show its rule names, scopes, and descriptions. Let the user select **all**, an explicit named subset, or **none**. Copy only the selected entries into that project's `[[rules]]`; the global file is never linked or included at runtime.
- If the library is absent, mention that the user can create reusable global rules, but continue setup without blocking. If it is invalid or uses an unsupported version, report the problem and do not offer it until repaired; the project can still be set up without global rules.
- Applying global rules to an already configured project requires an explicit request. Do not re-prompt on ordinary project tasks. A later library change never updates any project automatically.
- Before copying, compare rule names. Skip an identical project rule and report it. For a same-name rule with different scope or description, show both and ask whether to keep the project rule, replace it with the selected global rule, or rename the global rule for this project; never overwrite it silently. Resolve any selected-rule conflict with a typed project preference before saving.
- Once copied, a global rule is an ordinary project rule. Editing or deleting it in one place has no effect on the other. The project config remains the complete record of rules governing that project.

## Project config contract

Use `config_version = 1`. Keep the file local and ensure project-root `.gitignore` contains exactly `/oracle_project_config.toml` (show the pending `.gitignore` edit and ask before writing it). Preserve unknown keys.

Use this compact shape; omit optional/deferred fields rather than inventing values:

```toml
config_version = 1

[project]
name = "operations" # default: project-root directory name; stable after setup

[skills]
db = "db"
apex = "apex"

[database]
project_mode = "new"                 # new | existing
sqlcl_connection = "dev_saved_alias" # raw SQLcl saved-connection alias
schema = "OPERATIONS_APP"                 # verify; never assume login/current/parsing schema agree
new_object_conventions = "project_rules" # project_rules | inherit_existing
object_source = "filesystem"         # filesystem | database

[database.naming]
shared_prefix = "operations"              # no trailing underscore

[database.keys]
default_strategy = "identity"         # identity | sequence_default | sequence_trigger

[database.audit]
created_by = "created_by"
created_at = "created"
modified_by = "modified_by"
modified_at = "modified"
strategy = "trigger_api"              # trusted PL/SQL API called by triggers

[database.plsql]
keyword_case = "upper"
# Naming patterns belong here when explicitly chosen.

[database.environments]
dev = "dev_saved_alias"
beta = "beta_saved_alias"             # optional, but verify before Beta work

[apex]
enabled = true
workspace = "OPERATIONS_WORKSPACE"                  # exact workspace name; verify before live APEX work

[apex.defaults]
authentication = "saml"               # recommended; may be none or another explicit choice
authorization = "custom"              # apex_acl | custom | none (warn for none)
report_default = "interactive_report"

[[apex.applications]]
key = "orders"
name = "Orders"
prefix = "operations_orders"                 # full app-owned object prefix
status = "new"                         # new | active
# path = "applications/operations_orders"     # omit until a new app is materialized
# application_id = 120                  # omit until assigned/verified
# sqlcl_connection = "other_alias"     # app override; verify before save
# authentication_policy = "preserve_existing" # explicit existing-app exception
# authorization = "custom"             # app override

[authorization]
# Set only after the user chooses a model. Do not infer identity semantics.
# canonical_identity = ":APP_USER"
# proxy_enabled = false

# [[rules]]
# name = "rule_name"
# scope = "database"                  # all | database | apex | shared | <application key>
# description = "User-approved project rule."
```

Use `database.sqlcl_connection` as the raw SQLcl saved alias. Tell users that a connection saved through the Oracle SQL Developer VS Code extension is also a SQLcl saved connection and can be referenced here.

For a new APEX app, omit `application_id` and `path`, set `status = "new"`, then write the verified assigned values after materialization/import. Validate uniqueness of every app key, prefix, path, and assigned ID.

## Initialize or repair

1. **Existing project config:** parse and validate it first. Invalid config must be repaired before project work. For an older but safely readable version, show a migration and ask before applying it. If the user asks to change a verifiable value, verify it first and show the evidence before saving.
2. **Personal rule selection:** for a new project, read and validate the optional global library, present its rules, and let the user select all, named rules, or none. Include the selected copied rules in the pending project-config diff. For an existing project, read or apply the library only when the user explicitly asks. A global-library failure must not prevent otherwise valid project setup.
3. **Identity and connections:** default `project.name` from the root folder but let the user override it. Ask for the SQLcl alias and test it read-only (`SELECT 1 FROM dual` plus resolved schema). Try the alias first. On a TNS error only: use `$TNS_ADMIN`, then a previously configured exact path, then ask for an exact path. Never search or guess TNS locations.
4. **Connection/schema overrides:** validate every app or environment override at least once. For existing projects, check representative configured/discovered objects too. Reject a wrong schema or missing expected objects; retain the prior verified value. New/empty schemas need connection/schema confirmation only.
5. **Dependencies:** default to configured `db` and `apex` names. Report unresolved dependencies during setup but finish setup. At task time, stop if the needed skill cannot be resolved and tell the user to install it or update `[skills]`. `/db` is required for DB work. APEX normally requires both `/apex` and `/db`; if the user explicitly accepts the warning to continue without `/db`, record that acknowledgement and do not nag again. Resume using `/db` automatically once it resolves.
6. **Database preferences:** ask whether the project is new or existing. For existing work, ask whether new objects inherit legacy conventions or use project rules; never change old objects without an explicit migration task. Offer a read-only convention scan only when the user wants it. Ask/confirm shared and app prefixes, PK strategy, audit names, and PL/SQL style. Suggest lowercase snake-case prefixes from the project/app names but never choose them silently.
7. **Optional decisions:** absence means dependency defaults. Ask all relevant setup questions, but allow the user to defer nonessential preferences. Add a named scoped project rule only when the user asks to persist it; add a global rule only when they explicitly ask to manage their personal global library. If any project or selected global rule conflicts with a typed preference, identify the conflict, ask which wins, repair the project config, then continue.
8. **Filesystem records:** explicitly ask whether to keep DB objects on disk. Do not pick a default. If enabled, check for Git and offer `git init` when absent; never initialize Git silently. If disabled, DB changes still proceed normally but no deployable DB artifact exists.
9. **APEX branch:** if disabled, do not request `/apex`; DB work remains available. If enabled, collect the exact workspace and verify it before live work. Require APEX 26.1+ / APEXlang before APEX work; pending validation may finish setup but blocks APEX tasks. Older APEX is outside OPC APEX scope: disable APEX for DB-only usage or use another workflow.

## APEXlang projects

- One project has one workspace and zero or more apps. Require an app key/name target when a request is not unambiguous; never default to the first app. `shared` is a valid DB ownership target.
- In multi-app projects, create new app sources at `applications/<full-prefix>/`, e.g. `applications/operations_orders/`. Do not force this layout on a single-app project. Preserve a first app’s recorded path when later adding apps; never relocate it automatically.
- For new apps, direct `/apex` to materialize the appropriate structure, then record the resolved relative path and verified assigned ID. For an existing ID change, query the workspace, show the returned app name, and require reconfirmation. If app/path resolution is genuinely ambiguous, ask and record the answer.
- `/apex` determines its own offline/live workflow. For interactive DB-backed work, retain its explicit Offline/Live DB choice; recommend Live DB only when the stored connection and workspace are verified.
- This skill supports **APEXlang only**. Do not create or manage legacy root `f<ID>.sql`, `Deployments/`, `Deployment.txt`, or APEX archive folders. Git tracks APEXlang source. Legacy app-ID SQL releases are manual and outside OPC until this skill is updated.

### UI, authentication, and authorization

- Shared project defaults may have app overrides. Prefer Interactive Reports workspace-wide. Use an Interactive Grid only when the user explicitly asks or approves a clearly necessary grid.
- Recommend SAML. Store only scheme type/name/source references. A source may be a managed app key or an existing app ID in the workspace. Prefer supported APEX shared-component subscription; `/apex` must verify what can subscribe and use an approved copy only when needed.
- Inspect existing app authentication. If it is not SAML, explain the preference and ask once whether to migrate. On decline, save `preserve_existing` for that app and do not ask again unless the user requests a change. Any migration needs a plan, rollback/access check, and confirmation immediately before mutation.
- Authorization choices are APEX ACL, custom, or none. Warn and require confirmation for none. Default custom authorization is shared/project-owned; an app may extend it or use an explicit independent model.
- Require an exact canonical identity mapping (for example, `:APP_USER` to a named view column). Never assume emails, employee IDs, case mapping, claims, joins, or row predicates.
- Organization access is opt-in and user-defined. Record schema-qualified source tables/views, columns and their purposes, plus the explicit policy rule—no separate connection alias. Validate with zero-row/read-only queries through the app connection. On grant/reference failure mark it pending, warn, and continue unrelated setup; recheck when asked or when org-dependent work begins.
- A local user-role override is an intentional access grant and may bypass external active/org checks for special roles. Make that bypass explicit. Put custom/org/proxy logic in a shared project-prefixed PL/SQL authorization API called by APEX authorization schemes; do not duplicate security SQL in pages.
- Proxy is disabled by default. If enabled, require local override/custom authorization. A super-admin grants a proxy user access to act as a target user, time-bound or indefinite. Authorization uses target access while audit retains both identities. New writable tables owned by a proxy-enabled app—and shared tables it can write—must include configured standard audit fields plus nullable `created_by_proxy` and `modified_by_proxy`. The normal audit fields identify the effective user; proxy fields identify the real actor and are NULL when not proxying. Get both only from trusted server-side context/API.

## Database naming and audit

- Store prefixes without a trailing `_`; compose names with it. Use the shared prefix for shared DB objects and the full app prefix for app-owned tables, views, packages, sequences, triggers, named indexes/constraints, and types.
- Standard filenames/object role suffixes: sequences `_seq`; triggers `<event>_trg`; packages `<role>_pkg`. The user may explicitly override an individual name. Prefer packages over standalone functions/procedures unless the user requests standalone code.
- The four configured audit columns are mandatory for new relevant tables. Implement trusted auditing through a project PL/SQL API invoked by DB triggers, not client-controlled page items. Existing tables change only through explicit user-approved migration/adoption.
- The configured PK strategy is the default. A table-specific exception requires an explicit request and documented reason.

## Filesystem database mode

When `database.object_source = "filesystem"`, create and track this layout on demand:

```text
database/
  install.sql
  tables/       sequences/  views/       triggers/
  packages/     functions/  procedures/  types/
  indexes/      data/
```

Use `.sql` for tables, sequences, views, triggers, indexes, and data; `.pks/.pkb` for package spec/body; `.fnc`, `.prc`, and `.tps/.tpb` for standalone functions, procedures, and type spec/body.

- `database/install.sql` is the only ordered deployment manifest. It is checked in, idempotent, and includes the files required by the managed scope. Keep it deployment-correct automatically; update it when objects change and ask only when dependency order cannot be determined safely.
- A requested DB change normally: update the typed source file, update `install.sql`, apply/compile the affected file(s) in verified DEV, then verify the result. “Files only”, “do not compile”, or “do not add to DEV” suppresses the live DEV step for that request.
- Treat filesystem definitions as authoritative for managed objects. Flag live drift and offer explicit reconciliation; do not silently synchronize either direction.
- For an existing schema, ask whether to baseline objects. If declined, leave the tree empty and add files only for later user-approved changes. The first change to an unmanaged object adopts it into filesystem management. New projects support clean install; a legacy project that skipped baseline has an idempotent upgrade script that assumes its legacy baseline exists. Pull an adopted object’s DDL only when needed to make its file self-contained.
- Object files must be idempotent: first run creates what is absent; later runs safely add/replace needed state. Destructive or transforming work may stay in this path, but show exact impact and obtain explicit user approval before writing it. Do not add runtime SQLcl confirmation guards that burden DBA execution.
- Do not run the complete manifest for routine DEV edits. Run it only when the user explicitly requests a DEV run or a **Beta test deployment**. A full APEX Beta deployment is DB first, then `/apex` validation/import for the selected apps.
- After successful Beta DB deployment, record the tested Git commit mechanically in ignored config. Before APEX-only Beta promotion, compare `database/` changes since that commit and recommend combined deployment if present. If filesystem records are disabled, use only DB work known in the current conversation; otherwise warn that DB promotion cannot be determined and that the user must request/promote it manually.
- Production is a DBA handoff: provide the approved commit’s complete `database/` tree and `install.sql`. Create a ZIP only on explicit request. OPC does not run production deployment.

## Mutation gates and completion

- Show a plan and get explicit confirmation immediately before authentication migration, destructive/data-transforming DB changes, enabling public/no-role access, creating proxy behavior, writing a project config preference/security rule, copying selected global rules into a project, or changing the personal global rule library.
- Mechanical verified updates—assigned APEX IDs/paths, verification state, and last successful Beta commit—may be written after the user-requested operation and reported exactly.
- If facts required for a high-impact decision cannot be validated, stop that operation and ask for the missing input. Do not guess.
- On completion, report changed files, live actions taken, validations performed, global rules created or copied (and rules skipped or conflicted), pending warnings (missing skills/grants/verification), and any next explicit action required.
