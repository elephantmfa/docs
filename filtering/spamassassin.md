# SpamAssassin


It is recommended to add the following to your `/etc/mail/spamassassin/local.cf` if you want Spam scores for the tests.
Since the SpamAssassin module does not report the score reported from SpamAssassin, you will need the scores to be able to calculate if it's spam.
```
add_header all Status tests=_TESTSSCORES_ autolearn=_AUTOLEARN_ version=_VERSION_
```
