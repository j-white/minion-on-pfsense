# Updating the Minion port

```sh
cd ~/FreeBSD-ports/net/opennms-minion
```

Remove the current checksum file:

```sh
rm -f distinfo
```

Update the package version in the `Makefile` to match one available in the  [YUM Repository](http://yum.opennms.org/branches/foundation-2017/common/opennms/):

```sh
OPENNMS_PACKAGE_VERSION=        19.0.0-0.20170411.onms1318.foundation.2017.84
```

Bump up the tag, major, minor, inc and date version in the `Makefile`:

```
OPENNMS_MAJOR_VERSION=  19
OPENNMS_MINOR_VERSION=  0
OPENNMS_INC_VERSION=    0
OPENNMS_DATE_VERSION=   2016.05.05
```

Generate the checksum file:

```sh
make makesum
```

Trigger a new build. See [BUILDING](BUILDING.md) for details.

# Moving to a new branch

```sh
git rebase -i upstream/RELENG_2_3_2
git push -f
git checkout -b RELENG_2_3_3_MINION upstream/RELENG_2_3_3
git cherry-pick adde84279e1736e1fa4d430f7cae38fb443e9202
git push -u origin RELENG_2_3_3_MINION
```

