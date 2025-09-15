# N8N-Email-Classifier
<img width="1019" height="312" alt="image" src="https://github.com/user-attachments/assets/6cfc7956-aa66-4db6-bed2-0ff435593efb" />

### Features
- Polls Gmail for unread messages on a schedule.
- Extracts sender name robustly; gracefully falls back when missing.
- Classifies email text into one of several categories via an LLM chain.
- Applies corresponding Gmail labels to the email by messageId.

## Workflow overview

Main nodes and flow:
- Gmail Trigger: polls unread messages every minute.
- Information Extractor: uses an LLM extractor to parse senderName from email metadata.
- IF: checks whether senderName is present.
  - Name Found: sets greeting to “Dear {senderName}”.
  - No Name: sets greeting to “Hey,”.
- Merge: combines branches to continue a single path.
- Edit Fields: assembles introduction, emailBody, messageId, threadId.
- Text Classifier: classifies emailBody into Kaggle, Daily Dose of DS, Udemy, or Miscellaneous.
- Gmail: four addLabels operations route messages to label IDs per category.

Input and output signals:
- Inputs: Gmail unread emails (subject, text, from, ids).
- Outputs: Gmail labels added to the message based on classification.

## Categories and labels

Configured categories in classifier:
- Kaggle: general Kaggle-related emails.
- Daily Dose of DS: AI/DS update content.
- Udemy: training/course-related emails.
- Miscellaneous: fallback.
  
Label mapping by node:[1]
- Add label to message → Label_745819522269243592 (e.g., Kaggle).
- Add label to message1 → Label_5752208218461080031 (e.g., Daily Dose of DS).
- Add label to message2 → Label_2303365145849676185 (e.g., Udemy).
- Add label to message3 → Label_4161135832027494249 (e.g., Miscellaneous).

Note: Replace these labelIds with workspace-specific Gmail label IDs and keep names in sync.

## Prerequisites

- n8n instance (hosted or self-managed) with access to the Internet.
- Gmail account with OAuth2 credentials configured in n8n.
- OpenAI API credentials configured in n8n for LLM nodes.

## Installation

- Export or copy the JSON workflow into n8n via Import from File/Clipboard.
- Verify node credentials mappings:
  - Gmail Trigger and Gmail nodes use the gmailOAuth2 credential.
  - LLM nodes use the openAiApi credential.
- Update labelIds to match the Gmail labels in the target account.

## Configuration

- Trigger: set poll schedule (everyMinute by default).
- Filters: unread email filter is set; adjust if needed.
- Information Extractor: system prompt enforces strict attribute extraction for senderName.
- Text Classifier:
  - Categories and descriptions are defined inline; edit to fit needs
  - System prompt requests a pure JSON output; auto-fixing is disabled by default.
- Gmail label nodes: ensure messageId is passed correctly and labelIds exist.

## Running

- Activate the workflow in n8n; it will poll Gmail and label unread emails automatically.
- Use n8n’s Executions view to observe runs and debug.

## Technical requirements

- n8n version supporting: Gmail Trigger, Gmail, IF, Set, Merge, and LangChain LLM nodes.
- Credentials:
  - Gmail OAuth2 with appropriate scopes to read and label messages.
  - OpenAI API key with access to gpt-3.5-turbo or configured model.
- Network: outbound HTTPS to Gmail and OpenAI endpoints.
- Data volume: polling every minute; consider Gmail API quotas and n8n execution retention settings.

## Customization

- Add more categories and connect additional Gmail label nodes.
- Change the greeting logic or enrich messages with more extracted attributes.
- Add logging or persistence using n8n Database nodes or external storage.

## Troubleshooting

- Missing senderName: branch falls back to “Hey,”; verify extractor prompt and email header structure.
- Misclassification: refine category descriptions or enable auto-fixing; optionally swap to a different model.
- Gmail label errors: ensure labelIds exist and the OAuth2 credential has proper scopes.
- Rate limits: adjust poll frequency and ensure batching where possible.

## Security and privacy

- Emails may contain sensitive data; limit stored execution data and mask logs.
- Store OpenAI and Gmail credentials in n8n’s credential vault; avoid hardcoding secrets.




