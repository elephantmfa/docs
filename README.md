<p align="center"><a href="https://elephantmfa.app"><img width="150px" height="150px" src="images/logo.svg"></a></p>
<p align="center">
<a href="https://packagist.org/packages/elephantmfa/framework"><img alt="Total Downloads" src="https://poser.pugx.org/phpunit/phpunit/downloads"></a>
<a href="https://packagist.org/packages/elephantmfa/framework"><img alt="Latest Stable Version" src="https://poser.pugx.org/elephantmfa/framework/version"></a>
<a href="https://packagist.org/packages/elephantmfa/framework"><img alt="License" src="https://poser.pugx.org/elephantmfa/framework/license"></a>
</p>

## About Elephant Mail Filtering Application (MFA)
---

ElephantMFA is a framework very tightly based on [Laravel](https://laravel.com), however it is not used for HTTP requests, but SMTP requests.
In it's current state, it is meant to be used as a Content filter for the likes of [Postfix](http://www.postfix.org/). It is designed to be a highly customizable mail filter.

The reason that this was made was because similar mail filters out there all are configured with configuration files and maybe minor hooks for adding custom functionality. While this works well for ease of configuring the system, it hinders one's ability to finaly tune for the exact configuration you need. This would then result in extensive source diving and modification.

Here comes ElephantMFA, a highly extensible *framework* written in PHP using ReactPHP, to write all of your own filtering code, to have it configured exactly as you need it.

## Table of Contents

* [Home](README.md)

## Contributing

Thank you for considering contributing to the ElephantMFA framework!

### Current contributors:
 * [Marisa Clardy](https://clardy.eu)

## License

The ElephantMFA framework is open-sourced software licensed under the [MIT license](LICENSE.md).
