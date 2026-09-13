# Broker Inbox Triage Lite

Watches a Gmail inbox and reads each new email as it arrives. Sorts every one into a single category and adds the matching Gmail label, so the inbox is filed before anyone opens it.

That is the whole job. Nothing is replied to, nothing is deleted, nothing is moved out of the inbox. The only change to the mailbox is one label added to each new message.

## The eight categories

Every email gets exactly one.

- **renewal** | policies coming up for renewal, renewal invitations, renewal terms from an insurer
- **claim** | anything to do with a claim, new or ongoing
- **certificate_request** | requests for a certificate of currency or other proof of cover
- **endorsement** | mid-term changes to an existing policy, such as adding a vehicle or changing a sum insured
- **new_enquiry** | new business, either a prospect making contact for the first time or an existing client asking about additional or different cover
- **insurer_correspondence** | mail from an insurer in its own right, such as policy documents being issued, underwriting queries, product or appetite updates and systems notices
- **noise** | newsletters, marketing, automated notifications, anything needing no action
- **unknown** | anything the model is not confident about, and any email whose model response could not be read

The instructions tell the model to use **unknown** whenever it is unsure, and not to force an uncertain email into the nearest category. Mail sitting in unknown is the design working, not the design failing. A wrong label is worse than no label.

## What it does not do

It never replies to anyone. It never writes a draft. It never opens, changes or cancels a policy. It never tells anyone whether something is covered.

The instructions given to the model forbid it from stating or implying policy terms, cover, limits, excess, premium, dates or eligibility, and forbid it from giving any opinion on whether something is covered. When an email asks a coverage question, the model still files it and writes a short summary, and that is as far as it goes.

This is deliberate. Filing email is administration. Answering a coverage question is advice, and advice belongs with a licensed person who has read the policy. Keeping those two apart is the point of the design, not a gap to be closed later.

The model also writes a one-line summary, capped at 20 words, describing what the email is about. In this version nothing is done with it. It appears in the workflow's own run log inside n8n and is not added to the email or sent anywhere.

## What you need

- **n8n** | the automation tool that runs the workflow. Free to self-host on a machine you already have. n8n also sell a hosted version on subscription if you would rather not run a server.
- **A Gmail account** | the inbox being watched. Works with personal Gmail and Google Workspace.
- **An OpenAI API key** | pay as you go, no subscription, no minimum.

**Running cost.** The workflow uses gpt-4o-mini and sends it only the sender address, the subject and the first 600 characters of the body. At current prices that is under one US cent per 100 emails. An inbox taking 2,000 emails a month costs roughly 15 US cents to classify. Gmail adds nothing. Self-hosted n8n costs whatever the machine costs, which can be nothing. Model prices change, so check OpenAI's pricing page before relying on that figure.

## Setup

1. Import `Broker_Inbox_Triage_LITE.json` into n8n, under Workflows then Import from File.
2. In Gmail, create eight labels, one for each category above. Name them whatever suits your office.
3. Find the ID of each label and paste it into the **Settings** node, replacing the eight `PASTE_..._LABEL_ID` placeholders. A label ID is not the label name, it looks like `Label_1234567890`. The quickest way to find them is to add a temporary Gmail node set to Label then Get Many, run it once, read the IDs from the output, then delete that node.
4. Connect your credentials. Gmail on the **New email** and **Apply label** nodes, and your OpenAI key on the **Classify** node, which expects a header auth credential with `Authorization` set to `Bearer YOUR_KEY`.
5. Turn the workflow on. It arrives switched off and checks for new mail once a minute.

Before it classifies anything, the workflow checks whether the message already carries one of your eight labels and skips it if so, so nothing gets labelled twice.

## Paid version

The paid version adds draft replies, urgent alerts and a Google Sheet log of everything it files: LINK_TO_GUMROAD
