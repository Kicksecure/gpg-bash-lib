# gpg bash library #

Abstracts file verification into common functions. Allows detecting of stale
files, i.e. detection downgrade or indefinite freeze attacks by implementing
a valid-until like mechanism.

Internally parses gpg's --status-file output.

For better security.
## How to install `gpg-bash-lib` using apt-get ##

1\. Add [Whonix's Signing Key](https://www.whonix.org/wiki/Whonix_Signing_Key).

```
sudo apt-key --keyring /etc/apt/trusted.gpg.d/whonix.gpg adv --keyserver hkp://ipv4.pool.sks-keyservers.net:80 --recv-keys 916B8D99C38EAF5E8ADC7A2A8D66066A2EEACCDA
```

3\. Add Whonix's APT repository.

```
echo "deb http://deb.whonix.org buster main contrib non-free" | sudo tee /etc/apt/sources.list.d/whonix.list
```

4\. Update your package lists.

```
sudo apt-get update
```

5\. Install `gpg-bash-lib`.

```
sudo apt-get install gpg-bash-lib
```

## How to Build deb Package ##

Replace `apparmor-profile-torbrowser` with the actual name of this package with `gpg-bash-lib` and see [instructions](https://www.whonix.org/wiki/Dev/Build_Documentation/apparmor-profile-torbrowser).

## Contact ##

* [Free Forum Support](https://forums.whonix.org)
* [Professional Support](https://www.whonix.org/wiki/Professional_Support)

## Donate ##

`gpg-bash-lib` requires [donations](https://www.whonix.org/wiki/Donate) to stay alive!
