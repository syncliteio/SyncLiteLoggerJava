# Publishing synclite to Sonatype Central

This note captures the exact steps needed to publish the Java artifact from this repository.

## 1. Prerequisites

### Maven credentials
Make sure your Maven settings file contains the Sonatype Central token for the `central` server.

Example:

```xml
<settings>
  <servers>
    <server>
      <id>central</id>
      <username>YOUR_SONATYPE_USERNAME</username>
      <password>YOUR_SONATYPE_TOKEN</password>
    </server>
  </servers>
</settings>
```

### GPG signing
You need a private GPG key that can sign the artifacts.

Install GnuPG if missing:

```bash
sudo apt-get update
sudo apt-get install -y gnupg pinentry-curses
```

Make sure the agent can prompt interactively:

```bash
mkdir -p ~/.gnupg
printf 'pinentry-program /usr/bin/pinentry-curses\nallow-loopback-pinentry\n' > ~/.gnupg/gpg-agent.conf
chmod 600 ~/.gnupg/gpg-agent.conf

gpgconf --kill gpg-agent
gpgconf --launch gpg-agent
```

If you are setting this up on a new machine or VM, you can either import your existing exported keys or generate a fresh key pair there. For a fresh key, use:

```bash
gpg --batch --passphrase '' --quick-generate-key "your-name <your-email>" rsa3072 sign 1y
```

If you already exported the old keys, import them instead before testing signing:

```bash
gpg --import /tmp/synclite-public.asc
gpg --import /tmp/synclite-private.asc
```

Verify that GPG can sign:

```bash
export GPG_TTY=$(tty)
echo "test" | gpg --clearsign
```

Enter your passphrase if prompted.

## 2. Prepare the release artifacts

Before publishing, make sure the build output and the POM version match the release you want to publish.

### Update the version in the POM
Open [pom.xml](pom.xml) and verify that the version is correct.

```xml
<groupId>io.synclite</groupId>
<artifactId>synclite</artifactId>
<version>1.0.0</version>
```

If you need a new release version, update it before publishing.

### Ensure the release artifact and version are correct
The publish step uses the jar produced by the build under [target](target), not a jar copied from the resources folder. Before publishing, confirm that the expected jar and version are present.

```bash
cd /workspaces/SyncLiteLoggerJava
ls -1 target/synclite-*.jar
```

If you are changing the version, replace `1.0.0` with the version you intend to publish in [pom.xml](pom.xml) and ensure the corresponding artifact file names match.

## 3. Publish from the repo root

```bash
cd /workspaces/SyncLiteLoggerJava
export GPG_TTY=$(tty)
mvn -DskipTests deploy
```

If you want a clean rebuild first:

```bash
cd /workspaces/SyncLiteLoggerJava
export GPG_TTY=$(tty)
mvn clean deploy -DskipTests
```

## 4. Notes

- The artifact coordinates are:
  - Group ID: `io.synclite`
  - Artifact ID: `synclite`
  - Version: `1.0.0`
- The POM already uses the Sonatype Central publishing plugin and the `central` server ID.
- If you move to a new machine or VM, copy over:
  - the Maven settings file with the token
  - the private GPG key and public key
  - the repository contents
