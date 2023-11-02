Command line Pushover client
============================
This is a full-featured (implementing virtually all of the pushover.net API) pushover client written in bash.

I've seen various clients for the command line in various states of development and API support, but to my eye this is the best client available that
is written in bash - it will run with no or little modification on any system with `bash`, `curl` and the normal *nix tools available.

A [pushover.net](https://pushover.net/) account, user key and application API token are required to use the client.


Usage
=====
Configuration options for `pushover-client` can be provided in a (user specifiable or default) configuration file, given on the command line, or a mixture
of both.

The basic usage of the pushover client is: `pushover-client [options] [config file]`.

There are three **required** options that must be given in either the default or user specified custom config file, or on the command line:
  1. The user key(s) (config file: `USER_KEYS`, command line:` -u|--user`).
  2. The "application" API token (config file: `TOKEN`, command line: `-t|--token`).
  3. A plain text (config file: `MESSAGE`, command line: `-m|--message`) or HTML (config file: `HTML_MESSAGE`, command line: `-M|--html-message`) message to send as the alert.

All other (config file or command line) options are optional.

I recommend creating a config file (or use the default) to contain the "fixed" options (such as the user key(s) and API token) and use command line options
to specify the "dynamic" options (such as the message/alert to be sent).

Config files
------------
Config files can be used to define "usages" or alert types for individual circumstances.  Specifying the majority of "fixed" configuration options in the
config file makes the command line of `pushover-client` cleaner and easier to use.

A properly configured config file specified on the command line can be the only item required to send an alert.

To use only a config file, the command line would be: `pushover-client <config file>`.

If the `<config file>` is an absolute path, only that file is used as the basis for the config options, and it is an error if it does not exist.

An unqualified or non-absolute `<config file>` is searched for in the following locations (in order):
  1. Relative to the current directory.
  2. Relative to the user's personal configuration directory at: `~/.pushover-client`.
  3. Relative to the system wise configuration directory at: `/etc/pushover-client`.

If no `<config file>` is given on the command line, a `default` config file is searched for in and read from the following locations in order, even if all
options are given on the command line:
  1. A user's personal configuration directory at: `~/.pushover-client`.
  2. The system wide configuration directory at: `/etc/pushover-client`.

This allows the `default` file to configure the "fixed" options for all usage that does not specifically override those options on the command line.

It is **not** an error if no `<config file>` is given on the command line and the `default` configuration file does not exist, but some options will be
required on the command line.

Configuration options
---------------------
`-a`, `--attachment <filename>`
Config file: `ATTACHMENT=""`.
  * The picture to send with the alert.  By default, no picture is sent.

`-A`, `--api-url <url>`
Config file: `API_URL=""`.
  * The API URL to use for this submission.  The Default is coded into the `pushover-client` script and should not need to be changed.
    Quote `<url>` if it contains shell special chars.

`-c`, `--callback <url>`
Config file: `CALLBACK_URL=""`.
  * A URL which is accessed by API server when the user acknowledges the alert.  This option is only used in priority 2 alerts.
    Quote `<url>` if it contains shell special chars.

`-d`, `--devices <devices>`
Config file: `DEVICES=""`.
  * A comma seperated list of the devices to receive the alert.  The default is to send to all devices.

`-e`, `--expiry <seconds>`
Config file: `EXPIRY=""`,
  * Set the expiration time for alerts sent with priority 2.  The default is 3600 (1 hour).  The maximum expiry time is 10800 (3 hours).

`-m`, `--message <text>`
Config file: `MESSAGE=""`.
  * The plain text message to send as the alert.
    Quote `<text>` if it contains spaces.
    This option or `-M` (config file: `HTML_MESSAGE=""`) is required.

`-M`, `--html-message <text>`
Config file: `HTML_MESSAGE=""`.
  * The HTML message to send as the alert.  Only basic text formatting is supported.
    Quote `<text>` if it contains spaces.
    This option or `-m` (config file: `MESSAGE=""`) is required.

`--monospace`
Config file: `MONOSPACE=""`.
  * Use a monospace font to display the message given with -m.  The default is to use regular (variable width) font.
    This option cannot be used with `-M`.

`-p`, `--priority <priority>`
Config file: `PRIORITY=""`.
  * Set the priority of the message:
      `-2` * Lowest priority - no alert/notification will be generated on the device.  However, the app badge or number will update on devices.
      `-1` * Low priority - no alert sound is emitted but a notification will appear.
             During a user's configured quiet hours, priority -1 is used for messages.
      `0` * Normal priority (the default) - an alert sound and notification are generated.
      `1` * High priority - bypass the user's configured quiet hours and generate an audible alert and notification.
      `2` * Emergency - as per priority 1, but the alert and notification is repeated (subject to the `-r` (config file: `RETRY=""`) and `-e` (config file:
            `EXPIRY=""`) options) until it is acknowledged by the recipient.

`-q`, `--quiet`
Config file: `QUIET=""`.
  * Do not print the API execution reply to stdout.

`-r`, `--retry <seconds>`
Config file: `RETRY=""`.
  * Set the retry interval for alerts sent with priority 2.  The default is 60 (1 minute).  The minimum retry time is 30 seconds.

`-s`, `--subject <text>`
Config file: `SUBJECT=""`.
  * The subject/title of the message.  If unset, the configured app name is used.
    Quote `<text>` if it contains spaces.

`-S`, `--sound <sound>`
Config file: `SOUND=""`.
  * Set the alert sound to play with message:
      `none ` * None/silent.
      `vibrate` * Vibrate only.
      `pushover` * Pushover (short tone, the default).
      `bike` * Bike (short tone).
      `bugle` * Bugle (short tone).
      `cashregister` * Cash Register (short tone).
      `classical` * Classical (short tone).
      `cosmic` * Cosmic (short tone).
      `falling` * Falling (short tone).
      `gamelan` * Gamelan (short tone).
      `incoming` * Incoming (short tone).
      `intermission` * Intermission (short tone).
      `magic` * Magic (short tone).
      `mechanical` * Mechanical (short tone).
      `pianobar` * Piano Bar (short tone).
      `siren` * Siren (short tone).
      `spacealarm` * Space Alarm (short tone).
      `tugboat` * Tug Boat (short tone).
      `alien` * Alien Alarm (long tone).
      `climb` * Climb (long tone).
      `persistent` * Persistent (long tone).
      `echo` * Pushover Echo (long tone).
      `updown` * Up Down (long tone).
      Or a sound uploaded to the user's account.

`-t`, `--token <token>`
Config file: `TOKEN=""`.
  * The pushover.net API token/key for the specific application.  This option is required.

`-T`, `--ttl <seconds>`
Config file: `TTL=""`.
  * The number of seconds the alert will live (or be displayed) on a users device before being automatically removed.  The default is no ttl.
    This option is ignored when alerts are sent with priority (-p) 2.

`--timestamp <seconds>`
Config file: `TIMESTAMP=""`.
  * The number of seconds since the unix epoch to use as the timestamp for the alert.  The default timestamp is the time the API received the message.

`-u`, `--user <user_key>`
Config file: `USER_KEYS=""`.
  * The pushover.net user key(s).  If a single user key is specified, that account's configuration will be used for the alerts.  If a comma separated list
    (maximum 50) of user keys is given, the alert is sent only to those users.  This option is required.

`-U`, `--url <url>`
Config file: `URL=""`.
  * Set the URL to send with the alert.
    Quote `<url>` if it contains shell special chars.

`--url-title <text>`
Config file: `URL_TITLE=""`.
  * The title of the URL given with -U.  Ignored if -U is not used also.
    Quote `<text>` if it contains spaces.

Option processing ceases with the first non-option argument, or "--".

Error/Return Codes
------------------
`pushover-client` will return an error code depending upon the success or failure of submitting the alert request to the pushover API.  
The return codes are as follows:

* **0** - Successfully submitted the request to the pushover API.  
          If `-q|--quiet` was not specified, the JSON emitted by the API server is displayed on `stdout`.
* **1** - Error in usage.  The usage error is reported to `stderr`.
* **2** - The API server returned a non-success status code - the alert was not sent.  
          If `-q|--quiet` was not specified, the JSON emitted by the API server is displayed on `stderr` along with the error notifcation.
* **3** - There was a problem accessing the API server - the alert was not sent.


Examples
========
In the following examples, `<user key>` is the pushover.net user key(s), `<API token>` is the application API token and `<config file>` is the configuration
file name.

Command line examples:
  * Send a plain text message to all devices, showing all required [options].
      `pushover-client -u <user key> -t <API token> -m "Test message"`

  * Same as the above, but do not show any response from the API server (an appropriate error/return code is set).
      `pushover-client -q -u <user key> -t <API token> -m "Test message"`

  * Send a HTML message (which will be underlined) to all devices, showing all required [options].
      `pushover-client -u <user key> -t <API token> -M "<u>Test message</u>"`

  * Send a plain text message to all devices, with the addition of a subject/title to the message.
      `pushover-client -u <user key> -t <API token> -s "Test subject" -m "..."`

  * Send a message to all devices, including a picture attachment.
      `pushover-client -u <user key> -t <API token> -a </path/to/image> -m "..."`

  * Send a message to all devices, including a supplimentary URL with a title.
      `pushover-client -u <user key> -t <API token> -U "https://afterdark.org.uk" --url-title "Afterdark" -m "Check out the included URL!"`

  * Send a message to all devices, selecting an alternative sound alert.
      `pushover-client -u <user key> -t <API token> -s "magic" -m "..."`

  * Send a message to a specific list of devices.
      `pushover-client -u <user key> -t <API token> -d "iphone,ipad" -m "..."`

  * Send an emergency alert, repeated every minute for an 2 hours until it is acknowledged by the user.
      `pushover-client -u <user key> -t <API token> -p 2 -r 60 -e 7200 -m "..."`

Read all configuration from the given <config file> path and use that for sending the alert.
If this path does not exist, search for a file matching the <config file> name in the user's private pushover directory (~/.pushover) or the system wide
pushover directory (/etc/pushover), in that order.
  `pushover-client <config file>`
