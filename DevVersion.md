![https://thelq.ci.cloudbees.com/job/PircBotX/badge/icon?.jpg](https://thelq.ci.cloudbees.com/job/PircBotX/badge/icon?.jpg) ([Jenkins Build Server](https://thelq.ci.cloudbees.com/job/PircBotX/))

Bug fixes and features are being constantly added to PircBotX. The snapshot version allows you test the new changes and make PircBotX better.

**Note:** As this is a development version, the API is subject to change. Its **not** recommended to use this version in a production environment.

# Development Javadocs #

Most recent javadocs are available [here](http://site.pircbotx.googlecode.com/hg/apidocs/index.html).

# Configuration - Maven #

Using Maven is recommended as updates and dependencies will be downloaded automatically.

You will need to change your POM to use the Sonatype snapshot repository and the SNAPSHOT version of PircBotX

```
<dependencies>
	<dependency>
		<groupId>org.pircbotx</groupId>
		<artifactId>pircbotx</artifactId>
		<version>2.1-SNAPSHOT</version>
	</dependency>
</dependencies>
	
<repositories>
	<repository>
		<id>sonatype-nexus-snapshots</id>
		<name>Sonatype Nexus Snapshots</name>
		<url>https://oss.sonatype.org/content/repositories/snapshots</url>
		<releases>
			<enabled>false</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
	</repository>
</repositories>
```

# Configuration - Jar #

If you are just using the Jar, then you can obtain the latest SNAPSHOT by going to [this link](http://oss.sonatype.org/service/local/artifact/maven/redirect?r=snapshots&g=org.pircbotx&a=pircbotx&v=2.1-SNAPSHOT&e=jar). It will automatically download the latest build

The naming convention is as follows:

```
pircbotx-2.1-<buildDate>-<buildNumber>.jar 
```

Use the date and build number to figure out if you have the latest version

You must also obtain the latest versions of the libraries PircBotX uses. See the [Downloads](Downloads.md) page for more information.