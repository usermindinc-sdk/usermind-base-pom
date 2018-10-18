# integrations-base-pom
Base pom for Usermind

This brings in many standard libraries and thus lets us have a standard version for them. It also includes Spring, Dropwizard, and Docker meaning it is easy to set those up and will give you everything you need for our CI/CD solution out of the box.

If you do not want your project to have docker, include this snippet in your plugins section:

```          
    <build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

You can also add `-Ddockerfile.skip=true` as an argument to your maven commandline.  

Dockerfile notes!
Set up your username and password in Maven as described here:          
https://usermind.atlassian.net/wiki/spaces/EN/pages/26312740/Setting+Up+Maven

There is a docker issue with Spotify where it can't read the OSX Keychain. So if your build fails with a docker login issue even after you log in, check the file ~/.docker/config.json. If it has the line
```
"credsStore": "osxkeychain"
```
then remove it (and the comma before it so it stays valid JSON) and run 
```
docker login
```
again. That should recreate the credentials without using the keychain. On reboot docker will restart and put the keychain credentials back in, though, so you'll need to do this again. I just copied config.json to config.worked, and then instead of logging in I just copied the file back over.
  
For reference, here is a suggested entry for a shaded jar:
```
            <!--Put all dependencies into a single jar-->
            <!--https://maven.apache.org/plugins/maven-shade-plugin/-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <id>assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>${main-class}</Main-Class>
                                        <Build-Job>${build-job}</Build-Job>
                                        <Build-Number>${build-number}</Build-Number>
                                        <Build-Timestamp>${build-timestamp}</Build-Timestamp>
                                        <Build-Url>${build-url}</Build-Url>
                                        <Build-Version>${build-version}</Build-Version>
                                        <Git-Commit>${git-commit}</Git-Commit>
                                        <Git-Branch>${git-branch}</Git-Branch>
                                        <Implementation-Title>${project.name}</Implementation-Title>
                                        <Implementation-Vendor>${project.organization.name}</Implementation-Vendor>
                                        <Implementation-Version>${build-version}</Implementation-Version>
                                    </manifestEntries>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.schemas</resource>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.handlers</resource>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.tooling</resource>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer" />
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

```             

And for reference, if you want to disable the shade plugin (or any other plugin) in a derived jar, include this in the derived pom. It sets the execution phase to nothing, which means that step will never trigger.

```
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <executions>
                    <execution>
                        <id>assembly</id>
                        <phase/>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```            