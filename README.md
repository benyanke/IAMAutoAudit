# IAMaudit - An IAM Account Security Auditor

A tool designed to allow a user to audit IAM users and keys, ensuring credentials are being propery rotated. Designed to be run in Lambda, but could be run anywhere.

**This is purely an idea at this point - there has been no implementation yet**

This tool is runnable in two main modes:

## Audit Mode
Audit mode simply prints a report of all users to an alert channel. It lists both users who have action required (a key too old, for example),
as well as well as users who exist in a good state.

## Alert Mode
Alert mode sends alerts to specified endpoints on program run, both catchall alert addresses, as well as user-specific addresses (alerting a user when they need to change their IAM keys, for example).

Alert channels include:

 * Arbitrary Webhooks - integration with external systems
 * SMTP mail address - shared address
 * SMTP mail address - individual alerts to an email based on IAM account username
 * Slack alerts - shared group
 * Slack alerts - individual

## Checks Implemented

IAM Key Checks:
 * Key Age - tracks keys which have a creation date older than xxx days
 * Key Inactive - tracks keys which appear not to be used

IAM User Checks:
 * User Password Age - tracks which passwords are older than xxx days
 * User Inactive - tracks users who have not logged into the console or used keys in xxx days
 * User MFA Enabled - Ensures users in specified groups have MFA enabled
 * Direct attach policy - Ensures all policies are granted via groups, not directly to users (optional but best practice)

Additional settings:
 * Ignore Users With No Permissions - ignores users with no permissions granted (ie - they are effectively disabled)
 * Alert Delay - Don't send alerts more than every xx days
 * Email tag name - name of the tag to check for the user's email for alerting

## Implementation Notes

### Steps Required To Check Key Age

**List Users**

`aws iam list-users`

**List Keys By User**

Loop through users found above.

`aws iam list-access-keys --user-name "$USER"`



### Bash example

Listing all keys:

```
for u in `aws iam list-users | jq --raw-output '[.Users[].UserName] | join(" ")'` ; do
  echo "";
  echo "User $u keys:" ;
  aws iam list-access-keys --user-name "$u" | jq '.AccessKeyMetadata' ;
done;
```
