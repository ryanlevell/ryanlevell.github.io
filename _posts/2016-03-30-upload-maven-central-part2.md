---
layout: post
title:  "Publish Project To Maven Central Part 2"
date:   2016-03-30 22:28:03 -0500
categories: maven oss sonatype gpg mac
---

### [Part 1](http://ryanlevell.github.io/maven/oss/sonatype/gpg/mac/2016/03/28/upload-maven-central.html)
1. Met the requirements
2. Created a Jira ticket for access to Nexus Repository Manager

### Part 2
1. Set up Nexus Repository Manager
 * Login
 * Add distribution management
 * Generate user token
2. Publish to Maven Central
 * Deploy with Maven
 * PGP Sign with Maven
 * Push to Maven Central

### Set up [Nexus Repository Manager](https://oss.sonatype.org/)
This is where you can manage projects that are ready to be pushed to Maven Central and verify they meet the requirements.

#### Login
1. Go to the [homepage](https://oss.sonatype.org/)
2. Sign in using your Jira credentials

#### Add [distribution management](http://central.sonatype.org/pages/apache-maven.html#distribution-management-and-authentication)
Configure maven to deploy to Open Source Software Repository Hosting (OSSRH) Nexus Repository Manager by adding the ```<distributionManagement>``` tag.

```xml
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
  <repository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
  </repository>
</distributionManagement>
```

#### Generate User Token
This will be used to validate you can access Nexus Repository Manager. I believe you can also use your real password, this is just a security feature.

1. Login to [Nexus Repository Manager](https://oss.sonatype.org/)
2. Click ```<Username> > Profile```
3. Click ```Summary``` dropdown and select ```User Token```
4. Click ```Access User Token```
5. Enter password when prompted

These steps will reveal the XML:

```xml
<server>
  <id>${server}</id>
  <username><your username token></username>
  <password><your password token></password>
</server>
```

6. Add this text to ```~/.m2/settings.xml```
7. Run ```mvn deploy``` to test pushing to Nexus Repository Manager

I kept getting an authentication error. My problem was ```settings.xml``` was not formatted correctly. The above XML must be nested in two additional tags:

```xml
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username><your username token></username>
      <password><your password token></password>
    </server>
  </servers>
<settings>
```

If ```mvn deploy``` was successful, the project is ready for Maven Central.

### Publish to Maven Central
These are the last few steps before the project will be accessible via Maven Central.

#### Deploy with Maven
As described above, ```mvn deploy``` will push the project to Nexus Repository Manager. Two things can happen:

1. It will be pushed to the snapshot repo.
 * This repo will be used when ```-SNAPSHOT``` is appended to the ```<version>``` in ```pom.xml```
 * Copy the upload link and verify the files exist in a browser:
 * ```https://oss.sonatype.org/content/repositories/snapshots/com/<your group id>/<your artifact id>/```
 * [Stackoverflow](http://stackoverflow.com/a/5907727/807183) can better explain the purpose of the snapshot repository.
2. It will be pushed to the release repo.
 * This repo will be used when the ```<version>``` is not followed by ```-SNAPSHOT```
 * This is the repo that must be used to push to Maven Central.
 * You cannot view the upload link in a browser, Nexus Repository Manager must be used.
 1. Login to Nexus Repository Manager
 2. Click ```Staging Repositories```
 3. Scroll to bottom of Repository list
 4. Select your repository
 5. Click ```Close```
 6. Leave a comment like ```Ready to deploy```
 7. Click ```Refresh``` to view that it was closed successfully.
 * This is where all the requirements will be checked. It will not close if any of the requirements do not pass.  
8. Click on your repository again
9. Click the ```Activity``` tab
10. View any failed requirements

This is where I failed PGP signing requirement.

#### PGP Sign with Maven
1. Add the PGP plugin to ```pom.xml```

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-gpg-plugin</artifactId>
  <version>1.5</version>
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
```

2. Add OSSRH profile to ```settings.xml```

```xml
<settings>
  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.executable>gpg2</gpg.executable>
        <!-- I still have to enter key manually so not sure this does anything -->
        <gpg.passphrase><key password here></gpg.passphrase>
      </properties>
    </profile>
  </profiles>
  ...
</settings>
```

Rerun ```mvn deploy``` and signing should be handled by Maven.

#### Push to Maven Central
All requirements should now be passed.

1. Run ```mvn deploy``` again if not already
2. Find and close your repo again in Nexus Repository Manager
3. Refresh again and verify all requirements passed in the ```Activity``` tab
4. Select your repo again
 * The ```Release``` button should now be enabled
6. Click ```Release```
7. Add a comment ```Version x.x.x and bug fix xyz```
8. Refresh and your repo should now be gone.

At this point the project will soon be on Maven Central.  
At the time of this writing it takes ~10 minutes to be usable.  
It won't be searchable on [search.maven.org](http://search.maven.org/) for ~2 hours.  
It took my [project](http://mvnrepository.com/artifact/com.github.ryanlevell/adamant-driver) about 1 day to be found on [mvnrepository.com](http://mvnrepository.com/).
