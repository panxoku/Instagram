---
name: meta-ads-playbook
description: Playbook for planning, launching, testing and scaling Meta (Facebook/Instagram) ad campaigns, including access-token setup, audience targeting, the 20/60/20 budget split, creative testing, scaling signals and Meta Marketing API error fixes. Use when the user asks about Meta/Facebook/Instagram ads, campaign structure, ad budgets, ad creative tests, when to scale, Meta API tokens, or Meta ads API errors (190, 100, 1487390, 368, 17).
---

# Meta Ads Playbook

Based on Attainment Labs' "Engineer's Playbook for Meta Ads AI Automation"
(https://www.attainmentlabs.com/meta-ads-playbook). Apply these rules when
designing or reviewing campaigns; explain the reasoning to the user.

## Tooling options

- **Meta official Ads AI Connectors**: Meta-hosted OAuth, first-party support.
- **meta-ads-cli** (open source, https://github.com/attainmentlabs/meta-ads-cli): campaigns as YAML files, repeatable terminal launches.
  ```
  pip install meta-ads-cli
  cp campaign.example.yaml campaign.yaml
  meta-ads validate --config campaign.yaml
  meta-ads create --config campaign.yaml --dry-run
  meta-ads create --config campaign.yaml
  meta-ads activate <campaign-id>
  ```
- **meta-ads-mcp** (open source, https://github.com/attainmentlabs/meta-ads-mcp): `uvx meta-ads-manager-mcp`, for agent-driven work.

Campaigns should be created **PAUSED**; the user reviews in Ads Manager before activating. Never activate spend without explicit user confirmation.

## 1. Access token

1. Create an app at developers.facebook.com (type: Business).
2. Graph API Explorer → select the app → request `ads_management`, `pages_read_engagement`, `pages_show_list`.
3. Generate the token; store it as `META_ACCESS_TOKEN` in `.env` (never commit it).
4. Exchange for a long-lived token (short-lived = ~2 h, long-lived = ~60 days):
   ```
   curl "https://graph.facebook.com/v21.0/oauth/access_token?grant_type=fb_exchange_token&client_id=APP_ID&client_secret=APP_SECRET&fb_exchange_token=SHORT_TOKEN"
   ```
5. Remind the user to renew before 60 days.

## 2. Audiences

- **Interest targeting**: cold audiences; 3–5 tightly related interests, don't stack unrelated ones. Find IDs with `GET /v21.0/search?type=adinterest&q=<term>`.
- **Custom audiences**: customer lists / pixel retargeting; privacy-compliant uploads only.
- **Lookalikes**: source ≥ 100 people; 1% = most precise, 5–10% = wider. Move to lookalikes after ~1,000 pixel conversions.
- **Broad** (age/gender/location only): only for large budgets; not recommended under $100/day.
- Start with interest targeting to prove creative first.

Example targeting:
```yaml
ad_set:
  targeting:
    age_min: 25
    age_max: 45
    genders: [0]          # 0 = all
    countries: ["US", "CA"]
    interests:
      - id: "6003139266461"
        name: "Fitness and wellness"
    platforms: ["facebook", "instagram"]
    instagram_positions: ["stream", "story", "reels"]
```

## 3. Budget: 20/60/20

- 20% testing (new creatives/audiences/objectives, low spend until a winner)
- 60% scaling proven winners (ROAS > 2.5x), increase incrementally
- 20% retention / retargeting

`daily_budget` is in the smallest currency unit (cents/pence): `1000` = $10.00/day.

## 4. Creative testing

- 3 variants per ad set; change **one** variable at a time (order: image → headline → primary text).
- Don't judge before **$50 spend** per test.
- Winner = lowest CPC; pause losers and move their budget to the winner, then test the next variable.

## 5. Scaling — only when all four are green

1. ROAS > 2.5x sustained for 7+ days
2. CPM stable (not rising week over week)
3. Frequency < 3.0
4. Audience saturation < 50%

How to scale:
- Raise `daily_budget` by **20% every 2–3 days** (bigger jumps reset learning), or
- Duplicate the ad set with a higher starting budget, leaving the original running.

## 6. Common Meta API errors

| Code | Type | Cause | Fix |
|---|---|---|---|
| 190 | OAuthException | Token expired/invalid | Regenerate in Graph API Explorer, exchange for long-lived |
| 100 | Invalid Parameter | Bad YAML / unsupported value | Run `meta-ads validate`; check objective + optimization_goal pair |
| 1487390 | Billing Error | No valid payment method | Add payment method in Business Manager → Billing |
| 368 | Policy Violation | Creative/targeting breaks policy | Review against Meta Advertising Standards; remove restricted claims |
| 17 | Rate Limit | Too many calls | Wait 15–30 min; batch creation off-peak |
