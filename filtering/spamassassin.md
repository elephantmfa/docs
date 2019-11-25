# SpamAssassin

## Table of Contents

- [Introduction](#introduction)
  - [Example](#example)
- [SpamD](#spamd)
  - [ElephantMFA Configuration](#elephantmfa-configuration)
  - [SpamAssassin Configuration](#spamassassin-configuration)

## Introduction

Elephant has support for spam classification using [SpamAssassin](https://spamassassin.apache.org/).

Apache SpamAssassin is the #1 Open Source anti-spam platform giving system administrators a filter to classify email and block spam (unsolicited bulk email).

It uses a robust scoring framework and plug-ins to integrate a wide range of advanced heuristic and statistical analysis tests on email headers and body text including text analysis, Bayesian filtering, DNS blocklists, and collaborative filtering databases.

To integrate Elephant and SpamAssassin, it is required that [SpamD](https://spamassassin.apache.org/full/3.1.x/doc/spamd.html) is run so that Elephant may communicate with SpamAssassin. It is possible to tie SpamD in with Elephant so that Elephant controls the running state of SpamD.

### Example

Below is an example integration of a SpamAssassin [filter](filter.html) to spam test, and quarantine messages marked as spam:

```php
<?php

namespace App\Mail\Filters\Data;

use Elephant\Contracts\Filter;
use Elephant\Contracts\Mail\Mail;
use Elephant\Filtering\Exception\QuarantineException;
use Elephant\Filtering\SpamAssassin as SpamAssassinClient;
use Elephant\Foundation\Application;

class SpamAssassin implements Filter
{
    /**
     * Run a filter against the mail.
     *
     * @param Mail $mail
     * @param callable $next
     * @return void
     */
    public function filter(Mail $email, $next)
    {
        $sa = new SpamAssassinClient($email);
        if (! $sa->scan()) {
            error("SpamAssassin error: $sa->error");

            return $next($email);
        }
        $results = $sa->getResults();

        $totalScore = collect($results['tests'])->map(function ($test) {
            return $test['score'];
        })->sum();
        $tests = collect($results['tests'])->map(function ($test) {
            return "{$test['name']}={$test['score']}";
        })->implode(',');

        $spamStatus = $totalScore > 5 ? 'Yes' : 'No';

        info("X-Spam-Status: $spamStatus score=$totalScore tests=$tests");
        info("X-SpamChecker-Version: SpamAssassin v{$results['version']}; ElephantMFA v" . Application::VERSION);

        $email->appendHeader('X-Spam-Status', "$spamStatus score=$totalScore tests=$tests");
        $email->appendHeader(
            'X-SpamChecker-Version',
            "SpamAssassin v{$results['version']}; ElephantMFA v" . Application::VERSION
        );

        if ($totalScore > 5) {
            throw new QuarantineException();
        }

        return $next($email);
    }
}
```

## SpamD

### ElephantMFA Configuration

The configuration for integrating SpamAssassin with Elephant resides in `config/scanners.php`.

```php
'spamassassin' => [
    // Host or path to socket where spamd is running.
    //   Use ipv4:// for an IPv4 address. Must be IP:port format.
    //   Use ipv6:// for an IPv6 address. Must be [IP]:port format.
    //   Use unix:// for a Unix socket.
    'socket' => env('SPAMASSASSIN_PORT', 'ipv4://127.0.0.1:783'),
    // The total bytes sent to SpamAssassin
    'max_size' => 128 * 1000, // 128 Kb
    // The timeout until reading from SpamAssassin is given up.
    'timeout' => 60, // seconds
    'spamd' => [
        // If `manage` is enabled, SpamD will be started with ElephantMFA,
        //    and will be killed when ElephantMFA is killed. Additionally,
        //    log output from SpamD will be handled by ElephantMFA.
        'manage' => true,
        'parameters' => [
            '-u ubuntu'
        ],
    ],
],
```

### SpamAssassin Configuration

It is recommended to add the following to your `/etc/mail/spamassassin/local.cf` if you want Spam scores for the tests.
Since the SpamAssassin module does not report the score reported from SpamAssassin, you will need the scores to be able to calculate if it's spam.
```
add_header all Status tests=_TESTSSCORES_ autolearn=_AUTOLEARN_ version=_VERSION_
```
