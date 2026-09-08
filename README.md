# Oracle Preference Controller

**Oracle Preference Controller (OPC)** is a Codex plugin for setting up and governing Oracle Database projects with optional APEXlang applications. It stores project preferences locally, can copy selected reusable personal rules into new projects, applies those preferences before delegating work to Oracle DB/APEX skills, and keeps database/APEX conventions consistent.

## Install

Requires the Codex CLI.

```bash
codex plugin marketplace add xamdxlonewolf/oracle-preference-controller --ref main
codex plugin add oracle-preference-controller@oracle-preference-controller
```

Start a new Codex thread after installation so the skill is available.

## Use

Run `/opc` from the project root, or from any nested directory. OPC finds the nearest project configuration by walking upward.

On first use, OPC creates and configures a local `oracle_project_config.toml`. It asks about:

- SQLcl saved connection and database conventions
- New versus existing schema behavior
- Optional filesystem/Git tracking for database objects
- Optional APEXlang workspace and application configuration
- Authentication, authorization, organization access, proxying, and UI preferences

The local config is added to `.gitignore`; it must not contain passwords, tokens, certificates, or other secrets.

### Example requests

```text
/opc set up this Oracle Database project
/opc create a table for work orders
/opc create an APEXlang Orders application
/opc add an Interactive Report to the Orders app
/opc test the combined database and APEX deployment in Beta
```

OPC asks for an application target whenever a multi-app project request is ambiguous. Project rules are persisted only when you explicitly ask OPC to save or change a rule.

### Reusable personal global rules

OPC can maintain an optional personal global rule library at:

```text
$CODEX_HOME/oracle-preference-controller/global_config.toml
# or ~/.codex/oracle-preference-controller/global_config.toml when CODEX_HOME is unset
```

It contains reusable `[[rules]]` templates—not a second project config. During new-project setup, OPC shows the library and lets you select all, a named subset, or none. Selected rules are copied into that project's `oracle_project_config.toml`; they are never silently applied, linked, or retroactively synchronized.

For example, ask OPC to “save this as a global rule” or “apply my global rules to this project.” It will show the exact change and ask before writing either config. The global library must not contain project-specific details such as connection aliases, schemas, workspaces, app IDs/keys, prefixes, paths, or secrets; those still require project-level selection and verification.

## Dependencies and scope

OPC uses configurable `/db` and `/apex` skills. Setup can finish if either is not installed, but OPC stops a task that needs an unresolved dependency and explains how to install or override its configured name.

- Database-only projects are supported.
- APEX support is APEXlang-only and requires APEX 26.1 or later.
- Legacy APEX `f<ID>.sql` export/release workflows are intentionally out of scope.

## Database source files

If you choose filesystem tracking during setup, OPC manages a versioned `database/` tree with an idempotent `database/install.sql` deployment manifest. Routine work updates and applies only changed object files to DEV. The full manifest runs only when explicitly requested, typically for a Beta deployment test before DBA production handoff.

If filesystem tracking is disabled, OPC can still make DEV database changes, but it has no reproducible database promotion artifact; promote database work manually when needed.

## Update

After a newer plugin release is published:

```bash
codex plugin marketplace upgrade oracle-preference-controller
codex plugin add oracle-preference-controller@oracle-preference-controller
```

Start a new thread after reinstalling.
