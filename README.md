# LinkedIn AI Post Generator

An automated n8n workflow that researches, drafts, and publishes weekly LinkedIn posts about AI — with a human approval step in the loop before anything goes live.

## Overview

Every week, the workflow:
1. Pulls the latest AI news from an RSS feed.
2. Uses an LLM to turn that news into a LinkedIn-ready post — or, if no fresh news is available, has the LLM pick and write about its own current AI/agentic-systems topic.
3. Sends the draft to a specified email address for approval.
4. Publishes the approved draft directly to LinkedIn. If declined (or no response within the wait window), the post is discarded and the workflow ends quietly.

This removes the manual work of finding a topic and writing a post each week, while still keeping a human in control of what actually gets published.

## Workflow Diagram

```
Weekly Trigger (Mon 9:00 AM)
        │
Fetch AI News (RSS)
        │
Keep Latest Article
        │
Aggregate News Item
        │
   Has Fresh News? ──────────────┐
        │ Yes                    │ No
Write Post From News      Write Post From Topic
   (OpenAI)                   (OpenAI)
        │                        │
        └──────── Merge Drafts ──┘
                    │
            Request Approval (Gmail, send & wait)
                    │
              Approved? ──────────────┐
                │ Yes                  │ No
        Publish To LinkedIn      Post Discarded
```

## Nodes

| Node | Purpose |
|---|---|
| **Weekly Trigger** | Schedule trigger, runs every Monday at 9:00 AM |
| **Fetch AI News** | Reads an AI-focused RSS feed (TechCrunch AI by default) |
| **Keep Latest Article** | Keeps only the most recent article from the feed |
| **Aggregate News Item** | Collapses results into a single item so the next check always runs, even with zero articles |
| **Has Fresh News?** | Branches based on whether an article was found |
| **Write Post From News** | LLM chain that rewrites the article into a LinkedIn-voice post |
| **Write Post From Topic** | LLM chain that generates an original post on a self-chosen AI topic (fallback) |
| **Merge Drafts** | Combines both branches into a single path |
| **Request Approval** | Gmail node (send-and-wait) — emails the draft and waits for approve/decline |
| **Approved?** | Branches on the approval response |
| **Publish To LinkedIn** | Publishes the approved draft as a LinkedIn post |
| **Post Discarded** | No-op end state when declined or the approval window times out |

## Requirements

- An n8n instance (cloud or self-hosted)
- **OpenAI** credential (used by both LLM drafting nodes)
- **Gmail** credential (used to send the approval request)
- **LinkedIn** credential (OAuth2, used to publish the post)

## Setup

1. Import/create the workflow in n8n.
2. Attach your OpenAI and Gmail credentials to their respective nodes (n8n will auto-suggest existing credentials if available).
3. On the **Publish To LinkedIn** node, connect a LinkedIn OAuth2 credential, then select your **Person** from the dropdown (only loads once the credential is connected).
4. Update the **Request Approval** node's `sendTo` field with your own email address.
5. (Optional) Swap the RSS feed URL on **Fetch AI News** for a different AI news source.
6. Publish/activate the workflow.

## Configuration Notes

- **Schedule**: defaults to weekly (Monday, 9:00 AM). Adjust the interval on the Weekly Trigger node for a different cadence.
- **Approval window**: the Gmail send-and-wait node times out after 3 days by default; an unanswered request is treated as declined.
- **Fallback behavior**: if the RSS feed returns no items (or errors), the workflow doesn't skip the week — it generates an original post on a current AI topic instead.
- **Tone**: both LLM nodes use a system prompt tuned for a first-person, non-corporate AI/ML engineer voice, ending with 3–5 relevant hashtags.

## Tech Stack

n8n · OpenAI · Gmail (OAuth2) · LinkedIn (OAuth2) · RSS
