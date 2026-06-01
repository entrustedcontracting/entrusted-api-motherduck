# Entrusted Data Warehouse — REST API

A read-only API to query Entrusted's data warehouse. You can list tables, inspect columns, and run SQL queries against three datasets: **Leads**, **Opportunities**, and **Services**.

All data endpoints require a Bearer token. You'll receive your token from an Entrusted admin.

---

## Base URL

```
https://api-server-production-0c68.up.railway.app
```

---

## Authentication

Include your token in every request as an `Authorization` header:

```
Authorization: Bearer <your-token>
```

Requests without a valid token will receive a `401 Unauthorized` response.

---

## Endpoints

### Health check

```
GET /health
```

No authentication required. Returns `{"status": "ok"}` if the API is running.

---

### List tables

```
GET /tables
```

Returns the available tables you can query.

**Example:**

```bash
curl https://api-server-production-0c68.up.railway.app/tables \
  -H "Authorization: Bearer <your-token>"
```

**Response:**

```json
[
  { "name": "leads", "fully_qualified": "entrusted_dw.semantic.Leads" },
  { "name": "opportunities", "fully_qualified": "entrusted_dw.semantic.opportunities" },
  { "name": "services", "fully_qualified": "entrusted_dw.semantic.services" }
]
```

---

### Describe a table

```
GET /tables/{table_name}
```

Returns the column names and types for a given table.

**Example:**

```bash
curl https://api-server-production-0c68.up.railway.app/tables/leads \
  -H "Authorization: Bearer <your-token>"
```

---

### Run a query

```
POST /query
```

Execute a read-only SQL `SELECT` query. Only the three tables listed above can be referenced — any attempt to access other tables will be rejected.

**Request body (JSON):**

| Field | Type | Description |
|-------|------|-------------|
| `sql` | string | A SQL `SELECT` query |

**Example:**

```bash
curl -X POST https://api-server-production-0c68.up.railway.app/query \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{"sql": "SELECT customer_name, branch, booked_date FROM leads LIMIT 5"}'
```

**Response:**

```json
{
  "columns": ["customer_name", "branch", "booked_date"],
  "row_count": 5,
  "data": [
    { "customer_name": "Jane Doe", "branch": "HOU", "booked_date": "2024-03-15" },
    ...
  ]
}
```

---

## Quick reference

| What you want to do | Request |
|----------------------|---------|
| Check if the API is up | `GET /health` |
| See which tables exist | `GET /tables` |
| See columns for a table | `GET /tables/opportunities` |
| Query data | `POST /query` with `{"sql": "SELECT ..."}` |

---

## Good to know

- **Read-only** — only `SELECT` queries are allowed. You cannot modify any data.
- **Max 1,024 rows** — queries return at most 1,024 rows per request. Use `LIMIT` and `OFFSET` for pagination.
- **SQL dialect** — the database uses [DuckDB SQL](https://duckdb.org/docs/sql/introduction). Standard SQL works for most queries.
- **Tables can be referenced by short name** — use `leads`, `opportunities`, or `services` directly in your queries.
