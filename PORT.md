# Updating the Minion port

Poudriere only enables internet access during the fetch phase of the port build.
In order to build OpenNMS, we need to gather all of the required Maven artifats.

## Generate the m2 file

Cleanup the current repository:

```sh
rm -rf ~/.m2/repository
```

Build:

```sh
./clean.pl && ./compile.pl -DskipTests=true && ./assemble.pl -DskipTests=true
```

```sh
cd ~/.m2
find -s repository -type d -name '19.0.0-SNAPSHOT' -exec rm -rf {} +
tar zcvf FreeBSD-opennms-19.0.0.2016.04.28-maven-repository.tar.gz repository
```

## Updating the port

Remove the current checksum file:

```sh
rm -f distinfo
```

Bump up the tag, major, minor, inc and date version in the Makefile:

```
GH_TAG_NAME=	79f4b1213
OPENNMS_MAJOR_VERSION=  19
OPENNMS_MINOR_VERSION=  0
OPENNMS_INC_VERSION=    0
OPENNMS_DATE_VERSION=   2016.04.28
```

Generate the checksum file:

```sh
make makesum
```
