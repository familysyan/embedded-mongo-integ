# embedded-mongo-integ
Ant script that helps embed mongodb to your integration test. You will be able to start a standalone mongodb instance by just running maven. No pre installation required. This is inspired by https://github.com/elastic/elasticsearch/blob/2.4/dev-tools/src/main/resources/ant/integration-tests.xml

# What does this script do?
* download mongo db
* unzip/untar it
* launch the instance within your project during maven pre-integration-test phase
* run your integration tests against this mongo instance
* stop the instance after the integration test
* clean up all the temp files so that next time you run your integration tests it's brand new.

# How to use it?
1. Put the ant script into somewhere in your maven project. for example: src/it/ant
2. Use failsafe plugin to run your integration tests. Although you can still use surefire but it's highly recommended to use failsafe because it guarantees that post-integration-test phase will get executed even when there are test failures. So that you can make sure your mongodb shutdown property and won't leave your port open after the tests.
Example failsafe plugin - 
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <skipTests>
            ${skipIntegTests}
        </skipTests>
        <argLine>-Dtests.security.manager=false</argLine>
    </configuration>
</plugin>
```
2. Add the below plugins to your pom.xml
```
<!-- Run ant script for you -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>
    <version>1.8</version>
    <executions>
        <!-- start mongodb before your integration tests -->
        <execution>
            <id>mongodb-integ-setup</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <failOnError>false</failOnError>
                <skip>${skipIntegTests}</skip>
                <target>
                    <ant antfile="src/it/ant/mongodb-integration-tests.xml" target="start-external-cluster"/>
                </target>
            </configuration>
        </execution>
        
        <!-- shut down mongodb after your integration tests -->
        <execution>
            <id>mongodb-integ-teardown</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <failOnError>false</failOnError>
                <skip>${skipIntegTests}</skip>
                <target>
                    <ant antfile="src/it/ant/mongodb-integration-tests.xml" target="stop-external-cluster"/>
                </target>
            </configuration>
        </execution>
    </executions>
</plugin>
```
3. Declare a few properties in your pom.xml
```
<properties>
    <!-- what version do you want -->
    <mongodb.version>3.4.2</mongodb.version> 
    <!-- mongodb port -->
    <mongodb.integ.port>27017</mongodb.integ.port>
    <!-- if you want to be able to skip your integratin test -->
    <skipTests>false</skipTests>
    <skipIntegTests>${skipTests}</skipIntegTests>
</properties>
```

# Testing
The script is supposed to be platform independent but I have only tested it on my mac. If you use windows or other unix based OS you might run into issues. If that happens just take a look at the mac flow and make whatever changes necessary.
