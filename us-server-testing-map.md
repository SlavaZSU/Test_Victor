# Manual testing map

## Administration

- [x] Change language.
- [x] Rebuild. 

## Entities

### Account

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Contact

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Lead

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Lead Convert.

### Opportunity

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Create Document from Bottom Panel.
- [x] Select Document from Bottom Panel.

### Case

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Email Template

- [x] Creating.
- [x] Updating.
- [x] Remove from List View.
- [x] Remove from Detail View.
- [x] Duplicate.

### Email Filter

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Apply for imported emails (Personal Email).
- [x] Apply for imported emails (Group Email).

### Email Folder

- [x] Creating.
- [x] Updating.
- [x] Removing from Detail View.
- [x] Duplicating.

### Group Email Folder

- [x] Creating.
- [x] Updating.
- [x] Removing from Detail View.
- [x] Duplicating.

### Personal Email Account

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Group Email Account

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Target List

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Populate the target list with leads using 'select all results'.

### Calendar

- [x] Create events.
- [x] A shared view.
- [x] Timeline.

### Meeting

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Send invitations.
- [x] Popup reminder.
- [x] Email reminder.

### Call

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Task

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Campaign

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Knowledge Base

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Send in email.

### Document

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Creating by drag-n-drpping file on the list view.

### User

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

### Team

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.
- [x] Assign a team position to a user (Position List field).

### PDF Template

- [x] Creating.
- [x] Updating.
- [x] Removing from List View.
- [x] Removing from Detail View.
- [x] Duplicating.

## Features

### Personal Email Account

- [x] Create account with IMAP and SMTP.
- [x] Check emails are received and available in the user's INBOX.
- [x] Test email sending from the account.

### Group Email Account

- [x] Create account with IMAP and SMTP.
- [x] Check emails are received.
- [x] Test email sending from the account.
- [x] Email-to-Case.

### Email

- [x] Apply email template, test placeholders.
- [x] Save draft and send.
- [x] Remove email.

### Stream

- [x] Post on record.
- [x] Post attachments.
- [x] Update post.
- [x] Remove post.
- [x] Post to another user.
- [x] Post to team.

### Portal

- [x] Create a portal, with portal role, portal user, layout set.
- [x] Check whether ACL is applied for the portal user.
- [x] Check whether layout set is applied for a logged portal user.

### 2-factor authentication

- [x] Set up TOTP 2FA for a user and login under that user.

### Passwords

- [x] An admin creates a user (with an email address non-empty), password is empty, *Send Email with Access Info to User* is checked. Check a received access email. Try to login under the new user.
- [x] An admin changes a password to another user (on the User edit view). 
- [x] An admin changes a password to another user with *Send Email with Access Info to User* checked (the user should have email address). Check a received email.
- [x] A regular user changes own password (from the dropdown on the User detail view).
- [x] Send password change link for another regular user (under the admin from the dropdown on the User detail view).
- [x] Send password change link for a portal user.
- [x] Forgot password from the login page. Recover access for a user.

### Import

- [x] Import CSV with email address, phone number.
- [x] Import Create Only.
- [x] Import Create and Update.
- [x] Check duplicate detection.
- [x] Import invalid email, check the record is available in the Errors panel.
- [x] Import in idle.
- [x] Import by command line (manully).
- [x] Import with same params.
- [x] Revert import.

### Currency

- [x] Currency converting.

### Entity Manager

- [x] Create a custom entity.
- [x] Edit entity.
- [x] Edit formula.
- [x] Create custom field.
- [x] Edit custom field.
- [x] Delete custom field.
- [x] Delete custom entity.

### Layout sets

- [x] Create a layout set.
- [x] Apply the layout set to a team and test it's working for a logged user (the team should be a default team for the user).

### Lead Capture

- [x] Set up lead capture without opt-in confirmation, send API request, check lead is created.
- [x] Set up lead capture with opt-in confirmation, send API request, check whether works properly.
- [x] Check duplicate detecting works properly.

### Mass Email

- [x] Send mass email.
- [x] Tracking URL in mass email.
