# LinkedIn Posting with AI Agents: A Human-in-the-Loop Guide

An AI agent can do much of the preparation for a LinkedIn post, but the named author still owns every claim. Give the agent specific source material and ask for a first draft. Check the facts, add what you know from experience, and make publication a separate decision. Groniz Connectors handles OAuth, per-platform formatting, scheduling, and delivery to LinkedIn as part of its support for 32+ networks. The author still makes the editorial call. This guide explains how to prepare sources, review a draft, meet the live delivery requirements, and record useful results from LinkedIn's native analytics.

## Workflow at a glance

```text
Approved source material
  → LinkedIn-specific brief
  → agent creates a first draft
  → author adds expertise and approves the final text
  → Groniz schedules the approved post
  → delivery is verified
  → native LinkedIn results inform the next brief
```

The pause before scheduling is deliberate. Draft approval means "these words are ready." Publication approval confirms the account, date, time, media, and platform settings. Treat them as two decisions.

## Prerequisites

You need:

- an AI agent with access to the source material;
- a Groniz account and a connected LinkedIn profile or Page;
- a Groniz Skill, CLI, or MCP connection supported by your agent;
- a specific source, such as release notes, research, a customer lesson, or an event recap;
- approved media, if the post needs it;
- a person responsible for facts, voice, and publication approval;
- `jq` if you use the CLI media example below.

Groniz supports LinkedIn profiles and Pages, but channel capabilities and required fields can vary. The output of `groniz integrations:settings <id>` for the live connection is authoritative. Avoid building a permanent template around fields copied from another integration.

For a broader view of the connector model, start with [Introducing Groniz Connectors](https://groniz.com/blog/introducing-groniz-connectors). Agent-specific paths are covered in the [Codex guide](https://groniz.com/blog/automate-social-media-posting-with-codex) and [Claude Code guide](https://groniz.com/blog/automate-social-media-posting-with-claude-code).

## Connect your agent to Groniz

Claude Code, Codex, OpenCode, and OpenClaw can use a Skill, the native CLI, or MCP. Other clients may support a different subset, so use the path documented for the agent that will run the workflow.

Install the standard Skill with:

```bash
npx skills add groniz/groniz-cli
```

For a visible terminal workflow, install and authenticate the native CLI:

```bash
curl -fsSL https://groniz.com/install.sh | sh
groniz auth:login
groniz whoami
```

For MCP clients, connect to the remote endpoint:

```text
https://mcp.groniz.com/mcp
```

MCP authentication and configuration differ by client. Follow the Groniz setup for your client rather than copying a command meant for another agent. The delivery examples below use the CLI because you can inspect the destination, settings, media path, and scheduled time in the terminal.

## Why LinkedIn needs a human in the loop

[LinkedIn's guidance for AI-assisted writing](https://www.linkedin.com/help/linkedin/answer/a1517763) treats AI output as a starting point. It recommends specific input, followed by review, revision, and the author's own expertise or personal knowledge. That matters when a post makes professional claims. An agent can organize the evidence and try different structures. Only the author can confirm that an observation matches what happened.

A LinkedIn draft usually needs one clear point and enough context to support it. The ending should give readers something worth responding to instead of tacking on a generic question. The post should sound like a professional contribution, not a landing page squeezed into a smaller box. Pages and personal profiles also call for different voices. A founder can write a lesson in the first person. A company Page should state the organization's role plainly.

## Prepare evidence before asking for prose

Create a short source brief instead of prompting from memory. For example:

```markdown
# LinkedIn source brief

Audience: Engineering leaders evaluating our migration approach
Purpose: Share one lesson from the migration
Confirmed facts:
- Cutover date: 2026-07-12
- 18 services moved
- Rollback was tested twice

Author insight:
- The hard part was dependency ownership, not deployment tooling

Evidence/link:
- https://example.com/engineering/migration-notes

Exclude:
- Unreleased roadmap items
- Customer names
- Performance claims not in the source
```

The brief sets boundaries and leaves an audit trail. If it does not contain a number or quotation, the draft should leave that detail out or mark it for verification. Ask the agent to improve the structure and use specific details without covering up gaps in the evidence.

## Reusable LinkedIn drafting prompt

Save this beside the source brief:

```text
Draft one LinkedIn post from the attached source brief.

Goal: explain one useful professional lesson, not summarize every detail.
Structure: opening observation, concrete context, the author's lesson,
and one relevant question or understated closing line.

Rules:
- Use only facts in the source.
- Mark missing evidence as [VERIFY]; do not infer it.
- Preserve the author's point of view without inventing experience.
- Avoid promotional superlatives and generic engagement bait.
- Suggest a profile or Page voice, and explain the choice in one sentence.
- Return the draft, a fact-to-source checklist, and two alternative openings.

Do not schedule or publish anything. Draft approval is not publication approval.
```

Ask the agent to include the fact map with the draft. Review is faster when unsupported claims stay visible after the prose has been polished.

## Review the draft as the named author

Use a concrete checkpoint:

- Can every number, name, date, and causal claim be traced to the source?
- What would only the author know? Add that detail in the author's own words.
- Would the author say this aloud? Remove borrowed certainty and generic inspiration.
- Does the post help the intended reader before asking for attention?
- Does the voice fit a profile or a Page?
- Is the media relevant and approved? Confirm any accessibility requirements exposed by the live integration.
- Is the link correct, and has someone explicitly approved the proposed time?
- Has a person approved the exact final text and, separately, the publication details?

The agent solves the blank-page problem. During revision, the author supplies the judgment and remains responsible for any claim that carries consequences.

## Deliver the approved post with Groniz

After the final copy is approved, find the live connection and assemble the publication details:

```bash
groniz whoami
groniz integrations:list
groniz integrations:settings LINKEDIN_INTEGRATION_ID
```

Inspect every required setting, the current length limit, and any tools returned for that integration. Trigger a dynamic integration tool only if the live settings list it. If a live requirement forces a copy change, send the revision through copy review again. Upload approved media first and capture the returned `.path`:

```bash
LINKEDIN_MEDIA_PATH="$(
  groniz upload ./approved/linkedin-image.png | jq -r '.path'
)"
```

Use only that uploaded path in the scheduling request. A local path or external media URL is not a substitute. Then ask the agent to prepare a schedule request using the exact live schema:

```text
Using integration LINKEDIN_INTEGRATION_ID, prepare a scheduled post for
2026-08-06T15:00:00Z with the approved copy below. Use the uploaded media
.path if supplied. Show the complete resolved payload and required settings.
Wait for explicit publication approval before calling the write tool.
```

The agent should show the resolved copy, integration, settings, media, timestamp, and timezone. A person then makes the separate publication decision. Once that approval is recorded, the CLI request has this shape:

```bash
groniz posts:create \
  -c "APPROVED_LINKEDIN_CONTENT" \
  -m "$LINKEDIN_MEDIA_PATH" \
  -s "2026-08-06T15:00:00Z" \
  -t schedule \
  -i "LINKEDIN_INTEGRATION_ID" \
  --settings '<required-settings-json>'
```

Replace the content and settings placeholders with the approved copy and every required value from the live response. If the post has no media, omit `-m`; if the live response has no required settings, omit `--settings`. This example is a command template, not a claim that one fixed LinkedIn payload works for every connection. The live schema wins.

## Verify delivery and handle failure

After authorization, capture the returned Groniz post ID, scheduled time, target integration, status, and any platform URL. Check the scheduled-post list rather than treating a successful request as a confirmed delivery:

```bash
groniz posts:list \
  --startDate "2026-08-06T00:00:00Z" \
  --endDate "2026-08-07T00:00:00Z"
```

Find the returned post ID and confirm its destination, scheduled time, and state.

Match the fix to the failure:

- If authentication fails, rerun `groniz whoami` and confirm the intended account.
- If the destination is wrong, resolve the ID again. Display names are not enough.
- For a schema or length rejection, refresh `integrations:settings` and revise against the current requirements.
- For a media rejection, confirm that the upload completed and that the request uses its `.path`.
- If permissions fail, reconnect or correct the LinkedIn profile/Page authorization.
- If an uncertain response creates a duplicate risk, inspect scheduled posts before retrying.

If the copy changes during recovery, obtain copy approval again. A change to the media, destination, or timing needs new publication approval. Record the resolution so you can add a preflight check when the same failure recurs.

## Measure the next iteration on LinkedIn

Use LinkedIn's native analytics for the relevant surface. The available measures vary, but [LinkedIn's Page posting guide](https://www.linkedin.com/help/linkedin/answer/a1660869) documents impressions, members reached, click-through rate, reactions, comments, and reposts for Page posts. Judge each post against its purpose. Relevant comments may tell you more about a discussion post, while clicks may be the better measure for a resource post.

Keep a simple log with the opening, topic, format, author edits, measurement window, and the next question you want to test. That record gives the next brief something concrete to work with. The delivery layer still cannot guarantee growth.

When you are ready to connect an approved workflow, configure the appropriate LinkedIn destination in [Groniz Connectors](https://groniz.com/console/connectors).
