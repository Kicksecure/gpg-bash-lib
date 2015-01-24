# gpg bash library #

Abstracts file verification into common functions. Allows detecting of stale
files, i.e. detection downgrade or indefinite freeze attacks by implementing
a valid-until like mechanism.

Internally parses gpg's --status-file output.

For better security.

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

`gpg_bash_lib_input_slow_clock_lenient_up_to_seconds`

#### Input Variables ####
TODO: Document

### Library Conventions ####

To avoid conflicts with name spaces, the following conventions have been applied.

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
See also `usr/share/gpg-bash-lib/unit_test`.
