# Mail Log Plugin

Log every email your [October CMS](https://octobercms.com) site sends — with delivery status, open tracking and
resend.

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin  
**Login:** maillog  
**Password:** maillog

Check whether an important email went out, or simply confirm that mail is going out at all.

## Features

* Log all outgoing emails, with a cap on the size kept
* Delivery status: sent, failed with the recorded error, or pending
* Resend a logged message to its original recipients
* Optional storage of sent attachments, with download and resend
* Skip the body of chosen templates, so a password reset link never lands in the log
* Open tracking
* Mail Activity dashboard widget
* CSV and JSON export
* Automatic pruning of old logs
* Fine-grained permissions
* Works out of the box

## Requirements

This plugin requires October CMS 4.x and PHP 8.2 or newer.

If you are running October CMS 3.x or older, stay on version 1.0.6 of this plugin.

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else that looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to
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

A message sent without an HTML part is stored as its plain text, which the preview shows and a resend delivers as
preformatted HTML. The size cap is applied to that text before it is escaped, so the stored copy can come out slightly
larger than the cap. A body supplied as a stream is not stored, since reading it would empty the message on its way
out; the preview says so.

## Settings

Settings -> Logs -> Mail Log Settings.

### Prune period in days

Records older than this are deleted once a day by a scheduled task. The default is 30 days; `0` or empty keeps
records forever. Logged messages hold mail bodies and addresses, so keeping them indefinitely should be a deliberate
choice; see [Personal data](#personal-data).

> **Important note:** For scheduled tasks to operate correctly you must set up the
> scheduler: https://docs.octobercms.com/4.x/setup/scheduler.html

### Track when a recipient opens an email

Adds an invisible image to every HTML message to record when it is opened. Enabled by default.

Open tracking is a best-effort signal: many mail clients block remote images, and privacy proxies fetch them without
the recipient reading the message. When tracking is off, opens of messages sent earlier are no longer recorded, and
the open columns, the **Opened** filter and the **Opened** bar of the activity widget disappear.

The image URL of a logged message is rate limited to 30 requests a minute, so opens fetched through mail privacy relays
are recorded however large the send. Requests for any other address are limited to 600 a minute per IP address; behind
a proxy or CDN, configure trusted proxies, otherwise they all share the proxy's single limit.

### Maximum stored body size in MB

Caps the copy of the message body kept in the log; the message itself always goes out whole. The default is 1 MB,
`0` keeps every body in full. Templates that inline images as base64 write megabytes into the log on every send,
which is what this limit is for.

A truncated message cannot be resent, and the preview says so.

### Templates whose content is not stored

Messages built from the templates named here are logged with their recipients, subject and status, but without their
content. Use it for mail carrying a secret, such as the password reset link. This applies to plain-text messages too.
Such a message cannot be resent.

The following templates are always logged this way, whether or not they are listed, and every run of four or more
digits in their subject is masked, so a one-time code does not reach the log either:

* `backend:restore` and `backend:invite` (backend password reset and invitation)
* `user:recover_password`, `user:invite_email` and `user:verify_email` (RainLab.User)
* `renatio.twofactorauth::mail.code` (Renatio.TwoFactorAuth verification code)

Templates added to the list get the same subject masking.

### Store attachments

Keeps a copy of every attachment sent, so it can be downloaded from the preview and included in a resend. Disabled by
default.

Copies are written outside the web root, under `storage/app/uploads/protected`, and can only be downloaded by users
with the `renatio.maillog.manage_logs.attachments` permission; everyone else sees the file names as plain text. Files
are stored only once the mail transport has accepted the message, and images embedded with `cid:` are not stored.
Attachments of templates whose content is not stored are not kept either; only their names are logged.
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
The button is not shown for a message whose body was not stored or was truncated.

Images embedded with `cid:` (such as a logo added with `$message->embed()`) are not stored, so a resent message shows
them as broken; the confirmation says so before anything is sent.

The resent message is logged as a new record that links back to the original and names who resent it; the original
lists every resend.

## Filters

* **Template** - the mail template the message was built from
* **Sent** / **Failed** / **Pending** - delivery outcome
* **Opened** - opened at least once (hidden when open tracking is off)
* **Created at** - a date range

The **Template** filter lists the templates that appear in the log.

The **From**, **Reply to**, **IP address**, **Template**, **First opened** and **Last opened** columns are hidden by
default; turn them on with the list setup button.

## Mail Activity dashboard widget

Add it on the dashboard with **Add widget** -> *Mail Activity*. It shows the total logged, sent, opened (while open
tracking is on) and failed messages for the last `days` days (7 by default). The figures are cached for one minute.

## Export

The **Export all** button in the list toolbar, available with the `renatio.maillog.manage_logs.export` permission,
exports the whole log to JSON (the default), CSV or CSV with custom delimiters, streamed in chunks so a large log does
not exhaust memory. The rows checked and the filters set on the list have no effect on it. The export includes message
bodies and every address.

## Permissions

| Permission                                    | Grants                                                   |
|-----------------------------------------------|----------------------------------------------------------|
| `renatio.maillog.manage_logs`                 | Access to the Mail Logs list and the dashboard widget.   |
| `renatio.maillog.manage_logs.preview`         | Opening the preview of a single logged message.          |
| `renatio.maillog.manage_logs.delete`          | Deleting selected records from the list.                 |
| `renatio.maillog.manage_logs.truncate`        | Emptying the whole log with the "Empty Mail Log" button. |
| `renatio.maillog.manage_logs.export`          | Exporting the log to CSV or JSON.                        |
| `renatio.maillog.manage_logs.resend`          | Resending a logged message from its preview.             |
| `renatio.maillog.manage_logs.attachments`     | Downloading a stored attachment from the preview.        |
| `renatio.maillog.manage_settings`             | Access to the Mail Log Settings page.                    |

`renatio.maillog.manage_logs` is required to reach the list at all; the other permissions gate individual actions.

## Personal data

For every message the log stores:

* the To, CC, BCC, From and Reply-To addresses, the subject and the template code;
* the body, up to the size cap, unless its template is one whose content is not stored;
* the attachment file names, and a copy of each attachment when **Store attachments** is on;
* the IP address of the request that sent the mail (the visitor submitting a form, for example; `127.0.0.1` for mail
  sent by a queue worker or from the command line);
* the delivery status and error, and the backend user who resent it;
* with open tracking on, the time of the first and last open and the number of opens. The IP address and client of
  whoever opens the message are not recorded.

Every logged message carries an `X-LOG-HASH` header visible to the recipient. With open tracking on, HTML messages also
carry an image loading the public URL `/track/opened/{hash}`. Where recipients have not agreed to be tracked, turn open
tracking off.

To keep less:

* set a short **Prune period in days**;
* list every template carrying a secret under **Templates whose content is not stored**;
* grant the export permission sparingly, since an export holds every body and address.

Records can be deleted from the list, all at once with **Empty Mail Log**, or by pruning; their stored attachments go
with them. Logged records cannot be edited.

## Troubleshooting

* **Opens are not recorded.** Open tracking must be on, and the message must have an HTML part. For mail sent by a
  queue worker or from the command line the image URL is built from `APP_URL`, which must be the public address of the
  site.
* **Records stay Pending.** See [Delivery status](#delivery-status).
* **Old records are not deleted.** Pruning runs from the scheduler, which must be set up; see
  [Prune period in days](#prune-period-in-days).
