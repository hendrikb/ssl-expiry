# ssl-expiry
ssl-expiry helps you to track expiration dates of your SSL certificates

## Requirements

This tool has been developed and tested on Arch Linux. It is written entirely in bash and relies on fairly recent versions of the shell, openssl and [bc](https://www.gnu.org/software/bc/bc.html), sed and grep. In general, it might not work out of the box on operating systems other than GNU Linux.

## Installation

Just clone this repository and run `ssl-expiry -h`. Optionally, you can use `make install` (which might need root privileges or `sudo`) to install the tool to `/usr/local/bin`. 

## Usage

There is some help included, try `ssl-expiry -h`:

```
Usage: ssl-expiry [options] <HOST LIST>

Options: -s|--separator=';'  Defines the symbol(s) that separate the result table cells
         -l|--limit=FALSE    Defines a GNU date like duration limit up to which you want
                             to see results. Useful for limiting output to soon expiring
                             certifcates. Try: --limit="10 days" See: man 1 date
         -H|--hours          Shows expiration hint in hours, if ommitted, it will be days
         -h|--help           Shows this text and exits

HOST LIST is a space separated list of the following notations in arbitrary combinations:
some.example.com             asks some.example.com for a certificate on port 443 (https)
some.example.com:465         asks some.example.com for a certificate on port 465 (smtps)
@/some/file/path             open a file from the given path reading its entries one entry
                             per line. Supported formats are like the two mentioned above.`
```

## Examples

**Plain call**:
```
$> ssl-expiry google.com
google.com:443;Feb 2 15:31:00 2017 GMT;67 days left
```

**Combined target definitions**:
Provide some other file with domain names
```
$> cat > /tmp/other_search_engines
yahoo.com
bing.com
^D
```
Then, use the recently created file, and a regular domain entry, as targets.
```
$> ssl-expiry google.com @/tmp/other_search_engines
google.com:443;Feb 2 15:31:00 2017 GMT;67 days left
yahoo.com:443;Oct 30 23:59:59 2017 GMT;337 days left
bing.com:443;May 4 17:10:22 2018 GMT;523 days left
```

**Query many, but only show those, that expire within the next 90 days** (very handy for cron notifications!):
```
$> ssl-expiry --limit='90 days' google.com yahoo.com microsoft.com
google.com:443;Feb 2 15:31:00 2017 GMT;67 days left
```

This one supports the GNU date `--date="something something"` syntax. You can read up on additionally supported offset formats via `man 1 date` (the [man page](https://linux.die.net/man/1/date)).

**Advanced sorting:** The power of the pipe is strong on this one:
```
$> ssl-expiry google.com yahoo.com microsoft.com | sort -t';' -k3 -n
google.com:443;Feb 2 15:31:00 2017 GMT;67 days left
yahoo.com:443;Oct 30 23:59:59 2017 GMT;337 days left
microsoft.com:443;Apr 12 00:29:36 2018 GMT;500 days left
```

Remember that you can always make use of the pipeline and combine ssl-expiry with other fine tools like `awk`, `cut`, `sort`, `uniq` or `sed`, especially when working with huge domain listings using the `@/spath/to/my_domain_list` notation.

## Legal

This tool was quickly hacked by [Hendrik Bergunde](https://github.com/hendrikb) in 2016. Its license is [MIT](https://github.com/hendrikb/ssl-expiry/blob/master/LICENSE).
