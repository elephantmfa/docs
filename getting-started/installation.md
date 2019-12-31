# Installation

## Table of Contents

- [Introduction](#introduction)
  - [Server Requirements](#server-requirements)
  - [Installing Elephant](#installing-elephant)
  - [Configuration](#configuration)
- [Server Configuration](#server-configuration)

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

[1]: https://laravel.com/docs/6.x/installation#server-requirements
[ext-event]: https://pecl.php.net/package/event
[ext-ev]: https://pecl.php.net/package/ev
[ext-uv]: https://pecl.php.net/package/uv
