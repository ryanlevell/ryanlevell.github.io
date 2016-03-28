---
layout: post
title:  "Publish Project To Maven Central"
date:   2016-03-20 23:03:03 -0500
categories: maven oss sonatype gpg mac
---

#### Reasoning
I am in the early stages of a project called adamant-driver, a Selenium WebDriver + TestNG library to easily combine the two components with very little boiler-plate code. I was updating the readme file and was not satified with my instructions to "manually download the jar and put it on the build path". I wanted the project to be accessible from Maven Central. I am waiting on access to my "maven namespace" to use for my projects, so this will be most likely be a 2 part post. This can all be found at [apache.org](https://maven.apache.org/guides/mini/guide-central-repository-upload.html).

### Summary
Here are the high level steps:

1. Meet the requirements
 * Supply javadoc and source code.
 * Choose a namespace
 * Create a PGP signature
 * Add metadata
2. Publish to Central Repository
 * Create a Jira account
 * Create a Jira ticket
 * To be continued...

### Meet the requirements

1. #### Supply Javadoc and source code
I did this via the `pom.xml`:

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-source-plugin</artifactId>
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
  <artifactId>maven-javadoc-plugin</artifactId>
  <executions>
    <execution>
      <id>attach-javadocss</id>
      <goals>
        <goal>jar</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
These files will be generated with ```mvn package``` and will be located in the ```target``` folder.

2. #### Choose a namespace (AKA coordinates or GAV)
The group-id is the important one. It is the reverse domain name. **And** you must have access to that domain. If you do not have your own domain, your CVS account may be used. Since I do not own ```levell.com```, I had to change mine to use GitHub, ```com.github.<username>```.

```xml
<groupId>com.github.ryanlevell</groupId>
<artifactId>adamant-driver</artifactId>
<version>1.0.0</version>
```

3. #### Create a GPG signature
All files must be signed with GPG so the users can validate the files.

1. Download GPG software from [gnupg.org](https://www.gnupg.org/download/).
 * I used [GPG OSX](https://sourceforge.net/p/gpgosx/docu/Download/).
2. Verify it was successful with ```gpg2 --version```
 * It could also just be ```gpg```
3. Generate a key with ```gpg2 --gen-key```
 * Provide your name, email, and passphrase
4. Distribute the public key

```bash
# the line that starts with pub is needed
~ $: pgp2 --list-keys
pub   rsa2048/ECC54AB7 2016-03-27 [SC]
...
# use the key after rsa2048/ to distribute the it
~ $: gpg2 --keyserver hkp://pool.sks-keyservers.net --send-keys ECC54AB7
gpg: sending key ECC54AB7 to hkp://pool.sks-keyservers.net
```

4. #### Add metadata
The pom must have GAV, name, description, url, license, developers, and SCM tags.

```xml
<name>${project.groupId}:${project.artifactId}</name>
<description>Library to easily create tests using Selenium WebDriver and TestNG.</description>
<url>https://github.com/ryanlevell/adamant-driver</url>
<groupId>com.github.ryanlevell</groupId>
<artifactId>adamant-driver</artifactId>
<version>1.0.0</version>
<licenses>
	<license>
		<name>MIT License</name>
		<url>http://www.opensource.org/licenses/mit-license.php</url>
	</license>
</licenses>
<developers>
	<developer>
		<name>Ryan</name>
		<email>levellapps@gmail.com</email>
	</developer>
</developers>
<scm>
	<url>https://github.com/ryanlevell/adamant-driver.git</url>
</scm>
```

Full details can be found at [sonatype.org](http://central.sonatype.org/pages/requirements.html).  
\* Signing can be done automatically via a built tool such as [Maven](http://central.sonatype.org/pages/apache-maven.html).  
\* Expired keys can be updated shown [here](http://central.sonatype.org/pages/working-with-pgp-signatures.html#dealing-with-expired-keys).

### Publish to Central Repository

1. Create a [Jira account](issues.sonatype.org)
 * Use the same email as you used for GitHub to make verification easier for Jira admin.
2. Login to Jira
3. Create a new ticket
 * Add info
 * Reply to follow-ups, if any
 * Wait for the ticket to be ```resolved``` before continuing
 * My first ticket can be found [here](https://issues.sonatype.org/browse/OSSRH-21458)

### To be continued
I am currently waiting on my ticket to be resolved. I will continue when my ticket has been completed.
