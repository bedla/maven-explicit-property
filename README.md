# Showcase for Maven configuration overriding
This repo is showcase to see how Maven's configuration parameters are evaluated. 
Interesting combinations are: `plugin.configuration.skip` XML element, `-Dmaven.resources.skip` command parameter, default value, values from effective-pom.
Reason for this showcase is to show that command line arguments does not have precedence before explicit XML elements specified inside `pom.xml` configurations.  
  

- we can run Maven cmd with `-Dmaven.resources.skip=true` argument
- we have following `parameter` definition inside `ResourcesMojo`
```java
@Parameter( property = "maven.resources.skip", defaultValue = "false" )
private boolean skip;
```
- case `A` when no `<skip>` element is explicitly specified, resulting XML looks like following snippet
  - even when `effective-pom` is resolved
```xml
<skip default-value="false">${maven.resources.skip}</skip>
```
- case `B` when `<skip>` element is somewhere specified, resulting XML looks like following snippet
```xml
<skip default-value="false">false</skip>
```
- resulting XML is just evaluated
  - using Maven execution plan
- in case `B` there is no placeholder to be evaluated
  - this means that `false String` value from XML is evaluated to `false boolean`
  - there is no place where `${maven.resources.skip}` could be evaluated, because it just does not exists in effective-pom XML
- in case `A` we have placeholder ready to be evaluated
  - if we pass `-Dmaven.resources.skip=` argument to Maven command line, it is evaluated
  - if not, `default-value` is used
- configuration XML is created during building Maven's execution plan
  - it is merged with effective-pom
  - XML elements explicitly that are explicitly stated have higher precedence 
  - `-D` parameters are evaluated later 

## Effective poms
- Case when `<skip>` element is not specified and no `-Dspring-boot.repackage.skip` command parameter is supplied
  - command `mvn help:effective-pom -Dverbose=true -Doutput=out-from-default.xml`
  - see result `effective-poms/out-from-default.xml`
  - `default-value` is used 
- Case when `<skip>` element is not specified with `-Dspring-boot.repackage.skip` command parameter
  - command `mvn -Dspring-boot.repackage.skip=true help:effective-pom -Dverbose=true -Doutput=out-from-d.xml`
  - see result `effective-poms/out-from-d.xml`
  - `default-value` is used
- Case when `<skip>` element is specified with `-Dspring-boot.repackage.skip` command parameter
  - command `mvn -Dspring-boot.repackage.skip=true help:effective-pom -Dverbose=true -Doutput=out-element.xml`
  - see result `effective-poms/out-from-element.xml`
  - value from `<skip>` element is used
