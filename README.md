# jqassistant Example App
This project has the purpose of showing what we want to achieve using jqassistant.
We want to make sure that our application relies directly only on the usage of logback and not on other logging mechanisms.

To do so, some jqassistant rules have been created (/jqassistant/rules.xml).

```
<constraint id="logging:use-logback">
        <description>Ensures that Logback is used for logging.</description>
        <cypher><![CDATA[
              MATCH
                (app:Application)-[:CONTAINS]->(type:Type)
              WHERE NOT (
                (type)-[:DECLARES]->()-[:OF_TYPE]->(:Type {fqn:"ch.qos.logback.classic.Logger"})
                OR
                (type)-[:DECLARES]->()-[:INVOKES]->()<-[:DECLARES]-(:Type {fqn:"org.slf4j.LoggerFactory"})
              )
              RETURN type.fqn AS InvalidType
            ]]>
        </cypher>
    </constraint>

    <constraint id="logging:only-logback">
        <description>Ensure only Logback is used as the logging framework.</description>
        <cypher><![CDATA[
        MATCH (dep:Dependency)-[:DEPENDS_ON]->(artifact:Artifact)
        WHERE artifact.groupId IN ['org.slf4j', 'log4j', 'commons-logging', 'java.util.logging']
          AND NOT (artifact.groupId = 'org.slf4j' AND artifact.artifactId IN ['slf4j-api', 'jul-to-slf4j'])
          AND artifact.artifactId <> 'logback-classic'
        RETURN dep.fqn AS InvalidDependency, artifact.groupId AS GroupId, artifact.artifactId AS ArtifactId
        ]]></cypher>
    </constraint>
```

Since the default logging mechanism for springboot is logback, the result of jqassistant:analyze is successful.
To test that the rules are working fine and that the result of jqassistant:analyze is not a false positive, a third rule has been created which should deny the specific dependency to org.springframework.boot:spring-boot-starter-log4j2:3.1.3.
After the change, all the three rules result successful, so we can tell that there is something wrong either in our logic or in the way the rules were written.
```
    <constraint id="logging:deny-spring-boot-starter-log4j2">
        <description>Deny usage of spring-boot-starter-log4j2</description>
        <cypher><![CDATA[
        MATCH (dep:Dependency)-[:DEPENDS_ON]->(artifact:Artifact)
        WHERE artifact.groupId = 'org.springframework.boot'
        AND artifact.artifactId = 'spring-boot-starter-log4j2'
        AND artifact.version = '3.1.3'
        RETURN dep.fqn AS InvalidDependency, artifact.groupId AS GroupId, artifact.artifactId AS ArtifactId, artifact.version AS Version
    ]]></cypher>
    </constraint>
```

**To disable logback and enable log4j2, uncomment the lines in pom.xml and example-backend/pom.xml**