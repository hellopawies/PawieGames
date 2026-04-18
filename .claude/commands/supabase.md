# Supabase Skill

Connect to the PawieGames Supabase project and perform the requested operation using `curl`.

## Project constants (hardcoded — never change these)
- **URL**: `https://fwawlsbjltgvcpyezciw.supabase.co`
- **Project ref**: `fwawlsbjltgvcpyezciw`
- **Anon key**: `sb_publishable_cWfZRmHb55YMA-wa7hb6rw_qP6dIuYi`

## Credentials (from environment)

Before running any operation, check which credentials are available:

```bash
echo "Service key set: ${SUPABASE_SERVICE_KEY:+yes}"
echo "Mgmt token set:  ${SUPABASE_MGMT_TOKEN:+yes}"
```

- `SUPABASE_SERVICE_KEY` — service role key. Bypasses RLS. Required for direct table access.
- `SUPABASE_MGMT_TOKEN` — personal access token from supabase.com/dashboard/account/tokens. Required for raw SQL execution.

If a required credential is missing, tell the user exactly which one to add and how:
> "Add it to this project's env vars: Settings → Environment in Claude Code, or run `claude config set env.SUPABASE_SERVICE_KEY=<key>`"

---

## Operations

### 1. Execute raw SQL (requires SUPABASE_MGMT_TOKEN)

Use this for DDL, deletes across schemas, or anything that needs full SQL power.

```bash
curl -s -X POST "https://api.supabase.com/v1/projects/fwawlsbjltgvcpyezciw/database/query" \
  -H "Authorization: Bearer $SUPABASE_MGMT_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"query\": \"<SQL HERE>\"}" | jq .
```

### 2. Call an RPC function (works with service key or anon key)

```bash
curl -s -X POST "https://fwawlsbjltgvcpyezciw.supabase.co/rest/v1/rpc/<function_name>" \
  -H "apikey: ${SUPABASE_SERVICE_KEY:-sb_publishable_cWfZRmHb55YMA-wa7hb6rw_qP6dIuYi}" \
  -H "Authorization: Bearer ${SUPABASE_SERVICE_KEY:-sb_publishable_cWfZRmHb55YMA-wa7hb6rw_qP6dIuYi}" \
  -H "Content-Type: application/json" \
  -d '{"param": "value"}' | jq .
```

### 3. Query a table (requires SUPABASE_SERVICE_KEY for non-public data)

```bash
curl -s "https://fwawlsbjltgvcpyezciw.supabase.co/rest/v1/<table>?select=*" \
  -H "apikey: $SUPABASE_SERVICE_KEY" \
  -H "Authorization: Bearer $SUPABASE_SERVICE_KEY" \
  -H "Accept-Profile: IDI_Accounts" | jq .
```

Use `Accept-Profile: IDI_Accounts` for tables in that schema, omit for `public`.

### 4. Delete / update rows (requires SUPABASE_SERVICE_KEY)

```bash
curl -s -X DELETE "https://fwawlsbjltgvcpyezciw.supabase.co/rest/v1/<table>?<column>=eq.<value>" \
  -H "apikey: $SUPABASE_SERVICE_KEY" \
  -H "Authorization: Bearer $SUPABASE_SERVICE_KEY" \
  -H "Content-Profile: IDI_Accounts" | jq .
```

---

## Common tasks for this project

| Task | Best method |
|------|-------------|
| Delete non-admin users | Raw SQL via SUPABASE_MGMT_TOKEN |
| List all profiles | Query `IDI_Accounts.profiles` with service key |
| Call game RPC (save_game, etc.) | RPC with anon key |
| Create/alter tables or functions | Raw SQL via SUPABASE_MGMT_TOKEN |

### Delete all non-admin accounts (example)
```sql
DELETE FROM "IDI_Accounts".profiles WHERE is_admin = false;
```

---

## Workflow

1. Identify which operation is needed and which credential it requires.
2. Check if that credential is set in the environment.
3. If missing, ask the user to provide it before proceeding.
4. Run the curl command, pipe through `jq .` for readable output.
5. Report success/failure and show relevant rows returned.
