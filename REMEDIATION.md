# Remediation

## Where the search input meets SQL

Advanced ticket search is `advanced_ticket_search` in `meridian-servicedesk/src/helpdesk/views/staff.py`. The form sends the typed value as the `q` query parameter.

The statement text is fixed. The only placeholder is `%s`:

```sql
SELECT id, title, submitter_email, status
FROM helpdesk_ticket
WHERE title LIKE %s
ORDER BY created DESC
```

`q` is passed as the bound parameter, after `%` and `_` in the typed text are escaped so they stay ordinary characters inside the `LIKE` pattern:

```python
cursor.execute(sql, ["%" + escaped + "%"])
```

The database receives one statement and one data value. The value can change which titles match. It stays data for that comparison.

## The change

The change is parameter binding. The search text is the second argument to `cursor.execute`, and the SQL string contains `%s` where that value belongs.

A second raw query uses the same change. `ticket_status` in `meridian-servicedesk/src/helpdesk/views/public.py` compares the request reference with `WHERE secret_key = %s` and `cursor.execute(sql, [ref])`.

Ticket-list filters go through the Django ORM. Field names are checked against an allowlist before they are applied.

## What to re-check

Send the same advanced-search request again. A word that appears in a title should still return that ticket. A word that appears in none of the titles should return no rows. The statement should stay the `SELECT` above, with the typed text only in the bound `LIKE` value.

Capture that request and response for the submission, and name the change as parameter binding in `advanced_ticket_search`.
