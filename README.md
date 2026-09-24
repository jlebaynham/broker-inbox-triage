# Broker Inbox Triage

This is the full workflow, and it is free.

It watches a Gmail inbox and reads each new email as it arrives. It sorts every email into a
category, files it under a Gmail label, drafts a reply where one is safe to draft, tells you when
something is urgent, and keeps a record of every email in a Google Sheet.

Setup guide | free on Gumroad: https://baynhams.gumroad.com/l/broker-inbox-triage

## What it does

For every new email:

1. **Sorts it** into one of eight categories.
2. **Labels it** in Gmail, under a `Broker/` label for its category, so the inbox is filed before
   anyone opens it.
3. **Logs it** as a new row in a Google Sheet: the date, who it came from, the subject, the
   category, whether it is urgent, and a one-line summary.
4. **Drafts a reply** for new enquiries and certificate requests only. The draft is a short, polite
   acknowledgement saved in Gmail for a person to check and send. Nothing is ever sent
   automatically.
5. **Alerts you** by email when it judges something to be urgent, with the sender, the subject and
   the summary.

Nothing is deleted and nothing is moved out of the inbox. The only changes to the mailbox are one
label on each email and, for the two draft categories, a draft reply sitting in the thread.

## The eight categories

Every email gets exactly one.

- **renewal** | policies coming up for renewal, renewal invitations, renewal terms from an insurer
- **claim** | anything to do with a claim, new or ongoing
- **certificate_request** | requests for a certificate of currency or other proof of cover
- **endorsement** | mid-term changes to an existing policy, such as adding a vehicle or changing a
  sum insured
- **new_enquiry** | new business, from a prospect making contact for the first time or an existing
  client asking about additional or different cover
- **insurer_correspondence** | mail from an insurer in its own right, such as policy documents,
  underwriting queries, product updates and systems notices
- **noise** | newsletters, marketing, automated notifications, anything needing no action
- **unknown** | anything the workflow is not confident about, and any email it could not read

It is told to use **unknown** whenever it is unsure, rather than forcing an email into the nearest
category. Mail sitting in unknown is the design working, not failing. A wrong label is worse than
no label.

## It never talks about cover

The workflow never states or implies cover, limits, excess, premium, dates or eligibility, and never
gives an opinion on whether something is covered.

When an email asks a coverage question, the workflow files it and writes a short summary, and that
is as far as it goes. It does not draft a reply and it does not raise an alert for it. A person
reads it and answers it.

This is deliberate. Filing email is administration. Answering a coverage question is advice, and
advice belongs with a licensed person who has read the policy. The drafts it does write are
acknowledgements only, such as "thanks, we have your request and will be in touch".

## What you need

- **n8n** | the automation tool that runs the workflow. Free to run on a computer you already have,
  or available as a hosted service if you would rather not look after it yourself.
- **A Gmail account** | the inbox being watched. Personal Gmail and Google Workspace both work.
- **A Google Sheet** | where the log is kept.
- **An OpenAI API key** | pay as you go, no subscription.

**Running cost.** Each email is read by OpenAI's gpt-4o-mini model, and only the sender, the
subject and the first 600 characters of the body are sent to it. That costs very little per email.
Check OpenAI's pricing page for current rates before relying on any figure.

## Setup in brief

The setup guide linked above walks through every step. In short:

1. Import `Broker_Inbox_Triage_TEMPLATE.json` into n8n, under Workflows then Import from File.
2. In Gmail, create eight labels under `Broker/`, one for each category, for example
   `Broker/renewal`.
3. Find the ID of each label and paste it into the **Settings** node in place of the eight
   `PASTE_..._LABEL_ID` placeholders. A label ID is not the label name; it looks like
   `Label_1234567890`. The setup guide shows the quickest way to find them.
4. Create a Google Sheet with a tab named `Log` and these column headings in the first row: Date,
   From, Subject, Category, Urgent, Summary. Paste the sheet's ID into **Settings** in place of
   `PASTE_YOUR_GOOGLE_SHEET_ID_HERE`. The ID is the long string in the sheet's web address.
5. Put the address that should receive urgent alerts into **Settings** in place of
   `PASTE_YOUR_ALERT_EMAIL_HERE`. Read the note on this under Known limitations first.
6. Connect your accounts: Gmail on the email nodes, Google Sheets on **Log to sheet**, and your
   OpenAI key on **Classify**, as a header credential with `Authorization` set to
   `Bearer YOUR_KEY`.
7. Turn the workflow on. It arrives switched off and checks for new mail once a minute.

Before it reads anything, the workflow checks whether an email already carries one of your eight
labels and skips it if so, so nothing is handled twice.

## Known limitations

Stated plainly, so nothing surprises you:

- **Which emails get a draft is decided by the instructions to the model, not by a separate check
  in the workflow.** The model is told to leave the draft empty for every category except new
  enquiries and certificate requests, and the workflow saves whatever draft it is given. If the
  model ever got that wrong, a draft could appear on another kind of email. It would still only be
  a draft, never sent, but check drafts before sending them.
- **"Urgent" is not strictly defined.** The model decides what counts as urgent from the email
  itself. Expect it to flag some things you would not, and to miss some you would. Treat alerts as
  a nudge, not a guarantee.
- **The alert address must be a different address from the inbox being watched.** If alerts are
  sent to the same inbox, each alert arrives as a new email, gets read, may be judged urgent, and
  sends another alert, round and round.
