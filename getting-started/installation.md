# Installation

## Table of Contents

- [Installation](#installation)
  - [Server Requirements](#server-requirements)
  - [Installing Elephant](#installing-elephant)
  - [Configuration](#configuration)
- [Server Configuration](#server-configuration)
  - [Routing Postfix to and from Elephant](#routing-postfix-to-and-from-elephant)

## Installation

### Server Requirements

The Elephant Framework has a few system requirements. In addition to all of the
[server requirements that Laravel requires][1], Elephant has the following
requirements:

- ext-sockets

Additionally, it is recommended that you have one of the following installed:

- [ext-uv]
- [ext-ev]
- [ext-event]

These will provide much higher performance on the event loop.

Lastly, the following software is recommended for properly utilizing Elephant.

- Postfix
- SpamAssassin
- ClamAV


### Installing Elephant

Elephant utilizes [Composer](https://getcomposer.org/) to manages
it's dependencies. Please verify that Composer is installed prior to
using Elephant.

To install, you can create the project utilizing composer, running the
Composer `create-project` command in your terminal.

```sh
composer create-project --prefer-dist elephantmfa/elephantmfa mail-filter
```

### Configuration

#### Configuration Files

All of the configuration files for the Elephant framework are stored in the
config directory. Each option is documented, so feel free to look through the
files and get familiar with the options available to you.

#### Directory Permissions

After installing Laravel, you may need to configure some permissions.
Directories within the storage and the bootstrap/cache directories should be
writable by Elephant, or Elephant won't run.

## Server Configuration

### Routing Postfix to and from Elephant

It is recommended to have Postfix (or similar MTA) to handle external mail.
In the current version of Elephant, it is not recommended to have Elephant be
the external SMTP relay, as many necessary inter-server communication features
aren't implemented. As such, Postfix will handle external connections for mail.

To route mail from Postfix to Elephant, add a content-filter option either
globally, or (more recommended) in `master.cf` and forward the inbound mail
(port 25) to an inbound filtering port in Elephant, and the outbound mail (port
587) to an outbound filtering port in Elephant. Assuming these ports are named
properly, you can easily set it up to filter mail differently between inbound
and outbound mail.

Additionally, it is recommended to have an ingress port in Postfix to route the
mail back to. While you can have your routing logic in Elephant, if you need to
link to any other services, linking through Postfix would be recommended.
Create a port in `master.cf` like so:

```
10030      inet  n       -       n       -       -       smtpd
    -o additional_configuration_here...
```

Then configure Elephant to route to that port.

For more complex configuration of Postfix, please review the Postfix
documentation.

[1]: https://laravel.com/docs/6.x/installation#server-requirements
[ext-event]: https://pecl.php.net/package/event
[ext-ev]: https://pecl.php.net/package/ev
[ext-uv]: https://pecl.php.net/package/uv
