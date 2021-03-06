# Filters

## Table of Contents

- [Introduction](#introduction)
- [Basic Filters](#basic-filters)
  - [Defining Filters](#defining-filters)
  - [Generating Filters](#generating-filters)
  - [Routing a filter](#routing-a-filter)
- [Defining the outcome of a filter](#defining-the-outcome-of-a-filter)
  - [Allow](#allow)
  - [Reject](#reject)
  - [Defer](#defer)
  - [Quarantine](#quarantine)
  - [Drop](#drop)
- [Dependency Injection & Filters](#dependency-injection-&-filters)

## Introduction

Filters are the main way that ElephantMFA interacts with mail. Each filter will
run on every mail, allowing the fate of the mail to be determined.
ElephantMFA filters are classes that are run through a pipeline that will take
effect on each stage of the SMTP process.
Filters are stored in the `app/Mail/Filters` directory. By default, the
directory structure then has the following folders:

- `Connect`
  - These are the filters that are run immediately upon the connection to
    ElephantMFA.
- `Helo`
  - These filters run after the `HELO` command is sent.
- `RcptTo`
  - These filters run after the `RCPT TO` command is sent.
- `MailFrom`
  - These filters run after the `MAIL FROM` command is sent.
- `Data`
  - These filters are run after the `<cr><lf>.<cr><lf>` at the end of the
    `DATA` command.
- `Queued`
  - If queueing mails is enabled, then these filters run on any mail that has
    been queued by ElephantMFA.
  - If queueing mails is disabled, these are skipped altogether.
  - This is the only time that you are able to filter the body on a
    per-recipient basis.

There is an example filter in each of these folders.

## Basic Filters

### Defining Filters

Below is an example of a basic filter class. Note that the filter implements
the filter contract included with ElephantMFA.

```php
<?php

namespace App\Mail\Filters\Data;

use Elephant\Contracts\Filter;
use Elephant\Contracts\Mail\Mail;

class LogSubject implements Filter
{
    public function filter(Mail $email, $next)
    {
        info($email->headers['subject'][0]);

        return $next($email);
    }
}
```
As you may have noticed, there primary (and only in this case) function in the
file is `filter`. The filter function is required in every filter.

### Generating Filters

An easy way to create a new filter is to generate the filter using the elephant
command line tool. Just run:
```sh
php elephant make:filter
```

### Routing a filter

Unlike HTTP servers, we don't have routes, however we do need to define where
the filter will run. In the `app/Mail/Kernel.php`, define which step the filter
runs on, like so:

```php
<?php

namespace App\Mail;

use Elephant\Foundation\Mail\Kernel as MailKernel;

class Kernel extends MailKernel
{
    protected $filters = [

        /**
         * A list of filters to apply upon connection.
         *
         * @var array
         */
        'connect' => [
            \App\Mail\Filters\Connect\SetOutbound::class,
        ],

        /**
         * A list of filters to apply when HELO/EHLO command is called.
         *
         * @var array
         */
        'helo' => [
            \App\Mail\Filters\Helo\LogHelo::class,
        ],

        /**
         * A list of filters to call when the MAIL FROM command is called.
         *
         * @var array
         */
        'mail_from' => [
            \App\Mail\Filters\MailFrom\LogFrom::class,
        ],

        /**
         * A list of filters to call upon each call of RCPT TO command.
         *
         * @var array
         */
        'rcpt_to' => [
            \App\Mail\Filters\RcptTo\LogRecipient::class,
        ],

        /**
         * A list of filters to apply at the end of the DATA command.
         *
         * @var array
         */
        'data' => [
            \App\Mail\Filters\Data\LogSubject::class,
        ],

        /**
         * A list of filters to apply to queued mail.
         *
         * @var array
         */
        'queued' => [
            \App\Mail\Filters\Queued\LogQueueId::class,
        ],
    ];
}
```

Notice how the filters are all stored in the protected `$filters` variable in
the kernel. The sections need to be labeled as above, or else the filters
associated will not run on the appropriate step.

## Defining the outcome of a filter

### Allow

To allow a mail, skipping past future filters in the same step, just return the
mail itself.

```php
public function filter(Mail $email, $next)
{
    if ($forSomeReasonItsAllowed) {
        return $email;
    }

    return $next($email);
}
```

To bypass future filters in future steps, you can also set the final destination
using the `setFinalDestination()` function on the mail object.

```php
public function filter(Mail $email, $next)
{
    if ($forSomeReasonItsAllowed) {
        $email->setFinalDestination('allow');

        return $email;
    }

    return $next($email);
}
```

### Reject

When working before-queue, rejecting a mail will return a 5XX response and will
stop allowing more commands in regards to the email. This would be used if you
block email addresses, in the `RCPT TO` step, you have a filter that rejects it
before even getting to the data step, thus preventing the processing required.

To reject, it is as simple as throwing a `Reject` action:

```php
public function filter(Mail $email, $next)
{
    if ($forSomeReasonMailIsBlocked) {
        throw new Reject('5.0.0 Address not allowed', 550);
    }

    return $next($email);
}
```

The defaults for a reject exception are:
- `message`: "Not ok"
- `code`: 550

The code and message are what is responded back to the server connecting to
ElephantMFA.

> **Note:** In a post-queue setup, rejecting a mail will delete the mail, silently. Because
> ElephantMFA has already accepted the mail, it cannot return the error code back
> to the sender.

### Defer

When working before-queue, rejecting a mail will return a 4XX response and will
stop allowing more commands in regards to the email. This would be used to
implement a gray-listing system, temporarily blocking mail, forcing the sending
server to try again later (usually after 5 minutes).

To defer, it is as simple as throwing a `Defer` action:

```php
public function filter(Mail $email, $next)
{
    if ($forSomeReasonMailIsTempBlocked) {
        throw new Defer('4.0.0 Try again later', 450);
    }

    return $next($email);
}
```

The defaults for a reject exception are:
- `message`: "Try again later"
- `code`: 450

The code and message are what is responded back to the server connecting to
ElephantMFA.

> **Note:** In a post-queue setup, deferring a mail will delete the mail, silently. Because
> ElephantMFA has already accepted the mail, it cannot return the error code back
> to the sender.

### Quarantine

Quarantine is a special case. It (usually) accepts the mail through, but then
still holds it up, however unlike going into the queue, quarantined mail will
have no more processing done on it. This is what is commonly used when a mail
is determined to be Spam, so that it may be verified and released at a later
date.

To quarantine a mail, it is as simple as throwing the `Quarantine`:

```php
public function filter(Mail $email, $next)
{
    if ($forSomeReasonMailIsQuarantined) {
        throw new Quarantine('ok...', 250);
    }

    return $next($email);
}
```

The defaults for a reject exception are:
- `message`: "Ok"
- `code`: 250

Notice how quarantine still accepts the mail. This is important, as we do not
want the potential spammer to know we rejected it. Alternatively, a 4XX or 5XX
response may be given to defer or reject the mail, while still quarantining it.
If this is done before the end of the `DATA` step however, it will prevent the
body of the message to be quarantined, and thus only the envelope information
up until the point of the 4XX or 5XX response was sent will be stored. For this
reason, it is not a good idea to provide a 4XX or 5XX response.

If you would like the mail to be quarantined in addition to whatever else the
final result may be, then you can call `Transport::sendTo('quarantine');`:

```php
use Elephant\Mail\Transport;

...

public function filter(Mail $email, $next)
{
    if ($archive) {
        Transport::sendTo('quarantine');
    }

    return $next($email);
}
```

### Drop

Drop is very similar to reject, however it will kill the connection at the same
time. This can be used to prevent spammers who attempt to waste processing power
by consistently connecting and sending mail to ElephantMFA. For example,
dropping connections from a specific IP on the connect step.

To drop a connection, it is as simple as throwing a `Drop` action:

```php
public function filter(Mail $email, $next)
{
    if ($forSomeReasonMailIsToBeDropped) {
        throw new Drop('Please go away.', 550);
    }

    return $next($email);
}
```

The defaults for a reject exception are:
- `message`: "Not ok"
- `code`: 550

You can also pass a 2XX code to the `Drop`. If done, it changes the
behaviour of drop altogether. Instead of dropping the connection, it will drop
the mail. It will allow the mail all the way through (skipping anymore filters)
and then delete the mail after it has been accepted. This is a good way to trick
spammers into thinking the mail was successfully delivered. This is a less
common practice compared to quarantining though, as it doesn't allow for any
validation of the mail by human eyes.

## Dependency Injection & Filters

The Elephant [Service Container](https://laravel.com/docs/6.x/container) is used
to resolve all of the filters. As a result, you may type-hint any dependencies
your filter may need in it's constructor. The declared dependencies will
automatically be resolved and injected into the controller instance:

```php
<?php

namespace App\Mail\Filters\Queued;

use Elephant\Contracts\Filter;
use Elephant\Contracts\Mail\Mail;
use Elephant\Filtering\Scanners\SpamAssassin as SA;

class SpamAssassin implements Filter
{
    protected $sa;

    public function __construct(SA $sa)
    {
        $this->sa = $sa;
    }

    public function filter(Mail $email, $next)
    {
        $sa->scan($email);

        // Do stuff with SpamAssassin.

        return $next($email);
    }
}
```

You may also type-hint any
[Laravel or Elephant contract](https://laravel.com/docs/6.x/contracts). If the
container can resolve it, you can type-hint it. Depending on your application,
injecting your dependencies into your controller may provide better testability.
