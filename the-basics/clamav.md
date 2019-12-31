# ClamAV

## Table of Contents

- [Introduction](#introduction)
  - [Example](#example)
- [Clamd](#clamd)
  - [ElephantMFA Configuration](#elephantmfa-configuration)
  - [Converting to SpamTests](#converting-to-spamtests)

## Introduction

Elephant has support for virus detection using [ClamAV].

ClamAV is a free and open-source standard virus scanner for mail gateways. It
has a large enough community behind it creating extra filters for improved
catching of phishing attempts and the likes.

### Example

Below is an example integration of a ClamAV [filter](filters.html) to virus test, and quarantine messages marked as infected:

```php
<?php

namespace App\Mail\Filters\Data;

use Elephant\Contracts\Filter;
use Elephant\Contracts\Mail\Mail;
use Elephant\Filtering\Actions\Quarantine;
use Elephant\Filtering\Scanners\ClamAV as ClamAVClient;

class ClamAV implements Filter
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
        $clamAV = new ClamAVClient($email);
        // Scan, and if failed, report the error.
        if (! $clamAV->scan()) {
            error("ClamAV error: $clamAV->error");

            return $next($email);
        }

        // Get results.
        $results = $clamAV->getResults();

        $infected = $results['infected'];

        $names = '';
        if ($infected) {
            $names = implode(', ', array_map(function (string $k, string $v): string {
                return "[$k:$v]";
            }, array_keys($results['viruses']), $results['viruses']));
        }

        // Generate headers
        $xVirusScannerHeader = ($infected ? 'INFECTED' : 'CLEAN') . " $names";

        if ($results['error']) {
            $names = implode(', ', array_map(function (string $k, string $v): string {
                return "[$k:$v]";
            }, array_keys($results['errors']), $results['errors']));

            $xVirusScannerHeader = "ERROR $names";
        }

        // Log the headers being added.
        info("X-Virus-Scanner: $xVirusScannerHeader");

        // Append the headers.
        $email->appendHeader('X-Virus-Scanner', $xVirusScannerHeader);

        // Quarantine if infected.
        if ($infected) {
            throw new Quarantine();
        }

        return $next($email);
    }
}
```

## Clamd

### ElephantMFA Configuration

The configuration for integrating ClamAV with Elephant resides in `config/scanners.php`.

- `socket` [string]: The socket is the connection to clamd. It *must* be prefixed with
  `ipv4://`, `ipv6://` or `unix://` depending on the kind of socket that clamd is listening on.
  - `ipv4://`: Must be formatted as `ipv4://IP:port` format. See the example below.
  - `ipv6://`: Must be formatted as `ipv6://[IP]:port` format.
  - `unix://`: Must be formatted as `unix:///path/to/clamd.sock` format.
- `max_size` [int]: The maximum size a file will be to be sent to ClamAV. If a file
  is larger than this, it won't be sent. Large files take longer to scan, and
  viruses are usually stored in relatively small files, so larger files are
  often times safe to not scan.
  Set to 0 to send all files, regardless of size.
- `on_disk` [bool]: Recommendation is to leave this to false (see below).
  - If set to true, this option will save all the files to the
    `tmp/clamav/ab/abcdefghijklmnop/` folder, where `ab` is the first two characters
    of the queue ID, and the rest is the queue ID itself. After saving the files,
    it will call `MULTISCAN` on the directory, where ClamAV will then asynchonously
    scan all the files in the directory. It will delete the directory afterwards.
    This is very file-system intensive, and will slow down applications running on
    slower file systems. It is important to note that writes to the file system
    still is handled sequentially, even if scanning is asynchronous.
  - If set to false, this option will instead send each of the files sequentially
    through ClamAV using `INSTREAM`, causing it to be handled entirely in memory.
    Due to being handled in memory, each operation will be a bit faster, but because
    it is being handled sequentially it will likely be slower for a large number
    of smaller files. It's important to note, that in a docker setup,
    unless you are sharing the `tmp/` directory with your ClamAV container, this
    is the only way to scan items using ClamAV.
- `send_email` [array]: Control over whether or not to send the email itself to ClamAV.
  This can be very beneficial to send, because a number of phishing specific rulesets
  only work on .eml files, so sending the mail itself will allow those to be caught
  and turned into spam tests.
  - `enabled` [bool]: Whether or not to send the email.
  - `max_size` [int]: Much like the [spamassassin](spamassassin.md) configuration,
    this option will only send the first X bytes up to the size of specified for
    the mail. This prevents ClamAV taking an excessively long time, or potentially
    rescanning objects it has scanned already from the separated out files. The
    lower this value, the faster ClamAV will process larger mails.

See the example below:
```php
'clamav' => [
    'socket' => env('CLAMAV_DSN', 'ipv4://127.0.0.1:3310'),
    'max_size' => 64 * 1000, // 64 Kb
    'on_disk' => false,
    'send_email' => [
        'enabled' => true,
        'max_size' => 128 * 1000, // 128 Kb
    ],
],
```

### Converting to SpamTests

It would be prudent to transform a number of "virus" definitions from ClamAV to
spam tests or be more lenient with them in some way. The reason for this is,
especially with the [third-party] definitions, there are a number of definitions
that are either a cause for false positives, or are meant to only aid the checking,
and not actually be used to block a mail out-right.

[third-party]: https://github.com/extremeshok/clamav-unofficial-sigs
[ClamAV]: https://www.clamav.net/
