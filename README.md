# Minion meets pfSense

## Build the packages

Build from source, or skip to the next step to use the existing binaries.

Tested with a 64-bit images of FreeBSD 10.2 on Digital Ocean.

```sh
$ pkg install git
$ git clone git@github.com:j-white/FreeBSD-ports.git pfSense-ports
$ cd pfSense-ports/net/pfSense-pkg-minion/
$ sudo pkg install openjdk8
$ sudo make package
```

## Minion Setup

Tested with pfSense v2.2.6 on a Netgate SG-2220 w/ 60GB SSD

Fetch and install the package:

```sh
cd /tmp
fetch http://104.236.215.163/packages/pfSense-pkg-minion-1.0.0.txz
pkg install pfSense-pkg-minion-1.0.0.txz
```

Add the following snippets to the `<installedpackages>` tag in `/conf/config.xml`:

```xml
<minion>
  <config>
    <enable>on</enable>
    <httpurl>http://opennms/opennms</httpurl>
    <brokerurl>tcp://opennms:61616</brokerurl>
    <location>HOME</location>
  </config>
</minion>
```

```xml
<menu>
  <name>Minion</name>
  <section>Services</section>
  <url>/pkg_edit.php?xml=minion.xml</url>
</menu>
```

```xml
<service>
  <name>minion</name>
  <rcfile>minion-daemon.sh</rcfile>
  <executable>minion-daemon</executable>
  <description><![CDATA[Minion]]></description>
</service>
```

Delete the configuration file cache:

```sh
rm -f /tmp/config.cache
```

Refresh the WebUI, access `Services -> Minion` and hit `Save`.

## Verify your Minion

From the pfSense shell, verify that the Karaf shell is accessible:

```sh
ssh -oPort=8201 admin@localhost
```

From the Karaf shell, verify that the backend is reachable:

```sh
admin@minion>minion:ping
```

Now reboot the appliance, and verify that the service is restarted on reboot, and that all of the checks above still pass.

Congratulations, you're ready to go.

## Know Issues

* Karaf shell SSH key changes on restart
* Status -> Services doesn't show the status for the Minion service or allow you to restart it
