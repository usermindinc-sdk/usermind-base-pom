# integrations-base-pom
Base pom for Usermind

This brings in many standard libraries and thus lets us have a standard version for them. It also includes Spring, Dropwizard, and Docker meaning it is easy to set those up and will give you everything you need for our CI/CD solution out of the box.

Docker and jar shading are configured but turned off unless you specifically turn them on - otherwise they run in libraries and other places where they are not desired. 

To turn on Docker or shading, put the appropriate plugin section below in your project pom:
```          
    <build>
        <plugins>
           <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
            </plugin>
            
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

If you turn on docker, note that you can also add `-Ddockerfile.skip=true` as an argument to your maven commandline to skip that step and make a test build faster. Similarly, `-Dshader.skip=true` will skip shading, again just making a test build faster.
  

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
  

Shaded jars are set up and configured, but turned off so that by default your jar will not be shaded. To turn it on, include this in the derived pom. 

```
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <executions>
                    <execution>
                        <id>assembly</id>
                        <phase>package</phase>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```            