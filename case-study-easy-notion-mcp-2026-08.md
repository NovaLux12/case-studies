# Case Study: Notion MCP server swap (built-in → easy-notion-mcp) (2026-08)

**Topic:** Replaced OpenClaw's built-in `notion` skill with the third-party `easy-notion-mcp` MCP server for token efficiency, then added a Notion-backed Task Queue database with a polling cron for agent task execution.

**Outcome:** MCP server swap completed cleanly; Task Queue database live with schema for status, priority, assignee, and due date; polling cron runs every 30 minutes to pick up Nova-assigned tasks.

## What happened

The built-in `notion` skill worked but produced verbose responses. The `easy-notion-mcp` server promises markdown-first design and claims 6–7× fewer response tokens. Swapped it in as a replacement.

### The swap

1. Disabled `skills.entries.notion` in OpenClaw config
2. Added `mcp.servers.notion` pointing to `npx -y easy-notion-mcp` with `NOTION_TOKEN` env
3. Restarted gateway
4. Verified with `notion__read_page` — returned Tech Wiki content as markdown, confirming the MCP server works

### Task Queue database

Created a Notion database under the Tech Wiki root:
- **Name:** Task Queue
- **Schema:** Name (title), Status (Todo/In Progress/Done/Blocked), Priority (Low/Medium/High/Urgent), Assigned (Jack/Nova/Cleo), Due (date), Notes (rich_text)
- Added a sample task to verify write access

### Polling cron

Created a 30-minute polling cron that:
- Queries the Task Queue for highest-priority Todo task assigned to Nova
- Executes it
- Marks it Done

## Lessons

- MCP server swaps are surgical: disable the skill, add the server, verify with a read call. The gateway restart is the only downtime.
- Token efficiency claims should be verified with real data, not just marketing copy. The swap was made on the promise of 6–7× reduction, but actual reduction depends on page structure and query patterns.
- Notion databases need explicit parent page IDs on creation — same gotcha as the wiki build (`ntn pages create` defaulted to People database).
- The `ntn` CLI auth can desync from the MCP server auth after a gateway restart. The token stored in OpenClaw config works for the MCP; the CLI needs re-login if used directly.
- Polling crons for task queues should filter by assignee and status to avoid picking up stale or irrelevant tasks.

## Files

- OpenClaw config: `mcp.servers.notion` added, `skills.entries.notion` disabled
- Cron ID: `7aae3195-292b-4dd2-8e0b-fff5145d09b9`
- Notion database ID: `24880221-e39d-41bf-b437-2e55eff8691f`
