---
title: How to deploy with Rultor and not shoot yourself in the leg
layout: post
date: '2023-03-19'
last_modified_at: '2023-03-24'
categories:
  - guide
tags: 
  - ci/cd 
  - rultor
---

<img height="400" title="devops" alt="devops" src="https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fmycalling.ru%2Fwp-content%2Fuploads%2F2015%2F10%2Fsaper.jpg&f=1&nofb=1&ipt=97a587d4503d682789c88270dc1e36a351a6eae194548355114495839535c404&ipo=images">

This guy is trying to set up a CI/CD pipeline. Please don't distract him.

Update! I was saved by Rultor today!

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Today <a href="https://twitter.com/rultors?ref_src=twsrc%5Etfw">@rultors</a> saved a lot of time for me just by not letting me merge branches <a href="https://t.co/pJTpEpdwI7">pic.twitter.com/pJTpEpdwI7</a></p>&mdash; Ivan Ivanchuk (@L3r8y) <a href="https://twitter.com/L3r8y/status/1639228585696165888?ref_src=twsrc%5Etfw">March 24, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

This blog post is an addition to the [previous](/2023/02/22/complete-rultor-setup-guide) post for Java projects that will be published in [Maven Central](https://en.wikipedia.org/wiki/Apache_Maven). Well, if you are reading this, I assume you know about [Rultor](https://www.rultor.com/) and are trying to setup `@rultor release, tag is '1.0.0'`. But for some reason you can't. I'm here to help you deal with your helplessness. So let's begin...




### Register your groupId
The first thing that you need is to [create](https://issues.sonatype.org/secure/CreateIssue!default.jspa) a jira ticket for the new project and register your `groupId`.


### Secret repository
The second thing you need to create, is a **private** repository on [GitHub](https://github.com/new).

After that, you must create a configuration file for rultor `.rultor.yml` and a folder where your secret files will be stored. The usual name for this is `assets`.
Inside `.rultor.yml` you put the repositories that will be given access to your secrets:

```asm
friends:
  - nickname/reponame
```

Now your secret repository should look like this.

```asm
secrets-repo-name
|
|–– .rultor.yml
|–– assets/
```


### GPG keys
**Attention** if you already installed `gpg` and created some key, you should delete it
`gpg --delete-key "YOUR_OLD_KEY"`
`gpg --delete-secret-key "YOUR_OLD_KEY"`
Go to `~/.gnupg` and delete two files: `pubring.gpg` and `secring.gpg`, they may not exist.

**Start**

Well, you should [install](https://gnupg.org/download/index.html#sec-1-2) `gpg`

After that you have to create a new key:

  1. `gpg --full-gen-key`

  2. 
  ```asm
    Please select what kind of key you want:
    (1) RSA and RSA
    (2) DSA and Elgamal
    (3) DSA (sign only)
    (4) RSA (sign only)
    (9) ECC (sign and encrypt) *default*
    (10) ECC (sign only)
    (14) Existing key from card
    Your selection? 1 # choose 1
  ```

  3. ```asm
     RSA keys may be between 1024 and 4096 bits long.
     What keysize do you want? (3072) 2048 # choose 2048
     ```

  4. 
  ```asm  
    Please specify how long the key should be valid.
    0 = key does not expire
    <n>  = key expires in n days
    <n>w = key expires in n weeks
    <n>m = key expires in n months
    <n>y = key expires in n years
    Key is valid for? (0)  0 # choose 0
  ```

  5. Next steps are pretty obvious. Comment should be left empty.
   
  6. After you've created a key you should upload it into couple of servers:
      
      `gpg --keyserver keys.openpgp.org --send-keys YOUR_KEY`

      `gpg --keyserver keyserver.ubuntu.com --send-keys YOUR_KEY`
   
  7. You see a message like `gpg: sending key KEY_ID to SERVER_NAME`, okay, you should save this `KEY_ID` - it looks like a mess of numbers and letters.
   
  8. Now, you have to create the `secring.gpg` and `pubring.gpg`

     `gpg --export > ~/.gnupg/pubring.gpg`

     `gpg --keyring secring.gpg --export-secret-keys > ~/.gnupg/secring.gpg`
       
  9. Load these files from `~/.gnupg` in the secret repository which you created before.

  10. Create file `assets/settings.xml` which looks like:

      ```asm
      <?xml version="1.0" encoding="UTF-8"?>
      <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
          <servers>
              <server>
                  <id>ossrh</id>
                  <username>JIRA_USERNAME</username>
                  <password>JIRA_PASSWORD</password>
              </server>
          </servers>
          <profiles>
              <profile>
                  <id>ossrh</id>
                  <activation>
                      <activeByDefault>true</activeByDefault>
                  </activation>
                  <properties>
                      <gpg.passphrase>passphrase</gpg.passphrase>
                      <gpg.keyname>KEY_ID</gpg.keyname>
                      <gpg.homedir>/home/r</gpg.homedir>
                  </properties>
              </profile>
          </profiles>
      </settings> 
      ```

  10. Add `KEY_ID` that you saved in `settings.xml` into field `<gpg.keyname>KEY_ID</gpg.keyname>`
  
  12. Add your `passphrase` into `<gpg.passphrase>passphrase</gpg.passphrase>`

  13. Add your jira `username` and `password` that you used to create the ticket.

Your secret repository now looks like this.
```asm
secrets-repo-name
|
|–– .rultor.yml
|–– assets/
    |
    |–– settings.xml
    |–– secring.gpg
    |–– pubring.gpg
```

### Configure main repository
There are only the necessary things, I do not provide the complete `pom.xml` file.

```asm
...
<scm>
  <connection>scm:git:git@github.com:l3r8yJ/sa-tan.git</connection>
  <developerConnection>scm:git:ssh://@github.com:l3r8yJ/sa-tan.git</developerConnection>
  <url>https://github.com/l3r8yJ/sa-tan/tree/master</url>
</scm>
<developers>
  <developer>
    <name>Your Name</name>
    <email>Your Email</email>
    <roles>
      <role>Your Role</role>
    </roles>
  </developer>
</developers>
<licenses>
  <license>
    <name>Your License</name>
    <url>https://www.opensource.org/licenses/mit-license.php</url>
  </license>
</licenses>
<ciManagement>
  <system>Your CI</system>
  <url>https://www.rultor.com/</url>
</ciManagement>
...
...
<profiles>
  <profile>
    <id>release</id>
    <activation>
      <activeByDefault>false</activeByDefault>
      <property>
        <name>gpg.keyname</name>
      </property>
    </activation>
    <distributionManagement>
      <snapshotRepository>
        <id>ossrh</id>
        <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
      </snapshotRepository>
      <repository>
        <id>ossrh</id>
        <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
      </repository>
    </distributionManagement>
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-javadoc-plugin</artifactId>
          <version>3.5.0</version>
          <executions>
            <execution>
              <id>attach-javadocs</id>
              <goals>
                <goal>jar</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-source-plugin</artifactId>
          <version>3.2.1</version>
          <executions>
            <execution>
              <id>attach-sources</id>
              <goals>
                <goal>jar</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-gpg-plugin</artifactId>
          <version>3.0.1</version>
          <configuration>
            <gpgArguments>
              <arg>--pinentry-mode</arg>
              <arg>loopback</arg>
            </gpgArguments>
          </configuration>
          <executions>
            <execution>
              <id>sign-artifacts</id>
              <phase>verify</phase>
              <goals>
                <goal>sign</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
        <plugin>
          <groupId>org.sonatype.plugins</groupId>
          <artifactId>nexus-staging-maven-plugin</artifactId>
          <version>1.6.13</version>
          <extensions>true</extensions>
          <configuration>
            <serverId>ossrh</serverId>
            <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
            <autoReleaseAfterClose>true</autoReleaseAfterClose>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```

Then you need to configure your `.rultor.yml` inside the project repository.
```asm
assets:
  settings.xml: nickname/secret_repo#assets/settings.xml
  secring.gpg: nickname/secret_repo#assets/secring.gpg
  pubring.gpg: nickname/secret_repo#assets/pubring.gpg

release:
  pre: false
  sensetive:
    - settings.xml
  script: |-
    [[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || exit -1
    mvn versions:set "-DnewVersion=${tag}"
    git commit -am "${tag}"
    mvn clean deploy -Prelease --errors --settings ../settings.xml
```

### Done

Now you can go to some issue and try `@rultor release`!



*Thank you; I hope this post was interesting for you, also you can correct me in the comments if I made mistakes, etc.*
