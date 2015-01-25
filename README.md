# Why #

Writing bash scripts that do file verification using gpg that really is secure
and passes a comprehensive threat model (that covers indefinite freeze,
rollback, endless data attacks, etc.) is hard. gpg-bash-lib's goal is to
provide a bash library that we can audit and abstract the hard work into
reuseable functions.

# What does it do #

* Abstracts file verification into common functions.
* Allows detecting of stale files, i.e. detection downgrade or indefinite
freeze attacks by implementing a valid-until like mechanism.
* Internally parses gpg's --status-file output.
* It is signal friendly.
* Detects endless data attacks, aborts and reports this.
* Detects indefinite freeze and rollback (downgrade) attacks and reports this.
* Can help with verification of names of files, that are otherwise not covered
by default when using gpg.
* Provide diagnostic output (variables) that contain information if the local
clock is sane by comparing signature creation date with local clock.

# What does it NOT do

* Anything else not mentioned above in "What does it do".

# Requirements #
* bash

# Usage #
UNFINISHED!

## Theoretical ##
### Introduction ###
It is highly recommended that you `set -e` or set up your own ERR trap before
you run

```
source "/usr/lib/gpg-bash-lib/source_all"
```

A prerequisite is that you set

```
set -o pipefail
set -o errtrace
```
in your script before during execution of this library. This is a good practice
anyhow.

Before execution of this library, enable the ERR trap that it provides.

```
trap "gpg_bash_lib_function_error_handler" ERR
```

Feel free to undo these error handling mechanisms (`set` and trap ERR), if you
must, after execution of this library.

You configure that ERR trap to `eval`uate your own error handler through the
variable `gpg_bash_lib_input_error_handler_extra`, which is documented below.
Alternatively, you could just look at the `gpg_bash_lib_function_error_handler`
function and implement your own ERR trap after `source`ing this library.

Set at least all required `gpg_bash_lib_input_*` variables, that are documented
below. Then run the main function `gpg_bash_lib_function_main_verify` (or just
one function you wish to use). Then enjoy the `gpg_bash_lib_output_*`
variables, that this library has set for you.

### Variables ###
#### Input Variables ####

`gpg_bash_lib_input_temp_folder`
* description: Folder that can be used for temporary files. Warning: that
folder will be deleted before usage to make sure it is clean!
* required: no
* defaults to: `"$(mktemp --directory)"`
* expected value: `/path/to/temp/folder`
* example: `gpg_bash_lib_input_temp_folder=/home/user/some-tmp-folder`

`gpg_bash_lib_input_key_import_dir`
* description: The folder that contains gpg signing keys that are supposed to
be accepted. Must already exist. Must contain gpg public keys.
* required: yes
* expected value: `/path/to/folder`
* example: `gpg_bash_lib_input_key_import_dir=/usr/share/program-name/signing-keys.d`

`gpg_bash_lib_input_file_name_enforce`
* description: Enforce, that the name of the file is stored in the `file@name`
OpenPGP notation and matches the actual file name.
If enabled, `gpg_bash_lib_output_alright` will be set to `true` if it matches.
Otherwise to false. Rather `gpg_bash_lib_output_file_name_tampering` will be
set to `true` (match), `missing` (no `file@name` OpenPGP notation inside the
signature, `false` (mismatch) accordingly.
* required: no
* defaults to: disabled by default
* expected value: `true` or `false`
* example: `gpg_bash_lib_input_file_name_enforce=true`

`gpg_bash_lib_input_cleanup`
* description: Delete the folder stored in the `gpg_bash_lib_input_temp_folder`
variable or not.
* required: no
* defaults to: disabled by default
* expected value: `true` or `false`
* example: `gpg_bash_lib_input_cleanup=true`

`gpg_bash_lib_input_data_file`
* description: The file that is supposed to be verified.
* required: yes
* expected value: `/path/to/file`
* example: `gpg_bash_lib_input_data_file=/home/user/some-file.tar.gz`

`gpg_bash_lib_input_sig_file`
* description: The signature that is supposed to be verified.
* required: yes
* expected value: `/path/to/file`
* example: `gpg_bash_lib_input_data_file=/home/user/some-file.tar.gz.asc`

`gpg_bash_lib_input_error_handler_extra`
* description: The signature that is supposed to be verified.
* required: no
* defaults to: none
* expected value: A command or bash function name.
* example: `gpg_bash_lib_input_error_handler_extra='error_handler'`
* example: `gpg_bash_lib_input_error_handler_extra='error_handler "$gpg_bash_lib_output_message"'`

`gpg_bash_lib_input_verifiy_timeout_after`
* description: After how many seconds a gpg verification attempt should be
forced timeout. Sends signal SIGTERM to gpg. Useful to defeat an endless data
attacks or bugs. Increase this value when you are working with bigger files
and/or slow systems.
* required: no
* defaults to: 10
* expected value: integer
* example: `gpg_bash_lib_input_verifiy_timeout_after=120`

`gpg_bash_lib_input_verify_kill_after`
* description: After how many seconds, if gpg did not react to SIGTERM due to
timeout (see above), signal SIGKILL should be used. Useful to defeat an
endless data attacks or bugs.
* required: no
* defaults to: 10
* expected value: integer
* example: `gpg_bash_lib_input_verifiy_timeout_after=20`

`gpg_bash_lib_input_maximum_age_in_seconds`
* description: After how many seconds, a signature is considered outdated.
gpg adds the creation time of the signature (Signature Creation Date) to every
signature. That value is detected (see also variables
`gpg_bash_lib_output_signed_on_unixtime` and
`gpg_bash_lib_output_signed_on_date` below) and compared against this variable.
* required: no
* defaults to: 2592000 (which is sane as 1 month)
* expected value: integer
* example: `gpg_bash_lib_input_maximum_age_in_seconds=2592000`

`gpg_bash_lib_input_slow_clock_lenient_up_to_seconds`
* description: After how many seconds, a signature is considered outdated.
* required: no
* defaults to: 1800 (which is same as 30 minutes)
* expected value: integer
* example: `gpg_bash_lib_input_slow_clock_lenient_up_to_seconds=1800`

#### Output Variables ####
`gpg_bash_lib_output_failure`
* possible values: `""` or `true`
* recommendation: Make sure to check for this value!
Since the trap `gpg_bash_lib_function_error_handler` has been
invoked, something unexpected, a bug has occurred. Regard this as verification
failed.

`gpg_bash_lib_output_diagnostic_message`
* possible values: A verbose diagnostic textual string.
* recommendation: Display this value when running your script in verbose mode.

`gpg_bash_lib_output_gpg_import_output`
* possible values: A textual string containing output of the `gpg --import`
part.
* recommendation: Since already included in
`gpg_bash_lib_output_diagnostic_message`, you most likely will not need it.

`gpg_bash_lib_output_gpg_exit_code`
* possible values: integer. The exit code of the `gpg --verify` action.
* recommendation:

`gpg_bash_lib_output_gpg_verify_output`
* possible values: A textual string containing output of the `gpg --verify`
part.
* recommendation: Since already included in
`gpg_bash_lib_output_diagnostic_message`, you most likely will not need it.

`gpg_bash_lib_output_gpg_verify_status_fd_output`
* possible values: A textual string containing output of the
`gpg --status-file` part.
* recommendation: Since already included in
`gpg_bash_lib_output_diagnostic_message`, you most likely will not need it.

`gpg_bash_lib_output_signed_on_unixtime`
* possible values: integer.
* recommendation: Consider using this variable instead of
`gpg_bash_lib_output_signed_on_date` by converting it to some formatted date
string that you prefer.
* example content: `1419456919`

`gpg_bash_lib_output_signed_on_date`
* possible values: `gpg_bash_lib_output_signed_on_unixtime `converted to a
textual date string using
`"$(date --date "@$gpg_bash_lib_output_signed_on_unixtime")"`.
* recommendation: Show this variable in your script, ask the user for
confirmation.
* example content: `Wed Dec 24 21:35:19 UTC 2014`

`gpg_bash_lib_output_file_name_tampering` will be
* possible values: Set to `true` (match), `missing` (no `file@name` OpenPGP
notation inside the signature, `false` (mismatch) or "" (not in use,
not using `gpg_bash_lib_input_file_name_enforce=true`) accordingly.

`gpg_bash_lib_output_notation["file@name"]`
* possible values: name of file that was signed or `""` if not in use
(not using `gpg_bash_lib_input_file_name_enforce=true`).
* example content: `test-file`
* example use: `echo "${gpg_bash_lib_output_notation[$"file@name"]}"`

`gpg_bash_lib_output_slow_clock_lenient_up_to_pretty_output`
* possible value: Textual string containing the duration of how lenient the
clock leniency check is. (Contains the result of the conversion of
`gpg_bash_lib_input_slow_clock_lenient_up_to_seconds` to a pretty format.)
* example content: `30 minutes`

`gpg_bash_lib_output_validsig`
* possible values: `true` (successful verification) or `""` (unsuccessful
verification).
* recommendation: Check if its value is `true`. Abort otherwise.
* example content: `true`
* example usage:
```
if [ ! "$gpg_bash_lib_output_validsig" = "true" ]; then
   ## Notify about this situation.
   exit 1
fi
```

`gpg_bash_lib_output_validsig_fingerprint_in_hex`
* possible values: The fingerprint of the key that signed the file in hex or
`""` in case of unsuccessful verification.
* recommendation: May or may not be useful to show this value in your script
(in verbose mode).
* example content: `F38633B0A3F06A55CC0076C81081641AC4D57DB9`

`gpg_bash_lib_output_current_unixtime`
* possible values: integer, unixtime at time of running this library.

`gpg_bash_lib_output_current_time`
* possible values: A textual string, `gpg_bash_lib_output_current_unixtime`
converted to a date string.

`gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime`
* possible values: An integer, that represents the Relative Signature Creation
Time. See the example of
`gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime_pretty`
below to understand what it is doing. Can be used when
`gpg_bash_lib_output_freshness_detail` is `slow` or `lenient`.

`gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime_pretty`
* possible values: A textual
string, `gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime`
converted to a pretty format.
Can be used when
`gpg_bash_lib_output_freshness_detail` is `slow` or `lenient`.
* example usage:
```
if [ "$gpg_bash_lib_output_freshness_detail" = "slow" ] || [ "$gpg_bash_lib_output_freshness_detail" = "lenient" ]; then
   echo "Relative Signature Creation Time: According to your system clock, signature was created $gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime_pretty before current time."
fi
```

`gpg_bash_lib_output_current_unixtime_minus_signed_on_unixtime`
* possible values: An integer, that represents the Relative Signature Creation
Time. See the example of
`gpg_bash_lib_output_current_unixtime_minus_signed_on_unixtime_output_pretty`
below to understand what it is doing. Can be used if
`gpg_bash_lib_output_freshness_detail` is `current` or `outdated.`.

`gpg_bash_lib_output_current_unixtime_minus_signed_on_unixtime_output_pretty`
* possible values: A textual
string, `gpg_bash_lib_output_current_unixtime_minus_signed_on_unixtime`
converted to a pretty format.
Can be used if
`gpg_bash_lib_output_freshness_detail` is `current` or `outdated.`.
* example usage:
```
if [ "$gpg_bash_lib_output_freshness_detail" = "current" ] || [ "$gpg_bash_lib_output_freshness_detail" = "outdated" ]; then
   echo "Relative Signature Creation Time: According to your system clock, signature was created $gpg_bash_lib_output_current_unixtime_minus_signed_on_unixtime_output_pretty ago."
fi
```

`gpg_bash_lib_output_in_future_in_seconds`
* possible values: In case of `gpg_bash_lib_output_freshness_detail` is
`outdated`, it contains an estimation how many seconds the clock might be fast.

`gpg_bash_lib_output_in_future_output_pretty`
* possible values: A textual string,
`gpg_bash_lib_output_in_future_output_pretty` converted to a pretty format.
* example use:
```
echo "gpg_bash_lib_output_in_future_output_pretty: $gpg_bash_lib_output_in_future_output_pretty"
```

`gpg_bash_lib_output_freshness_status`
* possible values: `true` (fresh) or `false` (not fresh).
* example use:
```
if [ "$gpg_bash_lib_output_freshness_status" = "true" ]; then
   echo "Signature is current."
else
   echo "Signature NOT current." 2>&1
   exit 1
fi
```

`gpg_bash_lib_output_freshness_detail`
* description: A string, that contains the result of the comparison of the
local clock (unixtime) with the signature creation date (unixtime).
* possible values:
** `lenient` (Signature is not yet valid, still within accepted range.)
** `slow` (Signature is not yet valid.)
** `outdated` (Signature is no longer valid (outdated).)
** `current` (Signature is current.)
* recommended action:
* example usage:
```
   case "$gpg_bash_lib_output_freshness_detail" in
      "lenient")
         signature_creation_msg="Your clock might be slow.
According to your system clock, signature was created $gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime_pretty before current time.
You can probably ignore this, because it still is within range. (Okay up to $gpg_bash_lib_output_maximum_age_in_seconds_pretty_output before.)"
         ## ...
         ;;
      "slow")
         signature_creation_msg="Your clock might be slow.
According to your system clock, signature was created $gpg_bash_lib_output_signed_on_unixtime_minus_current_unixtime_pretty before current time."
         ## ...
         ;;
      "outdated")
         signature_creation_msg="Signature looks quite old already.
Either,
- your clock might be fast (at least $gpg_bash_lib_output_in_future_output_pretty fast). $clock_hint
- there is really no newer signature yet. Signature is really older than $gpg_bash_lib_output_maximum_age_in_seconds_pretty_output. already. (Older than $gpg_bash_lib_output_in_future_output_pretty already.)
- this is a $SCRIPTNAME bug
- this is an attack"
         ## ...
         ;;
      "current")
         signature_creation_msg="According to your system clock, signatures was created $gpg_bash_lib_output_current_unixtime_minus_signed_on_unixtime_output_pretty ago."
         ## ...
         ;;
      *)
         error "gpg_bash_lib_output_freshness_detail is neither lenient, nor slow, nor outdated, nor current, it is: $gpg_bash_lib_output_freshness_detail"
         ## ...
         exit 125
         ;;
   esac
```

`gpg_bash_lib_output_freshness_msg`
* possible values: A textual string, that contains diagnostic output, that puts
`gpg_bash_lib_output_freshness_detail` into more details, developers speech,
that may or may not be useful to show in your script [when in verbose mode].

`gpg_bash_lib_output_maximum_age_in_seconds_pretty_output`
* possible values: A textual string,
`gpg_bash_lib_input_maximum_age_in_seconds` converted into a pretty format.

`gpg_bash_lib_output_alright`
* possible values: `""`, `true` (if all checks succeeded) or `false` (if at
least one check failed, such as if `gpg_bash_lib_input_file_name_enforce` was
set to `true`, but verification of the name of the file failed or if the
signature was not considered fresh).
* example usage:
```
if [ ! "$gpg_bash_lib_output_alright" = "true" ]; then
   ## ...
   exit 1
fi
```

### File Name Verification ###
To verify the names of files, i.e. to verify the `file@name` OpenPGP Notation,
see also variable `gpg_bash_lib_input_file_name_enforce` above.

To create a signature, that contains this OpenPGP notation, you might like to
use something like the following command and and/or function.

```
sign_cmd() {
   ## GPG signatures do not authenticate filenames by default, therefore add
   ## the name of the file as a OpenPGP notation so at least users or scripts
   ## that look at OpenPGP notations have a chance to detect if file names
   ## have been tampered with. See also:
   ## https://github.com/adrelanos/gpg-bash-lib
   gpg --detach-sign --armor --yes --set-notation "file@name"="$(basename "$1")" "$1"
}
```

And for verification.

```
verify_cmd() {
   gpg --verify-options show-notations --verify "$1"
}
```

### Security Tips ###
#### Signature Creation Date Storage ####
To aid detection of indefinite freeze and rollback (downgrade) attacks,
consider storing the Signature Creation Date (see variables
`gpg_bash_lib_output_signed_on_unixtime` and/or
`gpg_bash_lib_output_signed_on_date`) in a file, so you can compare them next
time you download a supposedly newer signature. If the newly downloaded
signature is older than the last known one, then maybe something is wrong.

#### Signature Creation Date Preseeding ####
The above tip will not work for initial signature downloads. Therefore consider
preseeding a sane initial value.

#### Abstract it! ####
Like the two above tips? We should abstract that code as well. Interested to
implement it into gpg-bash-lib?

### Library Conventions ####
To avoid conflicts with variables or function names, which your script might
have defined earlier, the following conventions have been applied.

* Function names start with `gpg_bash_lib_function_`.
* Variable names found out by the library start with `gpg_bash_lib_output_`.
* Variable names that may be input by the user start with `gpg_bash_lib_output_`.
* Variable names that are only internally used start with `gpg_bash_lib_internal_`.

### Goodies ###
#### SIGNAL Friendliness ####
Any operations that could take longer, such as the `gpg --verify` operation,
are executed using bash's `wait` builtin. To explain this better, see the
following pseudo code, it is the style that this library is using.

```
gpg ... &
gpg_bash_lib_internal_gpg_verify_pid="$!"
wait "$gpg_bash_lib_internal_gpg_verify_pid" || { gpg_bash_lib_output_gpg_exit_code="$?" ; true; };
```

This has the advantage, that your script can still react to any eventual trap's
listening for example for signal SIGTERM.

## Practical ##
TODO: Write `usr/share/gpg-bash-lib/examples/...`.

See also `usr/share/gpg-bash-lib/unit_test`.

# Alternatives

* Writing your own custom code.
* Please add any not listed here.

# Forks, Patches, Testers, Comments, etc.

Welcome.

# Author

* Patrick Schleizer
* e-mail: adrelanos@riseup.net
* [gpg](https://www.whonix.org/wiki/Patrick_Schleizer): 916B8D99C38EAF5E8ADC7A2A8D66066A2EEACCDA
* twitter: https://twitter.com/Whonix
* [Donate](https://www.whonix.org/wiki/Donate)

# License

GPLv3+
