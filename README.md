# Mail Log Plugin

Log every email your [October CMS](https://octobercms.com) site sends — with delivery status, open tracking and
resend.

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin  
**Login:** maillog  
**Password:** maillog

Check whether a user received an important email, or simply confirm that mail is going out at all.

## Features

* Log all outgoing emails, with a cap on the size kept
* Delivery status: sent, failed with the recorded error, or pending
* Resend a logged message to its original recipients
* Optional storage of sent attachments, with download and resend
* Skip the body of chosen templates, so a password reset link never lands in the log
* Open tracking
* Mail Activity dashboard widget
* CSV export
* Automatic pruning of old logs
* Fine-grained permissions
* Works out of the box

## Requirements

This plugin requires October CMS 4.x and PHP 8.3 or newer.

If you are running October CMS 3.x or older, use the 1.x branch of this plugin.

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to
supporting and maintaining it. I ask my customers to do the same. I do not make money from this plugin by
advertisements, upgrades or additional services like hosting or setup.

Did you know that 30% of your purchase or donation goes to help fund the October Project?

My plugins take many hours to develop (40-120+) and even more hours to document and maintain. My paid plugins have to
pay for both this time, and the time I am spending on free plugins and less successful paid plugins. This means that it
will take even a successful plugin years to become profitable. Please consider buying an extended license if you want me
to continue to maintain these plugins for the very small fee I ask in return or hire me for adding functionality that
you feel is missing but valuable.

## Like this plugin?

If you like this plugin, give this plugin a like or make a donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/maillog-plugin-public/issues) to report any issues with
the plugin.

> Reviews should not be used for getting support or reporting bugs, if you need support please use the Plugin support
> link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan)
from [www.flaticon.com](https://www.flaticon.com/).

# Documentation

## Usage

The plugin works out of the box. It logs every mail sent from October CMS, including mail sent from other plugins and
from the queue.

The log lives in Settings -> Logs -> Mail Logs. Click a record to open its preview, which shows the message details and
renders the original message body.

## Settings

Settings -> Logs -> Mail Log Settings.

### Prune period in days

Records older than this are deleted once a day by a scheduled task. The default is 30 days; `0` or empty keeps
records forever. Logged messages hold the full mail body and the recipient IP address, so keeping them indefinitely
should be a deliberate choice.

> **Important note:** For scheduled tasks to operate correctly you must set up the
> scheduler: https://docs.octobercms.com/4.x/setup/scheduler.html

### Track when user opens email

Adds an invisible image to every HTML message to record when it is opened. Enabled by default.

Open tracking is a best-effort signal: many mail clients block remote images, and privacy proxies fetch them without
the recipient reading the message. When tracking is off, the open columns and the **Opened** filter disappear from the
list.

### Maximum message size in MB

Caps the copy of the message body kept in the log; the message itself always goes out whole. The default is 1 MB,
`0` keeps every body in full. Templates that inline images as base64 write megabytes into the log on every send,
which is what this limit is for.

A truncated message cannot be resent, and the preview says so.

### Templates whose content is not stored

Messages built from the templates named here are logged with their recipients, subject and status, but without their
content. Use it for mail carrying a secret, such as the password reset link. Such a message cannot be resent.

### Store attachments

Keeps a copy of every attachment sent, so it can be downloaded from the preview and included in a resend. Disabled by
default.

Copies are written outside the web root, under `storage/app/uploads/protected`, and can only be downloaded by users
with the `renatio.maillog.manage_logs.attachments` permission; everyone else sees the file names as plain text. Files
are stored only once the mail transport has accepted the message, and images embedded with `cid:` are not stored.
Stored files are removed together with their log record, whether it is deleted, emptied or pruned.

### Maximum attachment size in MB

Attachments larger than this are not stored; only their name stays in the log. The default is 10 MB, `0` stores
everything. The whole attachment is read into memory to be measured, so a generous limit needs a matching PHP
`memory_limit`.

## Delivery status

| Status  | Meaning                                                                                    |
|---------|--------------------------------------------------------------------------------------------|
| Sent    | The mail transport accepted the message.                                                   |
| Failed  | Sending threw an error; the error message is recorded on the record.                       |
| Pending | The outcome is unknown. The message was logged, but neither a send nor a failure followed. |

Pending is not a transient state: the message is logged just before the transport runs and marked as sent right
after, so a record that is still Pending after a few minutes should be treated as not sent. It stays Pending when:

* the process died between logging and sending — a fatal error, a PHP timeout, a killed queue worker;
* a `MessageSending` listener, or a `mailer.prepareSend` listener registered after the plugin, returned `false` and
  stopped the send;
* sending threw an exception the plugin could not attribute to the send, for example one wrapped in a custom
  exception without the original as `previous`;
* the application caught the exception without reporting it, so it never reached the exception handler where the
  plugin records failures. A caught exception passed to `report()` still marks the record as Failed;
* the message was delivered but marking the record as sent failed, for example on a lost database connection. The
  error is written to the application log and the send is not repeated.

The **Pending** filter lists such records.

Logging never blocks delivery. When the log row itself cannot be written, the message is still sent, nothing is
recorded for it, and the reason is written to the application log.

> **Note:** A queued mailable is logged again on every attempt, so a message that fails twice before finally being
> delivered leaves two failed rows plus one sent row.

## Resend

The preview has a **Resend** button for users with the `renatio.maillog.manage_logs.resend` permission. It sends the
stored body and subject to the original To, CC and BCC recipients, with the original reply-to address and the stored
attachments where available, using the mail configuration currently in effect. The original sender is not reused.

The resent message is logged as a new record that links back to the original and names who resent it; the original
lists every resend.

## Filters

* **Template** - the mail template the message was built from
* **Sent** / **Failed** / **Pending** - delivery outcome
* **Opened** - opened at least once (hidden when open tracking is off)
* **Created at** - a date range

The **From** and **Last Opened** columns are hidden by default; turn them on with the list setup button.

## Mail Activity dashboard widget

Add it on the dashboard with **Add widget** -> *Mail Activity*. It shows the total logged, sent, opened and failed
messages for the last `days` days (7 by default). The figures are cached for one minute.

## Export

The **Export all** button in the list toolbar, available with the `renatio.maillog.manage_logs.export` permission,
exports the whole log to CSV, streamed in chunks so a large log does not exhaust memory. The rows checked and the
filters set on the list have no effect on it.

## Permissions

| Permission                                    | Grants                                                   |
|-----------------------------------------------|----------------------------------------------------------|
| `renatio.maillog.manage_logs`                 | Access to the Mail Logs list and the dashboard widget.   |
| `renatio.maillog.manage_logs.preview`         | Opening the preview of a single logged message.          |
| `renatio.maillog.manage_logs.delete`          | Deleting selected records from the list.                 |
| `renatio.maillog.manage_logs.truncate`        | Emptying the whole log with the "Empty Mail Log" button. |
| `renatio.maillog.manage_logs.export`          | Exporting the log to CSV.                                |
| `renatio.maillog.manage_logs.resend`          | Resending a logged message from its preview.             |
| `renatio.maillog.manage_logs.attachments`     | Downloading a stored attachment from the preview.        |
| `renatio.maillog.manage_settings`             | Access to the Mail Log Settings page.                    |

`renatio.maillog.manage_logs` is required to reach the list at all; the other permissions gate individual actions.
