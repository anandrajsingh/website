---
title: Manage roles
enableTableOfContents: true
isDraft: false
redirectFrom:
  - /docs/manage/users
---

In Neon, roles are Postgres roles. Each Neon project is created with a default role that takes its name from your Neon account (the Google, GitHub, or partner account that you registered with). This role owns the default database (`neondb`) that is created in your Neon project's primary branch.

Your default role and roles created in the Neon console, API, and CLI are granted membership in the [neon_superuser](#the-neon_superuser-role) role. Roles created with SQL are only granted basic privileges, as you would see in a stand-alone Postgres installation.

<Admonition type="note">
Neon is a managed Postgres service, so you cannot access the host operating system, and you can't connect using the Postgres `superuser` account like you can in a stand-alone Postgres installation.
</Admonition>

You can create roles in a project's primary branch or child branches. There is no limit to the number of roles you can create.

In Neon, roles belong to a branch, which could be your main branch or a child branch. When you create a child branch, roles in the parent branch are duplicated in the child branch. For example, if role `sally` exists in the parent branch, role `sally` is copied to the child branch when the child branch is created. The only time this does not occur is when you create a branch that only includes data up to a particular point in time. If the role was created in the parent branch after that point in time, it is not duplicated in the child branch.

Neon supports creating and managing roles from the following interfaces:

- [Neon console](#manage-roles-in-the-neon-console)
- [Neon CLI](#manage-roles-with-the-neon-cli)
- [Neon API](#manage-roles-with-the-neon-api)
- [SQL](#manage-roles-with-sql)

## The neon_superuser role

Roles created in the Neon console, CLI, or API, including the default role created with a Neon project, are granted membership in the `neon_superuser` role. Users cannot login as `neon_superuser`, but they inherit the privileges assigned to this role. The privileges and predefined role memberships granted to `neon_superuser` are shown in this `CREATE ROLE` statement:

<CodeBlock shouldWrap>

```sql
CREATE ROLE neon_superuser CREATEDB CREATEROLE BYPASSRLS NOLOGIN IN ROLE pg_read_all_data, pg_write_all_data;
```

</CodeBlock>

- `CREATEDB`: Provides the ability to create databases.
- `CREATEROLE`: Provides the ability to create new roles (which also means it can alter and drop roles).
- `BYPASSRLS`: Provides the ability to bypass row-level security (RLS) policies. This attribute is only included in `neon_superuser` roles in projects created after the [August 15, 2023 release](/docs/release-notes/2023-08-15-storage-and-compute).
- `NOLOGIN`: The role cannot be used to log in to the Postgres server. Neon is a managed Postgres service, so you cannot access the host operating system.
- `pg_read_all_data role`: A predefined role in Postgres that provides the ability to select from all tables and views.
- `pg_write_all_data`: A predefined role in Postgres that provides the ability to insert, update, and delete in all tables and use all sequences in a database.

In addition, the `neon_superuser` role is able to add [Postgres extensions](/docs/extensions/pg-extensions) that are available for use with Neon.

You can think of roles with `neon_superuser` privileges as administrators. For all other users, you can create roles and manage database object access privileges with SQL. See [Manage roles with SQL](#manage-roles-with-sql).

## Manage roles in the Neon console

This section describes how to create, view, and delete roles in the Neon Console. All roles created in the Neon console are granted membership in the [neon_superuser](#the-neon_superuser-role) role, which is intended for users of the Neon console.

### Create a role

To create a role:

1. Navigate to the [Neon Console](https://console.neon.tech).
2. Select a project.
3. Select **Roles**.
4. Select the branch where you want to create the role.
4. Click **New Role**.
5. In the role creation dialog, specify a role name. The length of the role name is limited to 63 bytes.
6. Click **Create**. The role is created and you are provided with the password for the role.

### Delete a role

Deleting a role is a permanent action that cannot be undone, and you cannot delete a role that owns a database. The database must be deleted before you can deleting the role that owns the database.

To delete a role:

1. Navigate to the [Neon Console](https://console.neon.tech).
2. Select a project.
3. Select **Roles**.
4. Select a branch to view roles in the branch.
5. Click the delete icon for the role you want to delete.
6. On the delete role dialog, click **Delete**.

### Reset a password

To reset a role's password:

1. Navigate to the [Neon Console](https://console.neon.tech).
2. Select a project.
3. Select **Roles**.
4. Select a branch to view roles in the branch.
5. Select **Reset password**.
6. On the confirmation dialog, click **Sure, reset**. A reset password dialog is displayed with your new password.

## Manage roles with the Neon CLI

The Neon CLI supports creating and deleting roles. For instructions, see [Neon CLI commands — roles](/docs/reference/cli-roles). Roles created with the Neon CLI are granted membership in the [neon_superuser](#the-neon_superuser-role) role.

## Manage roles with the Neon API

Role actions performed in the Neon Console can also be performed using Neon API role methods. The following examples demonstrate how to create, view, reset passwords for, and delete roles using the Neon API. For other role-related methods, refer to the [Neon API reference](https://api-docs.neon.tech/reference/getting-started-with-neon-api).

Roles created with the Neon API are granted membership in the [neon_superuser](#the-neon_superuser-role) role.

In Neon, roles belong to branches, which means that when you create a role, it is created in a branch. Role-related requests are therefore performed using branch API methods.

<Admonition type="note">
The API examples that follow may not show all user-configurable request body attributes that are available to you. To view all  attributes for a particular method, refer to method's request body schema in the [Neon API reference](https://api-docs.neon.tech/reference/getting-started-with-neon-api).
</Admonition>

The `jq` option specified in each example is an optional third-party tool that formats the `JSON` response, making it easier to read. For information about this utility, see [jq](https://stedolan.github.io/jq/).

### Prerequisites

A Neon API request requires an API key. For information about obtaining an API key, see [Create an API key](/docs/manage/api-keys#create-an-api-key). In the cURL examples shown below, `$NEON_API_KEY` is specified in place of an actual API key, which you must provide when making a Neon API request.

### Create a role with the API

The following Neon API method creates a role. To view the API documentation for this method, refer to the [Neon API reference](https://api-docs.neon.tech/reference/createprojectbranchrole).

```text
POST /projects/{project_id}/branches/{branch_id}/roles
```

The API method appears as follows when specified in a cURL command. The `project_id` and `branch_id` are required parameters, and the role `name` is a required attribute. The length of a role name is limited to 63 bytes.

```bash
curl 'https://console.neon.tech/api/v2/projects/hidden-cell-763301/branches/br-blue-tooth-671580/roles' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer $NEON_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
  "role": {
    "name": "sally"
  }
}' | jq
```

Response:

```json
{
  "role": {
    "branch_id": "br-blue-tooth-671580",
    "name": "sally",
    "password": "FLfASd8mbKO9",
    "protected": false,
    "created_at": "2023-01-04T20:35:48Z",
    "updated_at": "2023-01-04T20:35:48Z"
  },
  "operations": [
    {
      "id": "b4fc0c92-8a4f-4a1a-9891-fd36155de853",
      "project_id": "hidden-cell-763301",
      "branch_id": "br-blue-tooth-671580",
      "endpoint_id": "ep-aged-math-668285",
      "action": "apply_config",
      "status": "running",
      "failures_count": 0,
      "created_at": "2023-01-04T20:35:48Z",
      "updated_at": "2023-01-04T20:35:48Z"
    },
    {
      "id": "74fef831-7537-4d78-bb87-222e0918df54",
      "project_id": "hidden-cell-763301",
      "branch_id": "br-blue-tooth-671580",
      "endpoint_id": "ep-aged-math-668285",
      "action": "suspend_compute",
      "status": "scheduling",
      "failures_count": 0,
      "created_at": "2023-01-04T20:35:48Z",
      "updated_at": "2023-01-04T20:35:48Z"
    }
  ]
}
```

### List roles with the API

The following Neon API method lists roles for the specified branch. To view the API documentation for this method, refer to the [Neon API reference](https://api-docs.neon.tech/reference/listprojectbranchroles).

```text
GET /projects/{project_id}/branches/{branch_id}/roles
```

The API method appears as follows when specified in a cURL command. The `project_id` and `branch_id` are required parameters.

```bash
curl 'https://console.neon.tech/api/v2/projects/hidden-cell-763301/branches/br-blue-tooth-671580/roles' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer $NEON_API_KEY' | jq
```

Response:

```json
{
  "roles": [
    {
      "branch_id": "br-blue-tooth-671580",
      "name": "daniel",
      "protected": false,
      "created_at": "2023-07-09T17:01:34Z",
      "updated_at": "2023-07-09T17:01:34Z"
    },
    {
      "branch_id": "br-blue-tooth-671580",
      "name": "sally",
      "protected": false,
      "created_at": "2023-07-13T06:42:55Z",
      "updated_at": "2023-07-13T14:48:29Z"
    }
  ]
}
```

### Reset a password with the API

The following Neon API method resets the password for the specified role. To view the API documentation for this method, refer to the [Neon API reference](https://api-docs.neon.tech/reference/resetprojectbranchrolepassword).

```text
POST /projects/{project_id}/branches/{branch_id}/roles/{role_name}/reset_password
```

The API method appears as follows when specified in a cURL command. The `project_id`, `branch_id`, and `role_name` are required parameters.

```bash
curl -X 'POST' \
  'https://console.neon.tech/api/v2/projects/hidden-cell-763301/branches/br-blue-tooth-671580/roles/sally/reset_password' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer $NEON_API_KEY' | jq
```

Response:

```json
{
  "role": {
    "branch_id": "br-blue-tooth-671580",
    "name": "sally",
    "password": "sFA4k3pESzVA",
    "protected": false,
    "created_at": "2023-01-04T20:35:48Z",
    "updated_at": "2023-01-04T20:38:49Z"
  },
  "operations": [
    {
      "id": "d319b791-96c7-48b4-8683-f127839dfb99",
      "project_id": "hidden-cell-763301",
      "branch_id": "br-blue-tooth-671580",
      "endpoint_id": "ep-aged-math-668285",
      "action": "apply_config",
      "status": "running",
      "failures_count": 0,
      "created_at": "2023-01-04T20:38:49Z",
      "updated_at": "2023-01-04T20:38:49Z"
    },
    {
      "id": "7bd5bb24-92e1-49d1-a3f4-c02e5b6d11e4",
      "project_id": "hidden-cell-763301",
      "branch_id": "br-blue-tooth-671580",
      "endpoint_id": "ep-aged-math-668285",
      "action": "suspend_compute",
      "status": "scheduling",
      "failures_count": 0,
      "created_at": "2023-01-04T20:38:49Z",
      "updated_at": "2023-01-04T20:38:49Z"
    }
  ]
}
```

### Delete a role with the API

The following Neon API method deletes the specified role. To view the API documentation for this method, refer to the [Neon API reference](https://api-docs.neon.tech/reference/deleteprojectbranchrole).

```text
DELETE /projects/{project_id}/branches/{branch_id}/roles/{role_name}
```

The API method appears as follows when specified in a cURL command. The `project_id`, `branch_id`, and `role_name` are required parameters.

```bash
curl -X 'DELETE' \
  'https://console.neon.tech/api/v2/projects/hidden-cell-763301/branches/br-blue-tooth-671580/roles/sally' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer $NEON_API_KEY' | jq
```

Response:

```json
{
  "role": {
    "branch_id": "br-blue-tooth-671580",
    "name": "sally",
    "protected": false,
    "created_at": "2023-01-04T20:35:48Z",
    "updated_at": "2023-01-04T20:38:49Z"
  },
  "operations": [
    {
      "id": "0789c601-9d97-4124-80df-cd7b332f11ef",
      "project_id": "hidden-cell-763301",
      "branch_id": "br-blue-tooth-671580",
      "endpoint_id": "ep-aged-math-668285",
      "action": "apply_config",
      "status": "running",
      "failures_count": 0,
      "created_at": "2023-01-04T20:40:27Z",
      "updated_at": "2023-01-04T20:40:27Z"
    },
    {
      "id": "d3b79f02-f369-4ad0-8bf5-ff0daf27dd9a",
      "project_id": "hidden-cell-763301",
      "branch_id": "br-blue-tooth-671580",
      "endpoint_id": "ep-aged-math-668285",
      "action": "suspend_compute",
      "status": "scheduling",
      "failures_count": 0,
      "created_at": "2023-01-04T20:40:27Z",
      "updated_at": "2023-01-04T20:40:27Z"
    }
  ]
}
```

## Manage roles with SQL

Roles created with SQL have the same privileges as newly created roles in a stand-alone Postgres installation. These roles are not granted membership in the [neon_superuser](#the-neon_superuser-role) role like roles created with the Neon Console, CLI, or API. You must grant these roles the privileges you want them to have.

To create a role with SQL, issue a `CREATE ROLE` statement from a client such as [psql](/docs/connect/query-with-psql-editor) or from the [Neon SQL Editor](/docs/get-started-with-neon/query-with-neon-sql-editor).

```sql
CREATE ROLE <name> WITH LOGIN PASSWORD 'password';
```

- `WITH LOGIN` means that the role will have a login privilege, required for the role to log in to your Neon Postgres instance. If the role is used only for privilege management, the `WITH LOGIN` privilege is unnecessary.
- A password is required and must have a minimum entropy of 60 bits.

    <Admonition type="info">  
    To create a password with 60 bits of entropy, you can follow these password composition guidelines:
    - **Length**: The password should consist of at least 12 characters.
    - **Character diversity**: To enhance complexity, passwords should include a variety of character types, specifically:
      - Lowercase letters (a-z)
      - Uppercase letters (A-Z)
      - Numbers (0-9)
      - Special symbols (e.g., !@#$%^&*)
    - **Avoid predictability**: To maintain a high level of unpredictability, do not use:
      - Sequential patterns (such as '1234', 'abcd', 'qwerty')
      - Common words or phrases
      - Any words found in a dictionary
      - **Avoid character repetition**: To maximize randomness, do not use the same character more than twice consecutively.

    Example password: `T3sting!23Ab` (DO NOT USE THIS EXAMPLE PASSWORD)

    Passwords must be supplied in plain text but are encrypted when stored. Hashed passwords are not supported.

    The guidelines should help you create a password with approximately 60 bits of entropy. However, depending on the exact characters used, the actual entropy might vary slightly. Always aim for a longer and more complex password if you're uncertain. It's also recommended to use a trusted password manager to create and store your complex passwords safely.
    </Admonition>

For role creation and access management examples, refer to the [Manage roles and database access with SQL](/docs/guides/manage-database-access) guide.

## Need help?

Send a request to [support@neon.tech](mailto:support@neon.tech), or join the [Neon community forum](https://community.neon.tech/).
