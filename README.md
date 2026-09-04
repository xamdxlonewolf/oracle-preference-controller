# Oracle Preference Controller

**Oracle Preference Controller (OPC)** is a Codex plugin for setting up and governing Oracle Database projects with optional APEXlang applications. It stores project preferences locally, applies them before delegating work to Oracle DB/APEX skills, and keeps database/Apex conventions consistent.

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
