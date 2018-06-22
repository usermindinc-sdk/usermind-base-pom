# integrations-base-pom
Base pom for Usermind

This brings in many standard libraries and thus lets us have a standard version for them. It also includes Spring, Dropwizard, and Docker meaning it is easy to set those up and will give you everything you need for our CI/CD solution out of the box.

If you do not want your project to have docker, include this snippet in your plugins section:

```          
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
            