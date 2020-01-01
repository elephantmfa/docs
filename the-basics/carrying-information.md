# Carrying Information

## Table of Contents

- [Introduction](#introduction)
- [Extra Information](#extra-information)
  - [Supplemental Data](#supplemental-data)
  - [Timings](#timings)
- [Functions for Accessing Extra Information](#functions-for-accessing-extra-information)

## Introduction

Sometimes it is very beneficial to be able to carry information around about a
mail message, whether this is spam tests, time taken to do certain operations,
or something else. As such, mails have two public properties that allow for the
carrying of data, `$supplementalData` and `$timings`.

## Extra Information

### Supplemental data

Supplemental Data (`$supplementalData`) is a map containing any data that needs
to be carried. The type is `array<string,mixed>` for the `$supplementalData`
value, so feel free to provide any typed value to the map. This could be used to
store whether a mail is considered outbound, what spam tests have triggered, or
something else. This way, if you have multiple filters that add information that
needs to be handled later on, it can be stored for carrying down the line.

### Timings

Timings (`$timings`) is a map containing the amount of time a specific action
took. By default, Elephant stores a number of timings out of the box. These
include the timings for each step of filtering, as well as the amount of time
different scanners took. This can be useful for debugging why mails are taking a
long time to scan. The type is `array<string,float>` for the `$timings` value,
so no other type (such as a string or bool) should be stored in this map.

> **Note:** The timings for the current step isn't available in a filter, as
> the timing is stored after all filters run. There is an extra method declared
> in the Mail kernel that runs after the QUEUED step (or the DATA step if queue
> is skipped) called `mailLog` that you can implement, as if it were just
> another filter. This is intentionally used for logging data about the message.

## Functions for Accessing Extra Information

The contract for `Mail`s provides two functions, one for setting and one for
getting data in these extra information values. This is so that the interface
is the same, even if a different `Mail` class is used.

```php
/**
 * Add extra data to the mail.
 *
 * @param  string $type Can be either "timing" or "supplemental".
 * @param  string $key
 * @param  mixed  $data The data to set.
 * @return void
 *
 * @throws \InvalidArgumentException
 */
public function addExtraData(string $type, string $key, $data): void;

/**
 * Get extra data.
 *
 * @param  string $type Can be either "timing" or "supplemental".
 * @param  string $key
 * @return mixed
 *
 * @throws \InvalidArgumentException
 */
public function getExtraData(string $type, string $key);
```

Additionally, the built in `Mail` class has two constants, which return the
strings to use for `$type`, in case they wish to be used.

> If there is no plan to utilize a different `Mail` contract implementation,
> then feel free to utilize the public variables.
