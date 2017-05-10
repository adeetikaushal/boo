# Boo

 The purpose of Boo is to provide a simple command-line tool for the OneOps developers and OneOps users to deploy,
update, destroy and operate their OneOps assemblies.

For the OneOps pack developers, it provides a fast and repeatable way to run tests against complex OneOps packs. For
example, it allows a pack developer to use a short, one-line bash command to verify a parameter or code changes in an
OneOps pack by attemping to instantiate the updated pack and executing tests on the resulting functional cluster. For an
OneOps user, it provides an command-line tool to deploy, update, destroy and operator their OneOps assemblies. The boo
tool is an addition to the OneOps UI interfaces.

## Usage

To use Boo you download the executable JAR and place it in your `$PATH`. Use the latest version available from
[http://repo1.maven.org/maven2/com/oneops/boo/boo](http://repo1.maven.org/maven2/com/oneops/boo/boo) e.g.:

```
curl -O http://repo1.maven.org/maven2/com/oneops/boo/boo/1.0.5/boo-1.0.5-executable.jar

chmod +x boo-1.0.5-executable.jar

mv boo-1.0.5-executable.jar boo

boo <options>
```

To see what options are available for use:

```
boo -h
```

## Configuration

Boo YAML templates are processed with [Mustache][1] to allow variable interpolation when the standard `~/.boo/config` file
contains a `default` profile. If you have a `default` profile that looks like the following:

```
[default]
host=https://localhost:9090
organization=devtools
api_key=oneops_api_key # copy from OneOps UI->profile->authentication->API Token
email=jvanzyl@walmart.com
cloud=prod-cdc6
```

With a Boo YAML template that looks like this:

```
boo:
  oneops_host: '{{host}}'
  organization: '{{organization}}'
  api_key: '{{api_key}}'
  email: '{{email}}'
  environment_name: 'dev'
  ip_output: 'json'
  clouds:
    {{cloud}}:
       priority: '1'
       dpmt_order: '1'
       pct_scale: '100'

...
```

It will yield the following:

```
boo:
  oneops_host: 'https://localhost:9090'
  organization: 'devtools'
  api_key: 'oneops_api_key'
  email: 'jvanzyl@walmart.com'
  environment_name: 'dev'
  ip_output: 'json'
  clouds:
    prod-cdc6:
       priority: '1'
       dpmt_order: '1'
       pct_scale: '100'

...
```
If you want to see what your Boo YAML template will look like with interpolated values you can use the following command:

```
boo -f boo.yml -v
```

## Inlining File Content

You can use the `file` directive to inline content into a Boo template. The following are examples of how you can inline
the content of files:

```
{{file(./id_rsa.pub)}}
{{file(~/.ssh/id_rsa.pub)}}
{{file(/Users/jvanzyl/id_rsa.pub)}}
```

So, for example, if you wanted to inline the content of your public SSH key into your Boo template before execution you
can do the following:

```
user:
  user-jvanzyl:
    system_account: true
    sudoer: true
    username: 'jvanzyl'
    description: "JvZ"
    authorized_keys: '["{{file(~/.ssh/id_rsa.pub}})"]'
```

The result will look something like:

```
user:
  user-jvanzyl:
    system_account: true
    sudoer: true
    username: 'jvanzyl'
    description: "JvZ"
    authorized_keys: '["ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7g6cjv8yxH0pbo..."]'
```

Again, you can see what the result will be using `boo -f your.yml -v`.

# Development

## Build the source code to an executable command:

```
mvn clean package

chmod +x ./target/boo-1.0.5-executable.jar

mv ./target/boo-1.0.5-executable.jar boo

boo <options>
```

## Build to an RPM

```
mvn clean package -Pbuild-rpms

Find the rpm at .//target/rpm/boo/RPMS/noarch/boo-1.0.5*.noarch.rpm
```

## Install the RPM

```
On Linux: sudo rpm -ivh boo-1.0.5*.noarch.rpm
On Mac: sudo rpm -ivh boo-1.0.5*.noarch.rpm --nodeps
```

## Running integration tests

The integration tests take some time to run as they spin up real assemblies, validate them and spin them down. To run the integration tests along with the tests use the following:

```
mvn clean verify -Pits
```

Note that in order to run the integration tests you also need a `default` profile in your `~/.boo/config` that looks like the following:

```
[default]
host=https://localhost:9090
organization=oneops_org_name
api_key=oneops_api_key
email=me@email.com
cloud=cloud_name_in_oneops
```

## Deploying SNAPSHOT

```
mvn clean deploy -P release
```

## Releasing

Provided you have permissions to deploy to OSSRH with a serverId `ossrh` in your settings and the needed gpg setup you
can build and deploy a release with

```
mvn release:prepare
mvn release:perform
```

And then just check and release the staging repo on [https://oss.sonatype.org](https://oss.sonatype.org)


## Code style

Boo uses the Google code style. The formatter for Eclipse you can find here:

https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml


[1]: https://github.com/spullara/mustache.java
