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
            
If you want to disable the shade plugin (for example for a library jar) include this in the pom:

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