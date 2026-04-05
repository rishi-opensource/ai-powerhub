# Input Source Handling

## Local Files

Use the `Read` tool with the absolute path provided.

```
Read("/Users/rishi/docs/design.md")
```

For multiple local files, read each in parallel.

## Google Docs

Extract the document ID from the URL:
`https://docs.google.com/document/d/<DOC_ID>/edit`

Preferred: use GWS CLI if available.
Fallback: use `WebFetch` with the URL directly.

If authentication fails, inform the user and ask them to paste the content manually.

## Confluence Pages

**By URL**: Extract the page ID or use `mcp__atlassian__fetchAtlassian` with the full URL.

**By page ID**: Use `mcp__atlassian__getConfluencePage` with `pageId`.

**By search**: Use `mcp__atlassian__searchConfluenceUsingCql` if only a title or space is known.

Fetch the body content (`body.storage` or `body.view`). Strip HTML tags if needed.

## Jira Tickets

Use `mcp__atlassian__getJiraIssue` with the issue key (e.g., `DGP-402`).

Extract: summary, description, comments, acceptance criteria, linked issues.

## Slack Threads

Use `mcp__plugin_slack_slack__slack_read_thread` with channel ID and thread timestamp.

If only a channel is provided, use `mcp__plugin_slack_slack__slack_read_channel`.

## Raw Pasted Text

Use directly — no tool needed. Treat as a first-class document.

## Other URLs (READMEs, web pages, etc.)

Use `WebFetch` with the URL. Extract the main content and ignore navigation/ads.

## When a Source Fails

- Note the failure clearly at the top of your synthesis
- Proceed with all successfully retrieved inputs
- Do not block the synthesis on a single unreachable source
