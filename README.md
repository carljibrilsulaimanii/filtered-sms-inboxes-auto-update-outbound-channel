<!--
  README.md -- Filtered SMS Inboxes and Auto-Update Outbound Channel (Aloware + HubSpot)
  Author:  Jibril Sulaiman
  Date:    2026-10-08
  What:    A setup guide for keeping marketing short code replies out of sales reps' inboxes. An AloAi
           agent sorts each reply, and buyers are moved to a toll-free sales line and their owner by
           one automatic text from that line.
  Why:     Aloware has no way to move a conversation to another line or filter a shared inbox by
           owner. When one short code carries both marketing blasts and sales replies, every blast
           buries the conversations reps are working.
  Status:  DESIGN AND SETUP PLAN, NOT YET BUILT OR TESTED. See the banner below.
-->

# Filtered SMS Inboxes and Auto-Update Outbound Channel (Aloware + HubSpot)

> **Status: design and setup plan, not yet built or tested.** Every step is based on Aloware's
> and HubSpot's documentation and on how the two platforms behave, but nothing here has been run end
> to end yet. Labels that couldn't be confirmed on screen say *(wording may differ)*. The
> [vendor questions](#open-questions-for-aloware) list what still needs confirming, and
> [Step 14](#step-14-test) is the test plan. This README will be updated with screenshots and
> results once the build is live.

When one SMS short code carries both marketing blasts and sales conversations, every blast floods the
reps' inbox with "STOP", "👍" and "yes" replies, and the people they were really talking to get buried.
This guide sets up Aloware and HubSpot so that:
- an AloAi agent sorts every reply to a marketing text,
- only buyers are moved to a separate **toll-free sales line** and their owner, by one automatic text
  from that line,
- support and general questions stay with marketing and customer service,
- each rep sees only their own sales conversations.

It's for teams running Aloware for SMS and calling, synced with HubSpot.

## Why it exists

**What the platforms can't do natively:**
1. **Aloware can't move a conversation to another line.** There's no button, setting or API call that
   moves a conversation to another inbox or changes a contact's outbound number. Aloware support
   confirmed this, including for the API. Other platforms (for example GoHighLevel) let you set a
   contact's outbound channel from the contact owner. Aloware has no equivalent.
2. **Aloware has no global default SMS line.** The line comes from the conversation, the sequence, the
   workflow action, or the agent's choice in the message box.
3. **A conversation stays with the last line that texted it.** Aloware support confirmed this: messages
   stay in the inbox of the line they came in on until a communication happens from a different line.
4. **Shared inboxes don't separate owners well.** Teams found Aloware's shared inbox hard to filter by
   owner. Aloware's docs list Contact Owner and Line filters, so check your account. HubSpot inbox
   views can filter by assignee, but a view is a filter, not a lock: any rep can open "All".
5. **The Aloware SMS channel in HubSpot's inbox opens one conversation per reply.** One blast to a few
   thousand contacts can open thousands of HubSpot conversations within a day or two.
6. **An AloAi agent is scoped to one line,** and a contact can be in only **one Aloware sequence** at a
   time.

**What goes wrong without this:** right after a blast, roughly 9 in 10 replies are opt-outs and most
of the rest are reactions. The few buyers, the support issues and the rep's ongoing conversations all
land in the same list. Reps scroll through thousands of threads, and anything sent after a weekend
gets lost.

**The way around it:** since a conversation follows the last line that texted it, **one outbound text
from the toll-free sales line moves the whole conversation** to that line's inbox. An AloAi agent
decides who gets that text, so only buyers move to sales.

## How it works

```
 Marketing blast goes out from the SHORT CODE
        │
 Contact replies
        │
 AloAi agent "Short Code Reply Router" (short code line only) sorts the reply
        │
        ├── opt-out ("Stop")                       → nothing (Aloware opt-out handling)
        ├── THANKS / NOT_INTERESTED                → tag only, no reply
        ├── UNCLEAR                                → "Needs Review" tag for marketing
        ├── CLASS_INFO ("What time?")              → one short answer, stays on short code
        ├── SUPPORT ("cancel", "link broken")      → one reply + "Customer Service" tag, stays on short code
        ├── ALREADY_HANDED_OFF (replied in the old short code thread) → nudge sequence from toll-free
        └── SALES ("I want in", "how much?")       → enroll in "SC Reply to Sales Handoff"
                                                            │
 HubSpot gives the contact an owner if they have none (an existing owner is never changed)
        │
 After 2 minutes the sequence texts FROM THE TOLL-FREE LINE:
   "Hi Ana, thanks for reaching out to [Your Company]! Sam from our enrollment team will text you
    from this number shortly."
        │
 The conversation now lives on the TOLL-FREE line:
   Aloware: toll-free inbox → the owner's "My Sales Texts" filter
   HubSpot: "SMS - Sales" inbox, assigned to the owner
   HubSpot marks the contact "SMS Handed to Sales" → left out of marketing blasts from now on
        │
 The rep's first text sets Lead status = In Progress
```

## What's in this repo

| Path | What it is | Where it goes |
|---|---|---|
| [`README.md`](README.md) | This setup guide | — |
| [`aloai-reply-router-instructions.txt`](aloai-reply-router-instructions.txt) | The AloAi agent's instructions: buckets, example replies, actions per bucket, rules | Aloware → AloAi Agents → *Short Code Reply Router* → **Instructions** ([Step 9](#step-9-the-reply-router-agent)) |

There's no custom code. Everything is built in Aloware and HubSpot settings, sequences, workflows and
one AI agent.

## Table of contents

- [1. Requirements](#1-requirements)
- [2. Before you start](#2-before-you-start)
- [3. Setup, step by step](#3-setup-step-by-step)
  - [Step 1: HubSpot properties and lists](#step-1-hubspot-properties-and-lists)
  - [Step 2: Rep visibility and inbox membership in Aloware](#step-2-rep-visibility-and-inbox-membership-in-aloware)
  - [Step 3: Line routing](#step-3-line-routing)
  - [Step 4: Saved inbox filters](#step-4-saved-inbox-filters)
  - [Step 5: The handoff sequence](#step-5-the-handoff-sequence)
  - [Step 6: The nudge sequence](#step-6-the-nudge-sequence)
  - [Step 7: HubSpot owner and "handed to sales" workflows](#step-7-hubspot-owner-and-handed-to-sales-workflows)
  - [Step 8: No-bot fallback workflow (build it, leave it off)](#step-8-no-bot-fallback-workflow-build-it-leave-it-off)
  - [Step 9: The reply router agent](#step-9-the-reply-router-agent)
  - [Step 10: Keep marketing away from sales contacts](#step-10-keep-marketing-away-from-sales-contacts)
  - [Step 11: HubSpot inbox channels](#step-11-hubspot-inbox-channels)
  - [Step 12: "Texts to answer" workflows and views](#step-12-texts-to-answer-workflows-and-views)
  - [Step 13: Lead status In Progress](#step-13-lead-status-in-progress)
  - [Step 14: Test](#step-14-test)
  - [Step 15: Go live](#step-15-go-live)
- [The channel switch, in detail](#the-channel-switch-in-detail)
- [Aloware inbox vs HubSpot inbox for reps](#aloware-inbox-vs-hubspot-inbox-for-reps)
- [What each team sees](#what-each-team-sees)
- [Training notes](#training-notes)
- [Day-to-day operations](#day-to-day-operations)
- [Troubleshooting](#troubleshooting)
- [Open questions for Aloware](#open-questions-for-aloware)
- [Limits](#limits)
- [Security](#security)

## 1. Requirements

| Need | Why |
|---|---|
| **Aloware** with two SMS lines: a **short code** (marketing) and a **toll-free** number (sales) | The switch only works between two different lines |
| **AloAi** text agents enabled on the Aloware account | The reply router |
| Aloware **admin** (Users, Inboxes, Lines, Sequences+, Tags, AloAi Agents, Settings → Integrations) | Every Aloware step |
| **Aloware ↔ HubSpot integration** connected, with the **Conversations** and **Custom Channels** scopes | SMS in the HubSpot inbox, SMS fields on contacts, Aloware workflow actions |
| HubSpot plan with **contact workflows**, **Rotate record to owner**, and inbox **views** (Sales or Service Hub Professional or higher) | Steps 7, 8, 11, 12, 13 |
| Optional: **Service Hub Enterprise** | Only for a hard owned-only lock through Help Desk ([Step 11g](#step-11-hubspot-inbox-channels)) |
| Contacts **owned in HubSpot**, with HubSpot users **linked to Aloware users** | Owner routing, owner names in the handoff text, the "Send As" field |

**SMS fields Aloware syncs to HubSpot contacts** (check the internal names in your portal):
`last_inbound_sms___date`, `last_inbound_sms__message`, `last_outbound_sms___date`,
`last_outbound_sms__message`, `sms_optout`. If your names differ, use yours everywhere below.

## 2. Before you start

About 30 minutes, plus a call with your Aloware account manager.

**2a. Fill in this worksheet.**

| Item | Where | Value |
|---|---|---|
| Short code line ID | Aloware → **Lines** → short code line (ID in the URL) | `REPLACE_WITH_SHORT_CODE_LINE_ID` |
| Toll-free line ID | Aloware → **Lines** → toll-free line | `REPLACE_WITH_TOLL_FREE_LINE_ID` |
| Toll-free line can send SMS/MMS | Aloware → **Lines** → toll-free → **General settings** | Yes / No |
| AloAi agent already on the short code? | Aloware → **AloAi Agents** → each text agent → assigned lines | None / *name* |
| Short code inbox name | Aloware → **Admin → Inboxes** | |
| HubSpot sales team / marketing team / support user | HubSpot → **Settings → Users & Teams** | |
| Lead status values that mean "do not contact" | HubSpot → **Settings → Properties → Contact → Lead status** | |
| A fixed phrase every marketing blast includes | Agree with marketing | |

**2b. Link every rep.** Aloware → **Admin → Users** → *user* → HubSpot user mapping *(wording may
differ)*. Owner names, owner routing and "Send As" all depend on it.

**2c. Make two test contacts** on phones you control: **Test A** owned by a rep, **Test B** with no
owner. Don't use a contact that has several emails or duplicates merged onto it.

**2d. Dedupe shared phone numbers.** When two HubSpot contacts share a number, Aloware links messages to
the first one, and the conversation view breaks.

**2e. Decide these with the sales and marketing leads:**

| Decision | Recommendation |
|---|---|
| Owners come from HubSpot. Aloware never assigns owners in this flow. | Yes. Unowned repliers get an owner from HubSpot's rotation ([Step 7](#step-7-hubspot-owner-and-handed-to-sales-workflows)). An existing owner is never changed. |
| Handoff wording and delay | 2 minutes. Under 1 minute feels robotic. |
| Is "interested but not now" (price objections, "saving up") sales? | Yes. Reps work those. |
| Where reps work day to day | Start in Aloware (no new habit). See [the comparison](#aloware-inbox-vs-hubspot-inbox-for-reps). |
| Should the short code also appear in a HubSpot inbox? | Yes, in a separate **Marketing SMS** inbox that marketing and support work, or disconnect it. Replies still log on each contact either way. |
| One agent per line: is another agent already on the short code? | If yes, merge the router's buckets into that agent's instructions. |
| Who works the short code inbox (Needs Review, manual switches) on blast days? | Name one person per shift. |
| Run the bot in tag-only mode first? | Yes, 2 days, then check 100 tags by hand. |

✅ **Check:** the worksheet is complete, every rep is linked, and the decisions are signed off.

## 3. Setup, step by step

**Settings for every HubSpot workflow in this guide:**
- **Settings → When to run actions:** any time, including evenings and weekends.
- **Unenrollment:** *"When a contact no longer meets the criteria"* **OFF** *(wording may differ)*.
- **Turning on:** choose to enroll **only contacts who meet the trigger after turning on**. Don't
  enroll existing contacts, or every past replier gets processed at once.
- **While testing:** add the filter **List membership → is member of `TEST - SMS handoff`**. Remove it
  in [Step 15](#step-15-go-live).

### Step 1: HubSpot properties and lists

About 10 minutes.

**1a. Create four contact properties.** **Settings → Properties → Object: Contact → Create
property.** Group: **Aloware / SMS** (create the group if needed) → **Next** → field type → **Create**.

| Label | Internal name | Field type | Description |
|---|---|---|---|
| SMS Handed to Sales | `sms_handed_to_sales` | Single checkbox | Yes once the sales handoff text has gone out. Excludes the contact from marketing SMS. |
| SMS Handed to Sales Date | `sms_handed_to_sales_date` | Date picker | When the handoff text was sent. |
| SMS Needs Response | `sms_needs_response` | Single checkbox | Yes when a rep owes this contact a text. |
| SMS Handoff Owner | `sms_handoff_owner` | HubSpot user | The owner at the time of the handoff, for audits. |

**1b. Exclusion list.** **CRM → Lists → Create list → Contact-based → Active list**, named
`SMS - Handed to Sales (exclude from marketing)`, filter **SMS Handed to Sales is equal to Yes** → **Save**.

**1c. Test list.** **Static list** `TEST - SMS handoff`, containing Test A and Test B.

✅ **Check:** the four properties show on a contact record, and both lists exist.

### Step 2: Rep visibility and inbox membership in Aloware

About 10 minutes.

**2a. Owned Only.** Aloware → **Admin → Users** → *rep* → **Visibility Settings** → **Owned Only** →
save. It takes effect immediately. Do one rep first, then the rest after
[Step 14](#step-14-test) test T3.

> ⚠️ **Owned Only hides unowned contacts.** That's fine here, because every replier gets an owner from
> HubSpot within minutes ([Step 7](#step-7-hubspot-owner-and-handed-to-sales-workflows)). Leave managers,
> admins, marketing and support on **Everything**.

**2b. Take reps off the short code inbox.** **Admin → Inboxes** → *short code inbox* → members →
remove every sales rep. Keep marketing, support and managers.

> ⚠️ **Owned Only isn't enough on its own.** Aloware's docs say Owned Only users can still see and
> reply in a team inbox they're a member of. Removing reps from the short code inbox is what really
> hides blast replies.

**2c. Put reps on the toll-free inbox.** **Admin → Inboxes** (or **Lines → toll-free → assigned
users**): every sales rep is a member.

**2d. Auto-assign contact.** **Account settings → Contact settings → Auto-assign contact:** leave
**ON**. It only acts on contacts with no owner, when a user texts them first, so it never overrides
a HubSpot owner.

✅ **Check:** logged in as the test rep, the short code inbox isn't listed and the toll-free inbox is.

### Step 3: Line routing

About 5 minutes. Ask your Aloware account manager if the setting names don't match.

**3a. Short code line.** Aloware → **Lines** → short code → routing / inbox settings *(wording may
differ)*: send inbound texts to the **short code team inbox**. **Not** to the contact owner, and no
sales rep as the fallback.

**3b. Toll-free line:** route to the **contact owner**.

> ⚠️ **Why this matters.** A line that routes inbound texts to the contact owner drops every blast
> reply from a rep's own contacts into that rep's **Personal** inbox. That recreates the flood even
> after Step 2b.

✅ **Check:** a blast reply from a contact owned by the test rep doesn't appear in that rep's
Personal inbox.

### Step 4: Saved inbox filters

About 5 minutes.

Aloware's docs list inbox filters for **Contact Owner**, **Assigned Contacts**, **Line**, direction,
**Unread Only**, **Opt-Out**, saved filter sets, and company-wide filters set by admins. Check that they
appear in your account. If not, ask your account manager.

**4a.** Inbox → **Filters**: **Line** = toll-free, **Contact Owner** = me (or **Assigned
Contacts**), **Opt-Out** = hide. Save as **My Sales Texts** and make it a **company-wide** filter.
**4b.** For support: **Line** = short code + communication tag **Customer Service**, saved as **Support
Replies**.
**4c.** For marketing: communication tag **Needs Review**, saved as **Needs Review**.

✅ **Check:** the test rep sees **My Sales Texts** in their filter list.

### Step 5: The handoff sequence

About 10 minutes. This sequence sends the **switching text**, the one that moves the conversation.

**5a.** Aloware → **Sequences+** → **+New Sequence**:

| Section | Setting | Value |
|---|---|---|
| Basic information | Name | `SC Reply to Sales Handoff` |
| Schedule settings | Process on | **Daily** (not business days) |
| Communication and line settings | Outbound line | **Toll-free** |
| | Use contact's owner line | **OFF** |
| | Use contact's last communication line | **OFF** |
| Enrollment controls | Prevent enrollment if contact is communicating with an agent | **OFF** |
| | Remove leads if they respond | OFF |
| | Remove if opted-out | **ON** |
| | Throttle | **As high as your account allows.** Dozens of buyers can reply within minutes of a blast. |

**5b.** **Save & Continue** → **Steps** → **Add Steps**:
1. **Add tag to contact** → `SMS - Handed to Sales`. Delay: **No Delay**.
2. **Send SMS message**. Delay: **Default Delay, 2 minutes** (time for HubSpot to set an owner).
   Use the variable picker for both names:
   > Hi [FirstName], thanks for reaching out to [Your Company]! [Owner first name] from our enrollment team will text you from this number shortly.

**5c. Intake & Triggers:** none. The agent enrolls contacts in [Step 9](#step-9-the-reply-router-agent).

> ⚠️ **Keep the phrase `from our enrollment team will text you`.** The HubSpot workflows in Steps 7,
> 12 and 13 look for it. If the owner name comes out blank in testing, use *"a member of our
> enrollment team will text you from this number shortly"*, which still contains it.

> ⚠️ **If either "use contact's … line" option is ON, the switch fails.** The text goes out on the
> line the contact last used (the short code), and nothing moves.

✅ **Check:** the Sequences+ list shows **Line used = toll-free** for this sequence.

### Step 6: The nudge sequence

About 5 minutes. This is for a contact who was already switched but replies in the **old short code
thread** on their phone.

**6a.** Same settings as Step 5, named `SC Reply to Sales Nudge`. One step, **Send SMS message**, **No
Delay**:
> Got it, thanks! [Owner first name] will reply from this number. Please text us here from now on.

It doesn't contain the handoff phrase, so Step 7's workflow W2 doesn't run again.

✅ **Check:** Line used = toll-free.

### Step 7: HubSpot owner and "handed to sales" workflows

About 15 minutes.

**7a. W1 `SMS - Assign owner to unowned repliers`.** **Automation → Workflows → Create workflow →
From scratch → Contact-based.**
1. **Trigger:** **When an event occurs → Property value changed → Last Inbound SMS - Date** has any
   value. **Re-enrollment ON.**
2. **Filters (all):** Contact owner **is unknown** · SMS Opt-out **is not** Yes · Last Inbound SMS -
   Message **doesn't contain** `stop` · doesn't contain `remove`.
3. **Action:** **Rotate record to owner** → your sales team → owner property **Contact owner** →
   **Overwrite current owner: unchecked**.

**7b. W2 `SMS - Mark handed to sales`.**
1. **Trigger:** **Last Outbound SMS - Date** has any value. **Re-enrollment ON.**
2. **Filter:** Last Outbound SMS - Message **contains** `from our enrollment team will text you`.
3. **Actions:** **Edit record** → `sms_handed_to_sales` = Yes · **Edit record** →
   `sms_handed_to_sales_date` = **Date of step** · **Copy property value** → Contact owner →
   `sms_handoff_owner` · **Edit record** → `sms_needs_response` = Yes.

> ⚠️ **Never tick "Overwrite current owner".** It would take contacts away from the reps who already
> own them.

✅ **Check:** after the handoff test in Step 14 (T1), the contact shows SMS Handed to Sales = Yes with
today's date.

### Step 8: No-bot fallback workflow (build it, leave it off)

About 15 minutes. While the agent is off, this hands off **every** reply that isn't an opt-out. It
uses HubSpot's Aloware action instead of the bot.

**8a.** Contact-based workflow `SMS - Short code reply to Sales handoff (no bot)`.
**Trigger:** Last Inbound SMS - Date has any value, re-enrollment ON.

**8b. Filters (all):**

| Property | Operator | Value |
|---|---|---|
| SMS Handed to Sales | is not equal to | Yes |
| SMS Opt-out | is not equal to | Yes |
| Last Inbound SMS - Message | doesn't contain | `stop` |
| Last Inbound SMS - Message | doesn't contain | `remove` |
| Last Inbound SMS - Message | doesn't contain | `unsubscribe` |
| Lead status | is none of | your "do not contact" values |

**8c. Actions:**
1. **If/then branch:** Contact owner is unknown → **Rotate record to owner** (overwrite unchecked).
2. **Delay** → 2 minutes.
3. **Aloware: Send a Text (SMS/MMS)**, found at the bottom of the action list:

   | Field | Value |
   |---|---|
   | Aloware Line to Send From | **Toll-free.** This is the switch. |
   | Send As | **HubSpot Contact Owner.** The owner must be linked in Aloware, or nothing sends. |
   | Phone Field | **Phone Number**, or Mobile Phone Number if that's where numbers live. An empty field means nothing sends. |
   | Phone Number | blank |
   | Message | `Hi {{ contact.firstname }}, thanks for reaching out to [Your Company]! {{ contact.hubspot_owner_id }} from our enrollment team will text you from this number shortly.` |
   | Media URL | blank |

**8d.** Save. **Leave it OFF.**

> ⚠️ **Never run it while the agent is on,** or buyers get two handoff texts. If the owner token prints
> a full name that reads badly, use the no-name wording from Step 5.

✅ **Check:** the workflow is saved and OFF.

### Step 9: The reply router agent

About 45 minutes, plus 2 days in tag-only mode.

**9a. Check the buckets against your own replies.** Pull a few days of short code replies. Two ways:
the HubSpot inbox (or an AI assistant connected to it) filtered to the short code channel, or a CSV
export from Aloware. Sort them into the buckets in
[`aloai-reply-router-instructions.txt`](aloai-reply-router-instructions.txt), then replace the example
replies with real ones from your traffic. Fill in every `[YOUR_...]` placeholder.

**9b. Create tags.** Aloware → **Admin → Tags**: `AI - Sales Intent`, `AI - Support`,
`AI - Class Info`, `AI - Thanks`, `AI - Not Interested`, `AI - Unclear`, `SMS - Handed to Sales`.
Communication tags: `Sales`, `Customer Service`, `Needs Review`.

**9c. Create the agent.** Aloware → **AloAi Agents** → **+ New agent** → **Text**. Name:
`Short Code Reply Router`. Line: **short code only**.

**9d. Instructions.** Paste all of
[`aloai-reply-router-instructions.txt`](aloai-reply-router-instructions.txt), starting at the line
`=== 1. ROLE ===`. The header comment above that line isn't part of the prompt.

**9e. Actions panel.** Add these, with the data actions before **Disengage contact**:

| Action | Setting | When |
|---|---|---|
| Add Tags | one per bucket tag | the bucket matched |
| Add Communication Tag | Sales / Customer Service / Needs Review | SALES and ALREADY_HANDED_OFF / SUPPORT / UNCLEAR |
| Enroll to sequence | `SC Reply to Sales Handoff` | SALES |
| Enroll to sequence | `SC Reply to Sales Nudge` | ALREADY_HANDED_OFF |
| Send SMS | support reply (from the file) | SUPPORT |
| Send SMS | class-info fallback (from the file) | CLASS_INFO when the answer isn't in the last blast |
| Create HubSpot ticket | your support pipeline, status New | optional, SUPPORT about a charge, refund or cancelling |
| Disengage contact | — | after every bucket |

**9f. Tag-only mode.** Leave both **Enroll to sequence** actions and both **Send SMS** actions **OFF**,
then activate. For 2 days the agent only adds tags. Check 100 tagged replies by hand, and move on when
SALES and SUPPORT are right at least 9 times out of 10.

> ⚠️ **The test list doesn't limit the agent.** It reads every real reply on the short code. Test in
> tag-only mode first. If the agent can read contact tags (ask your account manager), add a temporary
> first line to the instructions, *"TEST MODE: take actions only for contacts tagged TEST - SMS handoff.
> For everyone else, only add tags."*, while you run the full tests. Otherwise switch the actions on at
> a quiet time, not within 2 hours of a blast, and watch the first 30 minutes.

> ⚠️ **Don't put the agent on the toll-free line.** It would sort the reps' own conversations.

✅ **Check:** after 2 days, the tags match the messages at least 9 times out of 10.

### Step 10: Keep marketing away from sales contacts

About 15 minutes, with marketing.

1. Every HubSpot list or segment used for SMS blasts: **List membership is not member of
   `SMS - Handed to Sales (exclude from marketing)`**.
2. Every Aloware broadcast audience excludes the tag `SMS - Handed to Sales`.
3. Every marketing drip sequence: **Enrollment controls → Remove leads if they respond: ON**. A reply
   ends the drip, which also frees the contact's one sequence slot.
4. Every HubSpot workflow that sends marketing SMS through an Aloware action gets the filter
   **SMS Handed to Sales is not equal to Yes**.
5. **Short code keyword triggers** (on the short code line's **Triggers** tab) send automatic replies
   from the short code and take the contact's sequence slot. Keep them to opt-in confirmations, and
   exclude the tag `SMS - Handed to Sales` from any keyword sequence that sends more than one text.

> ⚠️ **Any short code text to a switched contact pulls them back** to the short code inbox, out of the
> rep's view. See [The channel switch, in detail](#the-channel-switch-in-detail).

✅ **Check:** a test blast audience that includes Test A after its handoff leaves Test A out.

### Step 11: HubSpot inbox channels

About 30 minutes. **Do this in order.** Connect the toll-free line before moving the short code, so the
sales inbox never goes without a line.

**11a. Connect the toll-free line.** HubSpot → **Settings → Inbox & Help Desk → Inboxes** → **SMS -
Sales** → **Channels → Actions → Connect Channel** → **Aloware SMS** → choose the **toll-free** line.
**Routing: Contact owner**. If no owner, leave it unassigned.

**11b. Check the sync.** Aloware → **Channel Accounts**: the toll-free line shows connected and synced.
If not, click **Sync with HubSpot**. If the line won't connect, go to **Settings → Integrations →
HubSpot → Re-authenticate**.

**11c. Rescue live conversations.** Before closing anything, skim the last 48 hours of short code
conversations for reps' active sales conversations and switch them by hand
([method 3](#four-ways-to-send-the-switching-text)).

**11d. Move the short code.** Disconnect the short code's **Aloware SMS** channel from SMS - Sales.
Then go to **Marketing SMS → Channels → Actions → Connect Channel → Aloware SMS** and choose the short code line.
Routing: unassigned. (Or leave it disconnected, per your decision in 2e.) Each Aloware line can connect
to **one** inbox only.

**11e. Access.** **Settings → Inboxes →** *inbox* **→ Access:** SMS - Sales = sales team and managers.
Marketing SMS = marketing, support and managers.

**11f. Close the old flood.** SMS - Sales → **All open** → filter to the short code channel → select
all → **Close**, in batches. Closing deletes nothing, and every message stays on the contact.

**11g. Views.** In SMS - Sales: **My sales texts** (Assignee = me, Open, the reps' default),
**Unassigned**, **All open - audit**. In Marketing SMS: **Open, newest first**, bulk-closed daily.
Optional hard lock: move SMS - Sales into **Help Desk** and set the sales team's permission to
**Tickets → View: Owned only**. Test it on one rep first.

> ⚠️ **HubSpot inbox threads only start from incoming messages.** The switching text alone doesn't
> create an SMS - Sales conversation. One appears when the contact replies on the toll-free line. Reps
> can't start new texts from the HubSpot inbox. They use Aloware or the Aloware card on the contact.

✅ **Check:** a reply from Test A on the toll-free line appears in SMS - Sales, assigned to its owner.

### Step 12: "Texts to answer" workflows and views

About 20 minutes.

**12a. `SMS - Needs response ON`.** Trigger: Last Inbound SMS - Date has any value, re-enrollment ON.
Filters: SMS Opt-out is not Yes · SMS Handed to Sales = Yes. Action: `sms_needs_response` = Yes.

**12b. `SMS - Needs response OFF`.** Trigger: Last Outbound SMS - Date has any value, re-enrollment ON.
Filters: Last Outbound SMS - Message **doesn't contain** `from our enrollment team will text you` ·
doesn't contain `will reply from this number` · doesn't contain your marketing phrase. Action:
`sms_needs_response` = No.

**12c. Contact views.** **Contacts → + Add view → Create new view:**

| View | Filters | Columns | Sort | Share with |
|---|---|---|---|---|
| **My texts to answer** | SMS Needs Response = Yes · **Contact owner = Me** | Name, Phone, Last Inbound SMS - Message, Last Inbound SMS - Date, Lead status | Oldest first | Sales team |
| **All texts to answer** | SMS Needs Response = Yes | Same + Contact owner | Oldest first | Managers |
| **Handoffs today** | SMS Handed to Sales Date = today | Name, SMS Handoff Owner, Last Inbound SMS - Message, Lead status | Newest first | Managers |
| **Unowned texts** | SMS Needs Response = Yes · Contact owner is unknown | Name, Phone, Last Inbound SMS - Message | Oldest first | Managers |

"Contact owner = Me" means each rep who opens the shared view sees only their own contacts. It's a
filter, not a lock. Reps can set **Lead status = DNC** right from the view when the message column
shows "STOP".

✅ **Check:** after T2, Test A appears in its owner's **My texts to answer**.

### Step 13: Lead status In Progress

About 10 minutes.

**13a.** Workflow `SMS - Lead status In Progress on first rep text`.
- **Trigger:** Last Outbound SMS - Date has any value, re-enrollment ON.
- **Filters (all):** Lead status is New · Contact owner is known · SMS Handed to Sales = Yes · Last
  Outbound SMS - Message doesn't contain `from our enrollment team will text you` · doesn't contain
  `will reply from this number` · doesn't contain your marketing phrase.
- **Action:** Lead status = **In Progress**, plus an "In Progress date" property if you have one.

> ⚠️ **Sequence texts are logged under the contact owner**, the same as a rep's text. Without the
> message filters, the bot's handoff text would flip the status.

**Aloware-only alternative:** with **Auto-assign contact** ON, build a sequence triggered by **Contact
owner changed** (one trigger per rep), with one step, **Change contact disposition status → In
Progress**. The disposition syncs to HubSpot Lead status. It only fires when the owner changes, so it
misses contacts that already have a HubSpot owner. Use it only if contacts arrive without owners.

✅ **Check:** after the rep's first reply in T4, Lead status is In Progress.

### Step 14: Test

About 1 hour, plus the 2 days of tag-only mode. Send each test as a one-person broadcast from the short
code, then reply from the test phone.

**Bot sorting (tag-only mode):**

| # | Reply | Expected tag | Expected action |
|---|---|---|---|
| T10 | "Stop" | none | Nothing sent. Opt-out recorded. |
| T11 | "Loved …" or "👍" | AI - Thanks | **Nothing sent.** This also answers whether the agent can stay silent. |
| T12 | "What time is class?" | AI - Class Info | One short answer from the short code |
| T13 | "I was charged twice, please cancel" | AI - Support | Support reply, Customer Service tag, stays on short code |
| T14 | "How much is it? I want in" | AI - Sales Intent | Enrolled in the handoff |
| T15 | "Yes" to a blast that asked "Want the link?" | AI - Sales Intent | Enrolled |
| T16 | "Who is this?" | AI - Unclear | Needs Review, nothing sent |

**Switch and inboxes (actions on):**

| # | Do | Expected in Aloware | Expected in HubSpot |
|---|---|---|---|
| T1 | Test A replies "I want in" | ~2 min later, the handoff arrives **from the toll-free number** and names the owner | W2: Handed to Sales = Yes, date, Handoff Owner, Needs Response = Yes |
| T2 | Test A replies "?" twice | Both arrive on toll-free, in the owner's **My Sales Texts**. **No** second handoff. | Conversation in SMS - Sales, assigned to the owner |
| T3 | Log in as the owning rep | Thread visible. Short code inbox not visible. No blast replies in their Personal inbox. | — |
| T4 | The rep replies, once from Aloware and once from the HubSpot inbox | Both go out from toll-free | Needs Response = No. Lead status → In Progress. |
| T5 | Test B (no owner) replies "how much?" | The handoff names the rotated rep | W1 rotated an owner first |
| T6 | Test A replies "STOP" to a new short code blast | Nothing sent | Not in any rep view |
| T7 | Marketing builds a test blast audience including Test A | Test A excluded | Test A is in the exclusion list |
| T8 | Check Test A's owner throughout | Unchanged | Unchanged |
| T9 | Manual switch from **All Inboxes** on a Needs Review contact | Line dropdown is offered, and the thread moves to toll-free | Mark by hand or include the phrase |
| T17 | After T1, Test A replies in the **old short code thread** | The nudge arrives from toll-free. **No** second handoff. | — |
| T18 | Test B is in a drip sequence and replies "I want in" | Moved into the handoff, or skipped. Note which. | — |

✅ **Check:** every test passes, with screenshots saved.

### Step 15: Go live

About 15 minutes.

1. Remove the test gate line from the agent's instructions, if you added one.
2. Remove the `TEST - SMS handoff` filter from every HubSpot workflow.
3. Set the remaining reps to **Owned Only**, remove them from the short code inbox and add them to the
   toll-free inbox.
4. Send the [training notes](#training-notes).
5. Watch the first real blast with marketing: **Handoffs today** (every row should read like a buyer)
   and **Unowned texts** (should stay empty).

**Rollback, one piece at a time:**

| If this goes wrong | Do this |
|---|---|
| The bot routes buyers wrongly or replies when it shouldn't | Switch the agent off, or back to tag-only. Turn on the [Step 8](#step-8-no-bot-fallback-workflow-build-it-leave-it-off) workflow if handoffs must continue. |
| Handoff texts misfire | Pause `SC Reply to Sales Handoff` |
| Reps can't see contacts | Visibility back to **Everything** |
| Reps miss replies in HubSpot | Reconnect the short code to SMS - Sales |
| Lead status flips wrongly | Turn off the Step 13 workflow |
| Marketing excludes too many | Remove the exclusion from the blast audience |

✅ **Check:** the first blast after go-live produces handoffs only for buyers, and the reps' views stay clean.

## The channel switch, in detail

### How it works

One outbound text from the **toll-free** line changes three things at once:
1. the conversation's current line becomes toll-free, so it shows in the **toll-free inbox**,
2. rep replies default to toll-free, because that's the conversation's line,
3. the contact replies to the number that just texted them, so **their next reply comes in on toll-free**.

```
 BEFORE                                         AFTER one text from the toll-free line
 Contact ⇄ SHORT CODE                           Contact ⇄ TOLL-FREE
 Aloware: short code inbox (everyone)           Aloware: toll-free inbox → owner's "My Sales Texts"
 HubSpot: flood in SMS - Sales                  HubSpot: new SMS - Sales thread when they reply
 Rep replies go out from the short code         Rep replies go out from toll-free
 Next blast lands mid-conversation              Contact is excluded from blasts
```

### Four ways to send the switching text

| # | Method | When | How toll-free is chosen |
|---|---|---|---|
| 1 | **Agent → handoff sequence** | Normal path | Sequence **Outbound line = toll-free**, both "use contact's … line" options OFF ([Step 5](#step-5-the-handoff-sequence)) |
| 2 | **HubSpot "Aloware: Send a Text (SMS/MMS)"** | Bot off, or any HubSpot-triggered case | **Aloware Line to Send From = toll-free** ([Step 8](#step-8-no-bot-fallback-workflow-build-it-leave-it-off)) |
| 3 | **By hand, with the line dropdown** | The bot missed a buyer, or a Needs Review contact | Open the conversation from **All Inboxes**, pick toll-free in the message box's line dropdown, send |
| 4 | **Aloware SMS API** | Future custom code only | `POST https://app.aloware.io/api/v1/webhook/sms-gateway/send` with the toll-free line id (check the field names under Aloware → Integrations → API) |

The text must contain `from our enrollment team will text you`, or tick SMS Handed to Sales by hand.

> **Why the dropdown sometimes doesn't appear:** when a conversation is opened **inside the short code
> inbox**, Aloware locks the sender to the short code. Open it from **All Inboxes** instead.

### What changes when it's sent

| Where | Before | After |
|---|---|---|
| Aloware conversation line | Short code | **Toll-free** |
| Aloware inbox | Short code inbox | **Toll-free inbox**, shown in the owner's "My Sales Texts" |
| Who sees it in Aloware | Everyone in the short code inbox | The owner (Owned Only), plus managers |
| Older short code messages | Short code inbox | **Still there.** History isn't moved, and the contact record shows both. |
| Aloware tags | `AI - Sales Intent` | Plus `SMS - Handed to Sales` |
| HubSpot inbox | Short code conversation | **No new conversation until the contact replies** |
| HubSpot contact | — | Last Outbound SMS = the switching text. Handed to Sales = Yes. |
| Contact owner | Unchanged | Unchanged |

### What the contact sees

Phones keep a separate thread per number, so after the switch there are two: the short code thread
with the marketing texts, and the toll-free thread with the handoff and the rep's texts. The handoff
says *"from this number"* so they reply on toll-free.

### What pulls a contact back to the short code

| Cause | Prevention |
|---|---|
| A blast or drip from the short code | Exclusions ([Step 10](#step-10-keep-marketing-away-from-sales-contacts)) |
| A short code keyword auto-reply | Keep keyword sequences to opt-ins, and exclude the handed-off tag |
| The contact replies in the old short code thread | The ALREADY_HANDED_OFF rule → nudge sequence ([Step 6](#step-6-the-nudge-sequence)) |
| Someone picks the short code in the dropdown | Reps aren't in the short code inbox, plus training |
| An opt-out on the short code | Nothing to do. It doesn't apply to the toll-free number. |

### How to confirm a switch

1. The phone got a text from the toll-free number.
2. In Aloware, the latest outbound line is toll-free, and the conversation is in the toll-free inbox
   and the owner's My Sales Texts.
3. The HubSpot contact shows Last Outbound SMS - Message = the switching text and Handed to Sales = Yes.
4. After the contact replies, a new SMS - Sales conversation appears, assigned to the owner.

## Aloware inbox vs HubSpot inbox for reps

Both work once the switch is in place. The choice is only where reps read and reply.

| | Reps work in **Aloware** (recommended start) | Reps work in the **HubSpot inbox** |
|---|---|---|
| What reps open | Toll-free inbox → **My Sales Texts** | **SMS - Sales** → **My sales texts** |
| "Only my conversations" | **Owned Only**, enforced by Aloware | **Assigned to me** view. A filter, not a lock, unless you use Help Desk. |
| Hides blast replies | Yes (Steps 2b, 3) | Yes (Step 11d, 11e) |
| Start a new text | Yes | No, they reply only |
| Contact info while texting | Aloware contact panel | Full HubSpot record next to the thread |
| Change for the team | None, if they already use Aloware | New habit |

Start in Aloware. Move reps to HubSpot if the Aloware filters aren't available or the shared inbox is
slow. Try it with two reps first.

## What each team sees

| Bucket | Aloware | HubSpot |
|---|---|---|
| SALES | Toll-free inbox, owner's My Sales Texts | SMS - Sales, assigned to the owner. In My texts to answer. |
| SUPPORT | Short code inbox, Support Replies filter | Marketing SMS. Optional support ticket. |
| CLASS_INFO | Short code inbox, answered by the bot | Marketing SMS |
| THANKS / NOT_INTERESTED | Short code inbox, tagged | Marketing SMS, bulk-closed |
| UNCLEAR | Short code inbox, Needs Review filter | Marketing SMS |
| Opt-out | Opt-out recorded | SMS Opt-out = Yes |

## Training notes

**Sales reps**
1. Your sales texts are on the toll-free line: **My Sales Texts** in Aloware, or **SMS - Sales → My
   sales texts** in HubSpot.
2. A bot reads replies to marketing texts. Buyers get a text from the toll-free number saying you'll
   reach out, and then they show up in your list.
3. Never reply from the short code. To move someone yourself, open them from **All Inboxes**, pick the
   toll-free number in the line dropdown, and send.
4. Start each shift with **My texts to answer**, oldest first.
5. Your first text sets Lead status to In Progress automatically.
6. For a new text, pick the toll-free number in the dropdown. The HubSpot inbox can only reply.

**Marketing**
1. Work the **Needs Review** filter daily, and hand off buyers the bot missed.
2. Every blast audience excludes the handed-to-sales list or tag.
3. Drips: **Remove leads if they respond = ON**. Keep the shared phrase in every blast.
4. Bulk-close Marketing SMS daily.

**Support**: work **Support Replies** and reply from the short code.

**Managers**: check **All texts to answer**, **Handoffs today** and **Unowned texts** daily.

## Day-to-day operations

| Task | Steps |
|---|---|
| New rep | HubSpot: add to the sales team. Aloware: create the user, link it, set Owned Only, add to the toll-free inbox (not the short code inbox), share the filters. |
| Rep leaves | Reassign their contacts in HubSpot first, then deactivate them in both tools. |
| New marketing campaign | Exclude handed-to-sales, include the shared phrase. If it asks a new kind of question ("Reply KEYWORD"), add that reply to the SALES examples. |
| Bot instructions change | Edit the file first, then paste it into the agent. Note the date in the header. |
| Handoff wording change | If the phrase changes, update Steps 7, 12 and 13 the same day. |
| Weekly check (15 min) | Check 20 random tagged replies, sequence errors, Channel Accounts sync, the unassigned count, and 5 handoffs (right owner, sent from toll-free). |
| Every 2 weeks at first | Repeat Step 9a on live tags. |

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Buyer got no handoff | Sorted into another bucket, or tag-only mode | Check the AI tag, switch by hand (method 3), add the example to SALES ([Step 9](#step-9-the-reply-router-agent)) |
| Support issue landed with a rep | Sorted as SALES | Send it to support, add the example to SUPPORT |
| Bot replied to "Stop" or a reaction | Instructions not followed, or the agent always replies | Tighten Section 2 of the instructions. Ask Aloware about silent actions. |
| Handoff came from the short code | A "use contact's … line" option is ON, or the wrong Outbound line | [Step 5](#step-5-the-handoff-sequence) |
| Bot enrolled, but no text arrived | Sequence paused, contact opted out, or empty phone field | Check the sequence and the contact |
| Owner name blank | No owner yet, or the owner isn't linked in Aloware | [2b](#2-before-you-start). Lengthen the delay, or use the no-name wording. |
| Two handoff texts | Two quick SALES replies, or ALREADY_HANDED_OFF wasn't recognised | [Step 6](#step-6-the-nudge-sequence). Ask Aloware about enroll-once. |
| Contact dropped out of the rep's view | Something went out from the short code after the switch | Find the exclusion that missed ([Step 10](#step-10-keep-marketing-away-from-sales-contacts)), then switch again |
| Rep still sees blast replies | Still a member of the short code inbox, or the short code routes to owners | [Steps 2b](#step-2-rep-visibility-and-inbox-membership-in-aloware) and [3](#step-3-line-routing) |
| Rep's new text went out from the short code | The dropdown was left on the short code | Pick toll-free |
| Line dropdown missing | Conversation opened from the short code inbox | Open it from **All Inboxes** |
| No-bot workflow sent nothing | "Send As" owner not linked, or empty phone field | [Step 8](#step-8-no-bot-fallback-workflow-build-it-leave-it-off) |
| Handoffs late after a big blast | Low sequence throttle | Raise it ([Step 5](#step-5-the-handoff-sequence)) |
| SMS - Sales still flooding | Short code still connected there | [Step 11d](#step-11-hubspot-inbox-channels) |
| "Unknown Visitor" in the HubSpot inbox | No matching contact, or duplicates | [2d](#2-before-you-start) |
| Lead status flipped from a blast or the handoff | Phrase filters missed | [Step 13](#step-13-lead-status-in-progress) |

## Open questions for Aloware

Ask your account manager before building:
1. Routing: can the short code go to the team inbox only and the toll-free line to the contact owner? What's the sequence throttle limit?
2. Can AloAi text-agent actions run **without the agent replying**?
3. Do opt-out replies reach the agent, and what does the agent cost per message at your reply volume?
4. Are the Contact Owner and Line inbox filters, and company-wide saved filters, available on your account?
5. If a contact is already in a sequence, does the agent's **Enroll to sequence** move them or skip them? Is there a force option?
6. Can the agent read contact tags (needed for ALREADY_HANDED_OFF and the test gate)?
7. Can a sequence refuse a second enrollment of the same contact?
8. Does **Create HubSpot ticket** work from a text agent?
9. Do replies sent from the HubSpot inbox always go out on the conversation's line?
10. Is a native "move conversation to another line" action, or a contact-level outbound line, planned?

## Limits

- **Untested plan.** See the status banner.
- The switch holds only while nothing new goes out from the short code to that contact.
- An AI classifier makes mistakes. The Needs Review filter, the manual switch and the weekly check catch them.
- HubSpot views are filters, not locks, unless you use Help Desk permissions.
- One sequence per contact in Aloware. Drips and keyword sequences can compete with the handoff.
- Every message the agent reads may cost AI credits.

## Security

- No API keys or tokens live in this repo, and none are needed for the main path. Method 4 (the SMS
  API) needs an Aloware API token. Keep it in a secret store, never in code.
- The agent's instructions forbid sending prices, payment links or promises. Sales stays with people.
- Opt-outs are never answered or tagged by the agent. Aloware's carrier-compliant opt-out handling covers them.

Related: [aloware-ai-hubspot-scheduling](https://github.com/carljibrilsulaimanii/aloware-ai-hubspot-scheduling), where an AloAi agent books calls on a HubSpot meeting link.

Built by [Jibril Sulaiman](https://github.com/carljibrilsulaimanii).
