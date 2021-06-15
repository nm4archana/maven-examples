Maven Dependency Management


**Introduction to POM**

POM stands for the "Project Object Model".  It describes the project, configures plugins and declares all the dependencies that is necessary to build a project. 

project, modelVersion, groupId, artifactId and version The following is the minimum requirement of the POM
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.example</groupId>
  <artifactId>my-reference-app</artifactId>
  <version>1.0-SNAPSHOT</version>
</project>

The groupId, artifactId and version together forms the projects fully qualified artifact name, i.e <groupId>: <artifactId>: <version>

In the above example, org.example:my-reference-app:1.0-SNAPSHOT forms the projects fully qualified artifact name


**Project Inheritance **

A POM file can inherit certain properties from parent POM file by specifying the parent element. If many projects have similar configuration, these projects can be refactored by pulling all similar configurations and making a parent project.

For a directory structure
my-reference-app
|
|-- my-reference-module
|   `-- pom.xml
|
 `-- pom.xml

where org.example:my-reference-module:1.0-SNAPSHOT inherits from org.example:my-reference-app:1.0-SNAPSHOT, following is the configuration to be specified
<project>
  <parent>
    <groupId>org.example</groupId>
    <artifactId>my-reference-app</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
 
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-reference-module</artifactId>
</project>



**Project Aggregation**

If a group of projects has to be built or processed together, a parent project can be created and the projects can be specified as modules in parent POM. Invoking maven command against a parent project will also build all the modules.

This is achieved by specifying the directories of child POMs in the parent POM modules and setting the packaging value to 'pom' in parent POM.

For example, my-first-module and my-second-module with the following directory structure 
.
|-- my-reference-app
    |`-- pom.xml
    |
    |`-- my-first-module
    |    `-- pom.xml
    |
     `-- my-second-module
         `-- pom.xml

are the two projects specified as modules in parent POM
<project>
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>org.example</groupId>
  <artifactId>my-reference-app</artifactId>
  <version>1.0-SNAPSHOT</version>
 
  <packaging>pom</packaging>
 
  <modules>
    <module>my-first-module</module>
    <module>my-second-module</module>
  </modules>
 
</project>


**Transitive Dependencies**

The dependencies that we specify for our project will be dependent on other libraries. Maven includes this transitive dependencies automatically. 

If multiple versions of same artifact are encountered as dependencies, how does maven resolve the versions of transitive depndencies?

Maven chooses the "nearest definition"/"shortest path". It chooses the dependency that is closest to the project in the tree of dependency. This is called dependency mediation

For example,

The nearest definition for Module A between ModuleD->ModuleC→ModuleA→1.0 and ModuleD->ModuleC→ModuleB→ModuleA→2.0 is 1.0 

A specific version of dependency can also be enforced by explicitly adding a dependency. In the above example, if we want maven to pick version 2.0 for Module A, a dependency for ModuleA can directly be introduced in ModuleD, i.e ModuleD->ModuleA->2.0
ModuleD
  |
  ├── ModuleC  
      │      
      ├── ModuleA 1.0
      │  
      └── ModuleB
         |
         └── ModuleA 2.0


What version of ModuleA will be picked here?


ModuleC  
  │      
  └── ModuleB
  |   |
  |   └── ModuleA 1.0
  |
  └── ModuleE
      |
      └── ModuleA 2.0

A specific version of dependency for same artifacts with multiple versions can be enforced by dependency management. 
Dependency Management 

Dependency management is a way to centralize the information of dependencies among projects. If a set of projects have a common parent, all the information about the dependencies can be specified in the parent POM and we can have simpler references to the artifacts in the child POMs. 

For example, my-first-module has the following dependencies
<dependencies>
  <dependency>
    <groupId>com.google.collections</groupId>
    <artifactId>google-collections</artifactId>
    <version>1.0</version>
  </dependency>
  <dependency>
    <groupId>com.github.seancfoley</groupId>
    <artifactId>ipaddress</artifactId>
    <version>5.0.0</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
  </dependency>
</dependencies>


and my-second-module has 
<dependencies>
  <dependency>
    <groupId>com.google.collections</groupId>
    <artifactId>google-collections</artifactId>
    <version>1.0</version>
  </dependency>
  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.8</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
  </dependency>
</dependencies>


since both the projects have common dependencies, this can be put in the parent POM as 


<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.collections</groupId>
      <artifactId>google-collections</artifactId>
      <version>1.0</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>
  </dependencies>
</dependencyManagement>


The refactored POM files will look as

my-first-module 
<dependencies>
  <dependency>
    <groupId>com.google.collections</groupId>
    <artifactId>google-collections</artifactId>
  </dependency>
  <dependency>
    <groupId>com.github.seancfoley</groupId>
    <artifactId>ipaddress</artifactId>
    <version>5.0.0</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
  </dependency>
</dependencies>


my-second-module
<dependencies>
  <dependency>
    <groupId>com.google.collections</groupId>
    <artifactId>google-collections</artifactId>
  </dependency>
  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.8</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
  </dependency>
</dependencies>

Dependency management is primarily used to control the versions of the artifacts that are used in transitive dependencies. 


**Bill Of Materials (BOM)**

BOM is the central place that is used to define or update the version of project dependencies. If any project has to use BOM, it can be imported in the dependency management section of pom file.

    If the same artifact is defined with different versions in 2 imported BOMs, then the version in the BOM file that was declared first will win
    We can overwrite the artifact's version by explicitly defining the artifact in our project's pom with the desired version

Order of precedence at which maven resolves the version of an artifact :

    The version of the artifact's direct declaration in our project pom
    The version of the artifact in the parent project
    The version in the imported pom, taking into consideration the order of importing files
    dependency mediation
  
**References:**
  https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html
  https://www.baeldung.com/spring-maven-bom