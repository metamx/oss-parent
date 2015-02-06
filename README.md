# Metamarkets Parent POMs

## Usage

Add the following snippet at the top of your project pom.

```xml
<parent>
    <groupId>com.metamx</groupId>
    <artifactId>oss-parent</artifactId>
    <version>1</version>
</parent>
```

Make sure your project pom includes an acceptable name, url, proper license,
as well as a developers section.

```xml
<name>${project.groupId}:${project.artifactId}</name>
<url>https://github.com/metamx/<project></url>

<licenses>
    <license>
        <name>Apache License, Version 2.0</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0</url>
    </license>
</licenses>

<developers>
    <developer>
        <name>John Doe</name>
        <email>john.doe@metamarkets.com</email>
        <organization>Metamarkets Group Inc.</organization>
        <organizationUrl>https://www.metamarkets.com</organizationUrl>
    </developer>
</developers>
```

Include the maven release plugin as part of your `<build><plugins>` section.
Release plugin version and configuration are taken care of by the parent pom.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
</plugin>
```

## Deploying Artifact

### Requirements

1. [Create a Sonatype account](https://issues.sonatype.org/secure/Signup!default.jspa)
   and request to be added to the list of users with deployment permissions.

1. Install GPG

  ```
  brew install gpg2 gpg-agent pinentry-mac
  # setup pinentry for gpg-agent
  echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
  ```

1. Create a GPG key and publish it

  - Generate the key

    ```bash
    gpg2 --gen-key
    ```

    Use the default values for type, size, expiration, and add your name and
    email

  - Upload your key to a key-server

    ```bash
    gpg2 --list-keys
    ```

        --------------------------------
        pub   4096R/ABCDEF12 2015-02-04
        uid       [ultimate] John Doe <johndoe@example.com>
        ...

    ```bash
    # use the key id as shown by gpg2 --list-keys
    gpg2 --keyserver hkp://pool.sks-keyservers.net --send-keys ABCDEF12
    ```

1. Add the required server entries to your maven `settings.xml` file.

  - Add the `ossrh` server entry to the `<servers>` list.  Replace `username` and `password` with your Sonatype username and password.

    ```xml
    <!-- OSSRH -->
    <server>
      <id>ossrh</id>
      <username>username</username>
      <password>mypassword</password>
    </server>
    ```

    See [password encryption](http://maven.apache.org/guides/mini/guide-encryption.html)
    for information on how to encrypt your maven passwords.

  - Add the following ossrh profile to the `<profiles>` list.

    ```xml
    <profile>
        <id>ossrh</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <gpg.executable>gpg2</gpg.executable>
        </properties>
    </profile>
    ```

### Releasing Artifacts

At the root of your project, do the following

```bash
# Make sure gpg-agent is running
eval $(gpg-agent --daemon)

# Prepare your release
mvn release:clean release:prepare

# Perform release
mvn release:perform -Dgpg.useagent=true

# If the release looks good, promote the artifact from staging to release
cd target/checkout
mvn -Prelease nexus-staging:release
```

## Making changes to the parent POM

### How to publish the parent POM to Sonatype

***Note:** this is only required if you need to make changes to the parent pom.
To deploy artifacts using the parent pom, see above.

```bash
# Make sure gpg-agent is running
eval $(gpg-agent --daemon)

# Prepare a release for the parent pom
mvn release:prepare

# Check out the tag for the version we just created
git checkout $(git tag -l --sort=version:refname | tail -1)

# Extract the version for the tag
export version=$(xpath pom.xml "/project/version/text()" 2>/dev/null)

# Create  a symbolic link to the pom with the right version
ln -s pom.xml oss-parent-${version}.xml

# Sign and publish the the pom
mvn gpg:sign-and-deploy-file \
  -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ \
  -DrepositoryId=ossrh \
  -DpomFile=oss-parent-${version}.xml \
  -Dfile=oss-parent-${version}.xml
```
