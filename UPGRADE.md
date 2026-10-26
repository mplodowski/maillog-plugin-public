# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

## Upgrading To 2.0.1

The plugin requires October CMS 4.x. If you are still running October CMS 3, stay on 1.0.6.

## Upgrading To 2.0.6

The controller action `previewEmail` was renamed to `preview_email`. Update any code that extends the `MailLogs`
controller or links to that action directly.

## Upgrading To 2.1.0

* Run `php artisan october:migrate`.
* Grant the new `utilities.mail_logs.resend` permission (`renatio.maillog.manage_logs.resend` since 3.0.0) to the
  roles that should be able to resend mail.
* If you serve a mirrored public folder, run `php artisan october:mirror`, or the Mail Activity widget renders
  unstyled.

## Upgrading To 2.2.0

Attachment storage is off by default. To use it, enable **Store attachments** in the settings and grant the new
`utilities.mail_logs.attachments` permission (`renatio.maillog.manage_logs.attachments` since 3.0.0) to the roles
that should be able to download stored files.

## Upgrading To 2.3.0

Run `php artisan october:migrate`.

Message bodies are now capped at 1 MB in the log. Set **Maximum message size in MB** to `0` to keep every body in
full, as before.

## Upgrading To 2.4.0

Run `php artisan october:migrate`.

## Upgrading To 3.0.0

Versions 3.0.0 to 3.1.1 required PHP 8.3; since 3.1.2 PHP 8.2 is enough.

Run `php artisan october:migrate`. It renames the permission codes as below and rewrites existing grants, so nobody
loses access; the permissions now sit on their own **Mail Logs** tab.

| Before                            | After                                       |
|-----------------------------------|---------------------------------------------|
| `utilities.mail_logs`             | `renatio.maillog.manage_logs`               |
| `utilities.mail_logs.preview`     | `renatio.maillog.manage_logs.preview`       |
| `utilities.mail_logs.delete`      | `renatio.maillog.manage_logs.delete`        |
| `utilities.mail_logs.truncate`    | `renatio.maillog.manage_logs.truncate`      |
| `utilities.mail_logs.export`      | `renatio.maillog.manage_logs.export`        |
| `utilities.mail_logs.resend`      | `renatio.maillog.manage_logs.resend`        |
| `utilities.mail_logs.attachments` | `renatio.maillog.manage_logs.attachments`   |
| `utilities.mail_logs_settings`    | `renatio.maillog.manage_settings`           |

Update any `hasAccess()` or `userHasAccess()` call in your own code that uses an old code.

## Upgrading To 3.1.0

Run `php artisan october:migrate`.

A resend is now logged as a record linked to its original, carrying who resent it, and the original lists every
resend. The `resent_at` column is dropped and the `resent_by` values held on originals are cleared, since earlier
resends cannot be matched to their originals.

## Upgrading To 3.1.1

Run `php artisan october:migrate`.

The `subject` and `from` columns are widened to `text`. On MySQL that rewrites the whole log table, so prune old
records first on a large installation.

## Upgrading To 3.2.0

Run `php artisan october:migrate`. The log is now read-only, so records can no longer be created or edited from the
backend form.

Password reset, invitation and verification code mail is now always logged without its body and with the digits of
its subject masked. Entries written by earlier versions still contain these links and codes, so review or delete the
existing log entries of the templates listed in the README, or empty the log.
