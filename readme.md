# Lucee Microsoft SQL Server OSGi JAR Builder

This Maven project can be used to re-compile the Microsoft SQL Server JDBC drivers into a OSGi JAR compatible with [Lucee](https://www.lucee.org).

## Requirements

In order to use this Maven project to build your own JAR files, there are a few things you need to do:

1. Make sure you have [GnuPG](https://gnupg.org/) installed. This needs to be configured in order for the JAR to be signed. You can add a Maven `settings.xml` to specify your path to GPG and to set you passphrase if you want do not want to enter it on each build. Here's an example `settings.xml` file:
	```
	<?xml version="1.0"?>
	<settings>
			<profiles>
					<profile>
							<activation>
									<activeByDefault>true</activeByDefault>
							</activation>
							<properties>
									<gpg.executable>gpg</gpg.executable>
									<gpg.passphrase>your_secret_passphrase_here</gpg.passphrase>
							</properties>
					</profile>
			</profiles>
	</settings>
	```
	* Storing passphrases in clear text is a major security issue. See the [Maven Password Encryption](https://maven.apache.org/guides/mini/guide-encryption.html) guide for how to store your passphrase in an encrypted format.
	* See [Maven Settings Reference](https://maven.apache.org/settings.html) for information on where the `settings.xml` should be placed and for other options.

		**NOTE** — The `${maven.home}` path in Windows is the same as `%USERPROFILE%`.

## Building your JAR

To build your OSGi-compliant JAR, you can just run the maven `install` commands. This will download the Microsoft SQL JDBC driver and build a new JAR with the fixed OSGi manifest that will work with Lucee.

If you want to try building a different version of the driver or to create a JAR for a different target JRE, you will need to change the `target` properties located at the top of the `pom.xml`. This Maven project does not recompile any classes, it just creates a new JAR with the correct OSGi headers, so changing the target JRE will just download the appropriate classes from the Maven repo.

## Building older versions of the Microsoft SQL Server JDBC drivers

Unfortunately, different versions of the Microsoft SQL Server driver have different external dependencies so in order to create the OSGi JAR, the Felix configuration will need to be updated to reflect the correct external dependencies.

The following provides the `<Import-Package>` and `<DynamicImport-Package>` definitions for older versions of drivers. If you plan on creating a JAR for an older version of the drivers, you will need to update the `pom.xml` with the configuration rules below.

### mssql-jdbc v8.4.1
```
<Import-Package>
	!org.bouncycastle.openssl.jcajce,
	!com.google.gson,
	!org.antlr.v4.runtime.dfa,
	!org.antlr.v4.runtime.atn,
	!org.antlr.v4.runtime,
	!org.osgi.framework,
	!org.osgi.service.jdbc,
	!com.microsoft.aad.adal4j,
	!com.microsoft.azure,
	!com.microsoft.azure.keyvault,
	!com.microsoft.azure.keyvault.authentication,
	!com.microsoft.azure.keyvault.models,
	!com.microsoft.azure.keyvault.webkey,
	!com.microsoft.azure.serializer,
	!com.microsoft.rest,
	!com.microsoft.rest.credentials,
	!com.microsoft.rest.protocol,
	!okhttp3,
	!retrofit2,
	*
</Import-Package>
<DynamicImport-Package>
	org.bouncycastle.openssl.jcajce,
	com.google.gson,
	org.antlr.v4.runtime.dfa,
	org.antlr.v4.runtime.atn,
	org.antlr.v4.runtime,
	org.osgi.framework,
	org.osgi.service.jdbc,
	com.microsoft.aad.adal4j,
	com.microsoft.azure,
	com.microsoft.azure.keyvault,
	com.microsoft.azure.keyvault.authentication,
	com.microsoft.azure.keyvault.models,
	com.microsoft.azure.keyvault.webkey,
	com.microsoft.azure.serializer,
	com.microsoft.rest,
	com.microsoft.rest.credentials,
	com.microsoft.rest.protocol,
	okhttp3,
	retrofit2
</DynamicImport-Package>
```

## Using with a local MSSQL JAR build

In order to use this with a local MSSQL JAR build, there are several steps involved. The first steps are to make sure your local environment is configured correctly to run the Maven build.

1. Make sure that the "maven-bundle-plugin" a newer version than 3.0.1. The older version contains some outdated libraries that may cause issues.

Once you have the initial steps configured, you should try to do a `clean install` of the `pom.xml` as-is, to make sure it runs successfully. If there are errors, you should track those down before going any further.

Once you can successfully do a `clean install` of the Maven file, the next step is to update the `pom.xml` to update your custom JAR.

1. Create a `/lib/` folder and copy the new JAR to this folder.
2. Next, you will need to run the `install:install-file` Maven command to install/update the JAR in your local repository. To do that, right-click on the Maven project and select `Custom` and enter the following command:
	```
	install:install-file -DgroupId="<UNIQUE-GROUP-ID>" -DartifactId="<UNIQUE-ARTIFACT-ID>" -Dversion="<UNIQUE-VERSION>" -Dpackaging=jar -DgeneratePom=true -Dfile="./lib/<JAR-FILENAME>"
	```
	**NOTE** — You will need to replace the tokens in between the `<` and `>` markers with your own values. You will need to remember these values, because you will need to update the `pom.xml` with the same values. You can use `com.microsoft.sqlserver` for the `<UNIQUE-GROUP-ID>`, but you will need a unique `<UNIQUE-ARTIFACT-ID>` to make sure you do not conflict with any Maven repository artifacts.
3. Now you must make some changes to the `pom.xml` file.
	* Change the `<target.mssql.version>` element to reflect the MSSQL driver version you are using.
	* Change the `<target.jre.version>` element to reflect the target JRE of the driver.
	* Change the `<mssql.groupId>` element to the match the `<UNIQUE-GROUP-ID>` value you used when adding the JAR to your local Maven repository above.
	* Change the `<mssql.version>` element to the match the `<UNIQUE-VERSION>` value you used when adding the JAR to your local Maven repository above.
	* Change the `<mssql.artifactId>` element to the match the `<UNIQUE-ARTIFACT-ID>` value you used when adding the JAR to your local Maven repository above.

Once the above is done, you should be able to run a `clean install` to generate the fixed OSGi build of your local SQL Server JAR.

## Using your OSGi JAR

Once you have your new JAR file created, use the [extension-jdbc-mssql](https://github.com/lucee/extension-jdbc-mssql) project to build your new extension. 

You will first need to [downloaded the project](https://github.com/lucee/extension-jdbc-mssql/archive/refs/heads/master.zip) and unzip it to a folder on your system.

Next, delete the existing jar (e.g. `org.lucee.mssql-8.4.1.jre8.jar`) and copy the JAR file from the `target` folder of this project into the folder with your `extension-jdbc-mssql` files.

Once this is done, you can run the ANT `build.xml` script's `clean` target. This will bundle up all the required files and create a new `lex` file in the `dist` folder.

You can now upload this extension in the Lucee Administrator to install the updated driver.

## Troubleshooting

If you are trying to compile a new version of the driver, it's possible that you may into errors when trying to install your new driver if there are new dependencies that have not been accounted for you your `pom.xml`.

When this happens, you may see an error like the following when trying to deploy your new extension in Lucee:

```
Unable to resolve org.lucee.mssql [68](R 68.0): missing requirement [org.lucee.mssql [68](R 68.0)] osgi.wiring.package; (osgi.wiring.package=com.azure.core.credential) Unresolved requirements: [[org.lucee.mssql [68](R 68.0)] osgi.wiring.package; (osgi.wiring.package=com.azure.core.credential)]
```

When you see this error, you must update the `<Import-Package>` and `<DynamicImport-Package>` in the `pom.xml` to define the missing dependency. 

For example, if you got the above error that the `com.azure.core.credential` package was missing, you would prepend the following rule to the `<Import-Package>` element:

```
!com.azure.core.credential,
```

This will make sure the JAR's manifest says that this library is not required in order to load.

Next, you need to prepend the following string to the `<DynamicImport-Package>` element:

```
com.azure.core.credential,
```

This will make sure the JAR's manifest says that the dependency should be loaded if it exists.

Once you have updated the `pom.xml` you will need to rebuild your extension and re-deploy it.

> **IMPORTANT** — Some versions of Lucee have a bug in which a deployed extension that has bad OSGi headers must be removed manually before you can deploy a new version (see [Extensions with OSGi wiring issues cannot be updated without manually purging installed files](https://luceeserver.atlassian.net/browse/LDEV-3759)). If you try updating your extension and are still seeing the same error, you will need to manually remove the extension files that Lucee unpacked. 
> 
> The steps are something like the following:
> 
> 1. Stop Lucee (e.g. `service tomcat.service stop`)
> 2. Find the installed files (e.g. `find /opt -name *mssql*`)
> 	* Find the path to the JAR file, and delete it (e.g. `rm -f /opt/lucee/config/server/lucee-server/bundles/org.lucee.mssql*.jar`)
> 	* Find the path to the cfclasses cache folders and remove them (e.g. `rm -fr /opt/lucee/config/web/d2d154bed8cf37f7d84aac7b28f70446/cfclasses/`)
> 3. Start Lucee (e.g. `service tomcat.service start`)
