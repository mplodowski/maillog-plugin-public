# Mail Log Plugin

View mail messages with their recorded time and details.

Plugin automatically logs all outgoing emails send through your October CMS project after installation. This is very useful when you need to check user received important email or just check if emails are send successfully.

## Features

* Log all outgoing emails
* Track user open clicks
* Automatically prune old mail logs
* Work out of the box

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

If you like this plugin, give this plugin a Like or Make donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/maillog-plugin-public/issues) to report any issues
with plugin.

> Reviews should not be used for getting support or reporting bugs, if you need support please use the Plugin support
> link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan)
from [www.flaticon.com](https://www.flaticon.com/).

# Documentation

## Usage

Plugin will work out of the box after installation. It will log all mail send from October CMS.

## Settings

Plugin register new Settings menu item in Settings -> Logs -> Mail Log Settings

### Prune period in days

This will prune records older than specified number of days. The default value is 30 days.

> **Important note:** For scheduled tasks to operate correctly you must set up the
> scheduler: https://docs.octobercms.com/3.x/setup/scheduler.html

### Track open clicks

When this is feature is enabled, plugin will add invisible image to sent email to track when user opens it.

This is enabled by default.
