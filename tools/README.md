# Meta Ads tools (patched for @MagliaCL)

Vendored copies of Attainment Labs' MIT-licensed tools, with Instagram identity support added:

| Tool | Upstream commit |
|---|---|
| `meta-ads-cli` | attainmentlabs/meta-ads-cli@6db1373f24067b7eeffbc42ed680cb708e8d06d4 |
| `meta-ads-mcp` | attainmentlabs/meta-ads-mcp@5e3d18dc134c3ab3e53a2ddb30264187b2629fd1 |

## What changed

- New env vars:
  - `META_INSTAGRAM_USERNAME` (e.g. `MagliaCL`): the expected Instagram handle. Before creating a creative,
    the tools look up the Instagram account linked to `META_PAGE_ID` and **refuse** if it is missing or a different handle.
  - `META_INSTAGRAM_USER_ID` (optional): skip the lookup and use this ID directly.
- Ad creatives include `instagram_user_id`, so Instagram placements show as @MagliaCL instead of the Facebook Page.
- CLI: `meta-ads instagram` shows the linked account and whether it matches.
- MCP: new read-only tool `get_instagram_account`.
- Neither variable set → original behaviour (Page identity).

## Requirements on Meta's side

1. @MagliaCL is a professional (Business/Creator) account.
2. It is linked to the Facebook Page in `META_PAGE_ID`, in the same Business portfolio as the ad account.
3. The access token includes `instagram_basic` (plus `ads_management`, `ads_read`, `pages_read_engagement`, `pages_show_list`).

## Install

```bash
uv tool install --force ./tools/meta-ads-cli   # CLI: `meta-ads`
# MCP: configured in ../.mcp.json (runs from tools/meta-ads-mcp)
```

## meta-mcp (organic Instagram: posts, reels, stories, comments, DMs, insights)

Git submodule `tools/meta-mcp` → https://github.com/panxoku/meta-mcp (pinned commit). Configured in `../.mcp.json` as `meta-instagram`.

```bash
git submodule update --init tools/meta-mcp
(cd tools/meta-mcp && npm ci && npm run build)
```

Env vars (set in the cloud environment, never commit):
- `INSTAGRAM_ACCESS_TOKEN`: can be the same Meta user token as `META_ACCESS_TOKEN` if it also has
  `instagram_basic`, `instagram_content_publish`, `instagram_manage_comments`, `instagram_manage_insights`.
- `INSTAGRAM_USER_ID`: @MagliaCL's ID — `meta-ads instagram` prints it.
- `INSTAGRAM_PAGE_ID` + `INSTAGRAM_MESSAGING_ACCESS_TOKEN`: only for DMs.
- `META_APP_ID` / `META_APP_SECRET`: only for token/webhook tools.
