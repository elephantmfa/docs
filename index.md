## About Elephant Mail Filtering App (MFA)

ElephantMFA is a framework very tightly based on [Laravel](https://laravel.com),
however it is not used for HTTP requests, but SMTP requests.
In it's current state, it is meant to be used as a Content filter for the likes
of [Postfix](http://www.postfix.org/). It is designed to be a highly
customizable mail filter.

The reason that this was made was because similar mail filters out there all are
configured with configuration files and maybe minor hooks for adding custom
functionality. While this works well for ease of configuring the system, it
hinders one's ability to finaly tune for the exact configuration you need.
This would then result in extensive source diving and modification.

Here comes ElephantMFA, a highly extensible *framework* written in PHP using
ReactPHP, to write all of your own filtering code, to have it configured exactly
as you need it.

## Table of Contents
> **Note:** It's an important thing to note, that a lot of documentation will
> defer to or reference Laravel's excellent documentation as ElephantMFA
> utilizes their components.

- [Home](/)
- Getting Started
  - [Installation](getting-started/installation.md)
  - [Directory Structure](getting-started/directory-structure.md)
  - [Deployment](getting-started/deployment.md)
- The Basics
  - [Filters](the-basics/filters.md)
  - [Carrying Information](the-basics/carrying-information.md)
  - [SpamAssassin](the-basics/spamassassin.md)
  - [ClamAV](the-basics/clamav.md)
  - [Pattern Matching](the-basics/pattern-matching.md)

## Contributing

Thank you for considering contributing to the ElephantMFA framework!

### Current contributors:
 - [Marisa Clardy](https://clardy.eu)

## License

The ElephantMFA framework is open-sourced software licensed under the
[MIT license](https://github.com/elephantmfa/framework/blob/master/LICENSE).
