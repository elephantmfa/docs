# Installation

## Table of Contents

- [Introduction](#introduction)
- [The Root Directory](#the-root-directory)
- [The App Directory](#the-app-directory)
  - [The `Mail` Directory](#the-mail-directory)

## Introduction

The default Elephant application structure is intended to provide a great
starting point for both large and small applications. But you are free to
organize your application however you like. Elephant imposes almost no
restrictions on where any given class is located - as long as Composer can
autoload the class.

For the most part, this follows the same structure as a Laravel application,
with a few exceptions explained below. Please reference the [laravel docs].

## The Root Directory

The root directory, unlike Laravel, *does not* contain a Routes folder nor a
public folder. This is because, unlike a web application, Elephant does not
interact via URLs, as it communicates SMTP and not HTTP.

## The App Directory

The App directory is relatively the same as a Laravel application, however the
mail directory is very different. And since Elephant doesn't speak HTTP, it is
missing the HTTP directory.

### The `Mail` Directory

The mail directory contains a `Filters` directory as well as a Kernel for
handling SMTP Mail processing. See the [filtering] documentation for more.

[laravel docs]: https://laravel.com/docs/6.x/structure#the-root-app-directory
[filtering]: /the-basics/filtering.md
