# Updating the Minion port

```sh
cd ~/FreeBSD-ports/net/opennms-minion
```

Remove the current checksum file:

```sh
rm -f distinfo
```

Update the package version in the `Makefile` to match one available in the  [YUM Repository](http://yum.opennms.org/bleeding/common/opennms/):

```sh
OPENNMS_PACKAGE_VERSION=        19.0.0-0.20160505.onms.develop.443
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
