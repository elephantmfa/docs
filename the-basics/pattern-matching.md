# Pattern Matching

## Table of Contents

- [Introduction](#introduction)
- [Regular Expression Matching](#regular-expression-matching)
    - [Available methods](#available-methods)
    - [Regex example usage](#regex-example-usage)
- [Wildcard Matching](#wildcard-matching)
    - [Wildcard special characters](#wildcard-special-characters)
    - [Wildcard example usage](#wildcard-example-usage)

## Introduction

The most important thing for spam detection is matching against strings to
detect patterns of spammy or phishing behaviour. While PHP provides regular
expression matching built in, it doesn't provide a very nice interface for
working with regular expressions. Elephant provides an Object-Oriented interface
for working with regular expressions (and also Wildcard matching).

## Regular Expression Matching

The primary way is using regular expressions. You can either create a new
regular expression matcher, or use the available static methods for a quick
match.

### Available methods

- `setFlags(int $flags): self`: Provides an interface to set the PCRE flags for
  `preg_match`.
- `compare(string $against): bool`: Compare a regular expression against
  the value, returning whether or not the regex matches the string. It will
  additionally cache the matches so that `matches()` may be called without
  repeating the `preg_match` call.
- `matches(string $against): array`: Compare a regular expression against
  the value, returning the matches. Throws a `MatcherException` if it fails to
  match.
- `in(iterable $against): bool`: Compares a regular expression against all
  strings in an array, recursively, returning true or false. It will
  additionally cache the matches for every string. It ignores items in an
  array that aren't strings.
- `matchesIn(iterable $against): array`: Compares a regular expression against
  all strings in an array, returning all the matches for every string. Throws a
  `MatcherException` if there are no matches.
- `static match(string $pattern, string $against): bool`: Provides a static
  interface for `compare()`.
- `static matchIn(string $pattern, iterable $against): bool`: Provides a static
  interface for `in()`.

### Regex example usage

```php
use Elephant\Helpers\Matchers\Regex;

$string = 'This is a spammy string.'
$arrayOfStrings = [
    'This is another spam string!.',
    'Some extra stuff' => [
        'this isn\'t dangerous though...',
        'nor is this...',
    ],
];

if (Regex::match('/string/i', $string)) {
    echo "This matches!";
}

$regex = new Regex('/spam(my)?/i');

if ($regex->compare($string)) {
    echo 'This is a spammy string!';

    $matches = $regex->matches($string);
}

if ($regex->in($arrayOfStrings)) {
    echo 'This contains a spammy string!';

    $matches = $regex->matchesIn($arrayOfStrings);
}

```

## Wildcard Matching

Alternatively, if you don't need a complex expression, and a more simple
expression would work for you, you can use a wildcard expression.
This still uses regular expressions under the hood, however regex special
characters are escaped. These should not be wrapped in regex delimiters, and
are always case insensitive. It contains all of the same methods as described in
(Regular Expressions.)[#regular-expression-matching]

### Wildcard special characters
- `*`: 0 or more of any character.
- `?`: Exactly one of any character.
- `[abc]`: One of any character in `abc`.
- `[[abc]]`: One or more of any character in `abc`.
- `[!xyz]`: One of any character *not* in `xyz`.
- `[[!xyz]]`: One or more of any character *not* in `xyz`.

### Wildcard example usage

```php
use Elephant\Helpers\Matchers\Wildcard;

$string = 'This is a spammy string.'
$arrayOfStrings = [
    'This is another spam string!.',
    'Some extra stuff' => [
        'this isn\'t dangerous though...',
        'nor is this...',
    ],
];

if (Wildcard::match('str*', $string)) {
    echo 'This is stating it\'s a string.';
}

$wc = new Wildcard('*spam*');

if ($wc->compare($string)) {
    echo 'This is a spammy string!';

    $matches = $wc->matches($string);
}

if ($wc->in($arrayOfStrings)) {
    echo 'This contains a spammy string!';

    $matches = $wc->matchesIn($arrayOfStrings);
}

```

