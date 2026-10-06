# Sysadmin Notes — Service Account & SPN Setup

## Creating a Service Account in Active Directory

**Service Principal Name (SPN)** — tells Kerberos that a specific service running on a specific machine is represented by a particular account.

**`setspn`** — the CLI tool used to register a Service Principal Name to a Service Account.
- `-S` flag checks for SPN duplicates
- `-A` flag does not

Registering an SPN is what makes an account kerberoastable, since it allows any authenticated user to request a TGS ticket for it.

**Service account:** `svc_sql` — created via ADUC with a weak password (`CHICKEN123!`) for lab purposes.

**Ran**: `setspn -S MSSQLSvc/svc_sql.lab.local LAB\svc_sql`


I am not actually running any MSSQL service — it isn't needed to perform Kerberoasting. The focus here is purely on the service account and its password.
