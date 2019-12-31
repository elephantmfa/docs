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

Below is an example integration of a SpamAssassin [filter](filters.html) to spam test, and quarantine messages marked as spam:

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
     * Check the mail using SpamAssassin.
     *
     * @param Mail $mail
     * @param callable $next
     * @return void
     */
    public function filter(Mail $email, $next)
    {
        // Create a new SpamAssassin client.
        $sa = new SpamAssassinClient($email);
        // Scan, and if failed, report the error.
        if (! $sa->scan()) {
            error("SpamAssassin error: $sa->error");

            return $next($email);
        }

        // Get results.
        $results = $sa->getResults();

        // Get the total score.
        $totalScore = $results['total_score'];

        // Format for the headers.
        $tests = collect($results['tests'])->map(function ($test) {
            return "{$test['name']}={$test['score']}";
        })->implode(',');

        $spamStatus = $totalScore > 5 ? 'Yes' : 'No';

        // Generate headers
        $xSpamStatusHeader = "$spamStatus score=$totalScore tests=$tests";
        $xSpamCheckerVersion = "SpamAssassin v{$results['version']}; ElephantMFA v" . Application::VERSION;

        // Log the headers to be added.
        info("X-Spam-Status: $xSpamStatusHeader");
        info("X-SpamChecker-Version: $xSpamCheckerVersion");

        // Append the headers.
        $email->appendHeader('X-Spam-Status', $xSpamStatusHeader);
        $email->appendHeader('X-SpamChecker-Version', $xSpamCheckerVersion);

        // Quarantine if considered Spam.
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

- `socket` [string]: The socket is the connection to SpamD. It *must* be prefixed with
  `ipv4://`, `ipv6://` or `unix://` depending on the kind of socket that SpamD is listening on.
  - `ipv4://`: Must be formatted as `ipv4://IP:port` format. See the example below.
  - `ipv6://`: Must be formatted as `ipv6://[IP]:port` format.
  - `unix://`: Must be formatted as `unix:///path/to/spamd.sock` format.
- `max_size` [int]: The total bytes sent to SpamAssassin. If the mail is larger,
  then only the first part of the mail, up to the size listed will be sent.
  Set to 0 for the entire mail, regardless of size to be sent.
- `timeout` [int]: The maximum amount of time to wait for SpamAssassin to return a response.
- `spamd` [array]: The configuration for Elephant to manage the SpamD process.
  - `manage` [bool]: If enabled, Elephant will start and stop SpamD along side it. Additionally log output from SpamD will be fed directly into Elephant's logs.
  - `parameters` [array[string]]: An array of all of the parameters to pass into the SpamD process.

See the example below:
```php
'spamassassin' => [
    'socket' => env('SPAMASSASSIN_PORT', 'ipv4://127.0.0.1:783'),
    'max_size' => 128 * 1000, // 128 Kb
    'timeout' => 60, // seconds
    'spamd' => [
        'manage' => true,
        'parameters' => [
            '-u ubuntu'
        ],
    ],
],
```

### SpamAssassin Configuration

It is recommended to add the following to your `/etc/mail/spamassassin/local.cf` if you want spam scores for the tests.
You will want the scores from individual tests if you plan on adjusting scores or using the tests in different ways than just an aggregate score.

```
add_header all Status tests=_TESTSSCORES_ autolearn=_AUTOLEARN_ version=_VERSION_
```

Additionally, while it may not be necessary to restart ElephantMFA for PHP file changes, it is a requirement restart SpamD if SpamAssassin configuration changes. Keep this in mind if Elephant is managing the SpamD instance.
