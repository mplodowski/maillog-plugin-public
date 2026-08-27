# Upgrade guide

## Upgrading To 2.0.0

The plugin requires October CMS 4.x. If you are still running October CMS 3, stay on the 1.x branch.

## Upgrading To 2.0.6

The controller action `previewEmail` was renamed to `preview_email`. Update any code that extends the `MailLogs`
controller or links to that action directly.

## Upgrading To 2.1.0

* Run `php artisan october:migrate`.
* Grant the new `utilities.mail_logs.resend` permission to the roles that should be able to resend mail.
* If you serve a mirrored public folder, run `php artisan october:mirror`, or the Mail Activity widget renders
  unstyled.

## Upgrading To 2.2.0

Attachment storage is off by default. To use it, enable **Store attachments** in the settings and grant the new
`utilities.mail_logs.attachments` permission to the roles that should be able to download stored files.

## Upgrading To 2.3.0

Run `php artisan october:migrate`.

Message bodies are now capped at 1 MB in the log. Set **Maximum message size in MB** to `0` to keep every body in
full, as before.

## Upgrading To 2.4.0

Run `php artisan october:migrate`.

## Upgrading To 3.0.0

The plugin requires PHP 8.3 or newer; upgrade the runtime before running Composer.

The permission codes changed, so run `php artisan october:migrate`. The migration rewrites existing grants, so nobody
loses access. The permissions also moved from the **Utilities** tab into their own **Mail Logs** tab.

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

If your own code calls `hasAccess()` or `userHasAccess()` with one of the old codes, update those calls.
