---
title: Connect a GUI application
subtitle: Learn how to connect a GUI application to Neon
enableTableOfContents: true
---

This topic describes how to connect to a Neon database from a GUI application or IDE. Most GUI applications and IDEs that support connecting to a Postgres database also support connecting to Neon.

## Gather your connection details

The following details are typically required when configuring a connection:

- hostname
- port
- database name
- role (user)
- password

You can gather these details from the **Connection Details** widget on the **Neon Dashboard**. Select a branch, a role, and the database you want to connect to. A connection string is constructed for you.

![Connection details widget](/docs/connect/connection_details.png)

<Admonition type="note">
Neon supports pooled and direct connections to the database. Use a pooled connection string if your application uses a high number of concurrent connections. For more information, see [Connection pooling](/docs/connect/connection-pooling#connection-pooling).
</Admonition>

The connection string includes the role, password, hostname, and database name.

```text
postgres://daniel:<password>@ep-raspy-cherry-95040071.us-east-2.aws.neon.tech/neondb
             ^                                 ^                                ^
             |- <role>                         |- <hostname>                    |- <database>
```

- role: `daniel`
- hostname: `ep-raspy-cherry-95040071.us-east-2.aws.neon.tech`
- database name: `neondb`

Neon uses the default Postgres port, `5432`.

## Connect to the database

In the GUI application or IDE, enter the connection details into the appropriate fields and connect. Some applications permit specifying a connection string while others require entering connection details into separate fields. In the pgAdmin example below, connection details are entered into separate fields, and clicking **Save** establishes the database connection.

![Register - Server](/docs/connect/pgadmin4.png)

Some Java-based tools that use the pgJDBC driver for connecting to Postgres, such as DBeaver, DataGrip, and CLion, do not support including a role name and password in a database connection string or URL field. When you find that a connection string is not accepted, try entering the database name, role, and password values in the appropriate fields in the tool's connection UI when configuring a connection to Neon. For example, the DBeaver client has a **URL** field, but connecting to Neon requires specifying the connection details as shown:

![DBeaver connection](/docs/connect/dbeaver_connection.png)

## Tested GUI applications and IDEs

Connections from the GUI applications and IDEs in the table below have been tested with Neon.

<Admonition type="note">
Some applications require an Server Name Indication (SNI) workaround. Neon uses compute endpoint domain names to route incoming connections. However, the Postgres wire protocol does not transfer the server domain name, so Neon relies on the Server Name Indication (SNI) extension of the TLS protocol to do this. Not all application clients support SNI. In these cases, a workaround is required. For more information, see [Connection errors](/docs/connect/connection-errors).
</Admonition>

| Application or IDE | Notes |
| ----------- | ----- |
| [Appsmith](https://www.appsmith.com/) | |
| [AWS Database Migration Service (DMS)](https://aws.amazon.com/dms/)| Use [SNI workaround D](/docs/connect/connection-errors#d-specify-the-endpoint-id-in-the-password-field). Use a `$` character as a separator between the `endpoint` option and password. For example: `endpoint=<endpoint_id>$<password>`. Also, you must set **Secure Socket Layer (SSL) mode** to `require`. See [Migrate with AWS DMS](/docs/import/migrate-aws-dms).
| [Azure Data Studio](https://azure.microsoft.com/en-us/products/data-studio/) | Requires the [PostgreSQL extension](https://learn.microsoft.com/en-us/sql/azure-data-studio/extensions/postgres-extension?view=sql-server-ver16) and [SNI workaround D](/docs/connect/connection-errors#d-specify-the-endpoint-id-in-the-password-field) |
| [Beekeeper Studio](https://www.beekeeperstudio.io/) | Requires the **Enable SSL** option |
| [CLion](https://www.jetbrains.com/clion/) | |
| [Datagran](https://www.datagran.io/) | Requires [SNI workaround D](/docs/connect/connection-errors#d-specify-the-endpoint-id-in-the-password-field) connection workaround |
| [DataGrip](https://www.jetbrains.com/datagrip/) | |
| [DBeaver](https://dbeaver.io/) | |
| [dbForge](https://www.devart.com/dbforge/) | |
| [DbVisualizer](https://www.dbvis.com/) | |
| [DronaHQ hosted cloud version](https://www.dronahq.com/) | Requires selecting **Connect using SSL** when creating a connector |
| [Forest Admin](https://www.forestadmin.com/) | The database requires at least one table |
| [Grafana](https://grafana.com/docs/grafana/latest/datasources/postgres/)| Requires `sslmode=verify-full`. See [SNI workaround C](/docs/connect/connection-errors#c-set-verify-full-for-golang-based-clients). |
| [Google Looker Studio](https://lookerstudio.google.com/) | Requires **Enable SSL** and uploading the [ISRG Root X1](https://letsencrypt.org/certificates/) self-signed `pem` root certificate from Let's Encrypt. See [Connect to Looker Studio](https://community.neon.tech/t/connect-to-data-studio-looker-studio/299/3), in the _Neon Community_ forum. |
| [ILLA Cloud](https://www.illacloud.com/) | |
| [Luna Modeler](https://www.datensen.com/data-modeling/luna-modeler-for-relational-databases.html) | Requires enabling the SSL/TLS option |
| [Metabase](https://www.metabase.com/) | |
| [PopSQL](https://popsql.com/) | No SNI support. Use [SNI workaround D](/docs/connect/connection-errors#d-specify-the-endpoint-id-in-the-password-field). |
| [Postico](https://eggerapps.at/postico2/) | SNI support since v1.5.21. For older versions, use [SNI workaround B](/docs/connect/connection-errors#b-use-libpq-keyvalue-syntax-in-the-database-field). |
| [PostgreSQL VS Code Extension by Chris Kolkman](https://marketplace.visualstudio.com/items?itemName=ckolkman.vscode-postgres) | |
| [pgAdmin 4](https://www.pgadmin.org/) | |
| [Retool](https://retool.com/) | |
| [TablePlus](https://tableplus.com/) | SNI support on macOS since build 436, and on Windows since build 202. No SNI support on Linux currently. For older versions, use [SNI workaround B](/docs/connect/connection-errors#b-use-libpq-keyvalue-syntax-in-the-database-field). |
| [Segment](https://segment.com/) | Requires [SNI workaround D](/docs/connect/connection-errors#d-specify-the-endpoint-id-in-the-password-field) |

## Connection issues

Applications that use older client libraries or drivers that do not support Server Name Indication (SNI) may not permit connecting to Neon. If you encounter the following error, refer to [Connection errors](/docs/connect/connection-errors) for possible workarounds.

<CodeBlock shouldWrap>

```txt
ERROR: The endpoint ID is not specified. Either upgrade the Postgres client library (libpq) for SNI support or pass the endpoint ID (the first part of the domain name) as a parameter: '&options=endpoint%3D'. See [https://neon.tech/sni](https://neon.tech/sni) for more information.
```

</CodeBlock>

## Need help?

Send a request to [support@neon.tech](mailto:support@neon.tech), or join the [Neon community forum](https://community.neon.tech/).
