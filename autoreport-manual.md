# tapper-autoreport

## ABOUT

`tapper-autoreport` is a "bash include file" you can add ("source") at
the end of your own bash script.

It then magically turns your bash script into an "**Tapper test
suite**".

It also allows your bash script to be executed locally via the `prove`
command, a standard tool to run test scripts that produce TAP output
("Test Anything Protocol") -- without requiring an actual Tapper
framework.

Due to that and no other external dependecies it's a good starting
point to write actual **function tests* that are reusable.

It collects meta information from system, reports test results and can
upload logs and other files over network to a
[Tapper server](http://github.com/amd/Tapper-Reports-Receiver).

## SYNOPSIS

Run the scripts locally in a subdir where the `tapper-autoreport`
script is located.

### most simple usage

* script:

```bash
 #! /bin/bash
 # your own stuff here ...
 . /tapper-autoreport
```

Explanation:

* First it imports utility functions to be used by your script.
* Then you write your test.
* Then calls the actual reporting of test results, with all the Tapper details, like:
  * TAP
  * filename gets name
  * run itself is already a success
  * meta information
  * report grouping heuristics
  * self-report to Tapper
  * upload files
  * print out Tapper report URL

### Run it via 'prove' (a standard test utility)

The tool `prove` is a standard tool available in every Linux
distribution. Use it this way:

```bash
 $ prove ./trivial-example-1.sh
 ./trivial-example-1.sh .. ok
 All tests successful.
 Files=1, Tests=5, 20 wallclock secs
 Result: PASS
```

Explanation:

* "prove" is an existing standard tool
* it prints success statistics
* no report sending happens
* is meant for manual developing/testing

### Run it without 'prove'

Cmd line and output:

```bash
 $ ./trivial-example-1.sh
 # http://tapper/tapper/reports/id/129218
 # - upload ./trivial-example-1.sh ...
 # - upload /boot/config-2.6.32-22-generic ...
 # - upload /proc/cpuinfo ...
 # - upload /proc/devices ...
 # - upload /proc/version ...
```

Explanation:

* execute script
* report output to Tapper server
* upload files
* prints out Tapper report URL
* is meant for final reporting


### Extended usage - using environment variables and params

You can influence the meta information when the automagic and defaults
don't work perfectly:

```bash
 #! /bin/bash
 . ./tapper-autoreport nok /tmp/results.log $?
 ok 0 "affe loewe tiger"
 ok 0 "some other description"
 ok 0 "yet another test description"
 append_tapdata "timecpb: 12.345"
 append_tapdata "timenocpb: 23.456"
 SUITENAME="CPUID-ON"
 SUITEVERSION="2.007"
 OSNAME="Gentoo 10.1"
 CHANGESET="98765"
 HOSTNAME="J-F-Sebastian"
 TICKETURL='https://mybugtracker/bugs/show_bug.cgi?id=901'
 WIKIURL=https://mywiki/wiki/SomeTopic
 PLANNINGID=foocompany.coolproduct.qa.coolfeature.test
 TAPPER_REPORT_SERVER="tapper-devel"
 NOSEND=1
 uname -a | grep -q Linux  # example for exit code
 . ./tapper-autoreport nok /tmp/my-results.txt $?
```

Explanation:

* define additional test results, using fix success status of 0 (0 is TRUE in shell)
* define additional YAML data lines
* overwrite suite name
* overwrite suite version
* overwrite OS name
* overwrite changeset, usually kernel
* overwrite hostname
* specify relevant URL in used ticket system (like Bugzilla, RT, ...)
* specify relevant URL in used wiki
* specify relevant task planning id (like MS Project, TaskJuggler, ...)
* use different report server (e.g. "tapper-devel" for experiments)
* set NOSEND=1 to suppress sending to reports server completely
* param "nok" say "something was not ok"
* param of filename /tmp/my-results.txt means upload the file
* param of integer ($? is last exit code, 0 means ok, else not ok)

## UTILITY FUNCTIONS

Import utility functions at the beginning of the script via

```bash
 . ./tapper-autoreport --import-utils
```

Then you have several functions available.

### Expressing test results

#### ok ARG1 "some description"

Evaluates the first argument with Shell boolean semantics (0 is true)
and appends a corresponding TAP line.

#### negate_ok ARG1 "some description"

Evaluates the first argument with Shell inverse boolean semantics (0
is false) and appends a corresponding TAP line.

#### append_tap "ok - some description"

Appends a complete TAP line where you have taken care for the
"ok"/"not ok" at the beginning.

#### append_tapdata "key: value"

Appends a key:value line at the tapdata YAML block. The key must
start with letter and consist of only alphanum an underscore.

### require_* functions

All `require_*` functions check for something and **gracefully exit**
the script if the requirement is not fulfilled. Use this to
allow the script to run everywhere without polluting results with
false negatives.

#### require_amd_family_range [ MIN [ MAX ] ]

Ensures vendor is AMD and cpu family from `/proc/cpuinfo` is in a
minimum/maximum range. If you don't specify MIN it defaults to 0. If
you don't specify MAX it defaults to MIN.

#### require_cpufeature "foo"

Verifies that the string "foo" occurs in `/proc/cpuinfo` flags section.

#### require_program "foo"

Verify that the program "foo" is available.

Use it to declare external programs you call, like "awk", "bc",
"perl", etc.

#### require_l3cache

Verify that L3 cache is available (checked in
/sys/devices/system/cpu/cpu0/cache/index3).

#### require_netcat

Verify that a variant of `netcat` (netcat, nc) is available.

#### require_root

Verify that the user executing the script is root (UID 0).


#### require_kernel_release_min_1

Verify that the current LK release is less than required 1st level
version number.


#### require_kernel_release_min_2

Verify that the current LK release is less than required 2nd level
version number.


#### require_kernel_release_min_3

Verify that the current LK release is less than required 3rd level
version number.


### Misc auxiliary functions

These are really just utilities to help you but they don't influence
the behaviour like the require_* functions do.

#### has_l3cache

Returns 0 (shell TRUE) if L3 cache is available (checked in
/sys/devices/system/cpu/cpu0/cache/index3).

#### has_cpufeature "foo"

Returns 0 (shell TRUE) if string "foo" occurs in `/proc/cpuinfo` flags
section.

#### has_program "foo"

Return 0 (shell TRUE) if the program "foo" is available.

#### get_vendor

Prints vendor "AMD" or "Intel" from `/proc/cpuinfo`.

#### vendor_intel

Returns 0 (shell TRUE) if vendor is Intel.

#### vendor_amd

Returns 0 (shell TRUE) if vendor is AMD.

#### get_random_number [ MAX ]

Prints a random integer between 0 and MAX (default 32768).

#### get_cpu_family

Print cpu family from `/proc/cpuinfo`.

#### cpu_family_min [ MINFAMILY ]

Returns 0 (shell TRUE) if cpu family from /proc/cpuinfo is greater or
equal to MINFAMILY. Defaults to 0.

#### cpu_family_max [ MAXFAMILY ]

Returns 0 (shell TRUE) if cpu family from /proc/cpuinfo is less or
equal to MAXFAMILY. Defaults to 999.

#### get_first_file [ LIST OF FILENAMES ]

Goes through all specified filenames and prints the first one that
exists and is readable.

#### is_element_in_list WORD "SPACE SEPARATED LIST OF WORDS"

Returns 0 (shell TRUE) if WORD appears in "SPACE SEPARATED LIST OF
WORDS". Remember the usual shell quoting rules.

#### get_kernel_release_1

Prints the 1st part of the kernel version number from uname -r.

#### get_kernel_release_2

Prints the 2nd part of the kernel version number from uname -r.

#### get_kernel_release_3

Prints the 3rd part of the kernel version number from uname -r.


## INFLUENCING BEHAVIOUR

You can

* use environment variables to provide more content
* provide commandline params that "Do What I Mean"
* define hooks (functions) to be called

### Environment Variables

* `TAP[*]`                - Array of TAP lines
* `TAPDATA[*]`            - Array of YAML lines that contain data in TAP
* `HEADERS[*]`            - Array of Tapper headers
* `OUTPUT[*]`             - Array of additional output lines
* `SUITENAME`             - alternative suite name instead of $0
* `SUITEVERSION`          - alternative suite version
* `KEYWORDS`              - space separated keywords to influence suite name
* `OSNAME`                - alternative OS description
* `CHANGESET`             - alternative changeset
* `HOSTNAME`              - alternative hostname
* `TAPPER_REPORT_SERVER`  - alternative report server
* `TICKETURL`             - relevant URL in used ticket system (Bugzilla)
* `WIKIURL`               - relevant URL in used wiki
* `PLANNINGID`            - relevant task planning id (MS Project, TaskJuggler)
* `NOSEND`                - if "1" no sending to Tapper happens
* `NOUPLOAD`              - if "1" no uploading of default files happens


### Command Line Arguments

* `--version`             - print version number and exit
* `nok`                   - declare something was not ok
* [integer]               - exit code of a program, 0 == ok, else not (Hint: use '$?' to refer to last program)
* [filename]              - upload the file


### Function Hooks

#### main_after_hook ()

* optional shell function to be executed at the end of autoreport's main()
* all stdout will be part of the report

## RESULT URLS

They look like this: http://tapper/tapper/reports/id/129218

## WHAT IT REPORTS

### Tapper information

* report group
* testrun
* suite name
* suite version
* machine name
* reporter name (owner)

### System information

* uname
* OS name
* kernel version
* changeset
* kernel flags
* cpuinfo
* ram
* execution time
* bogomips

### File uploads

* itself
* /proc/cpuinfo
* /proc/devices
* /proc/version
* other files you give as params
