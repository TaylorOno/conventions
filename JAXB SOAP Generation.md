---
title: JAXB SOAP Generation
created: '2019-03-23T15:23:14.423Z'
modified: '2019-03-23T16:26:11.678Z'
tags: [Gradle, WebServices]
---

# JAXB SOAP Generation

There are several types libraries and methods used for XML bindings and generated the code required for marshalling/unmarshalling.  Below are some samples processes that we have converted from maven to gradle.

## Generate Code from WSDL using JAXB 
__Config Setup__
```groovy
configurations {
    jaxb
}
```
__Required Dependencies__
```groovy
jaxb group: 'org.glassfish.jaxb', name: 'jaxb-xjc', version: '2.2.11'
```
__Command__  
```groovy
task genJaxb {
    ext.sourcesDir = "${buildDir}/generated-sources/jaxb"
    ext.classesDir = "${buildDir}/classes/jaxb"
    ext.schema = "src/main/resources/YOUR_XSD.xsd"

    outputs.dir classesDir

    doLast() {
        project.ant {
            taskdef name: "xjc", classname: "com.sun.tools.xjc.XJCTask",
                    classpath: configurations.jaxb.asPath
            mkdir(dir: sourcesDir)
            mkdir(dir: classesDir)

            xjc(destdir: sourcesDir, schema: schema) {
                arg(value: "-wsdl")
                produces(dir: sourcesDir, includes: "**/*.java")
            }

            javac(destdir: classesDir, source: 1.8, target: 1.8, debug: true,
                    debugLevel: "lines,vars,source",
                    classpath: configurations.jaxb.asPath) {
                src(path: sourcesDir)
                include(name: "**/*.java")
                include(name: "*.java")
            }

            copy(todir: classesDir) {
                fileset(dir: sourcesDir, erroronmissingdir: false) {
                    exclude(name: "**/*.java")
                }
            }
        }
    }
}
```

## Generate Code from xsd using JiBX  
__Config Setup__  
```groovy
configurations {
    axisGenAntTask
}
```  
__Required Dependencies__  
```groovy
axisGenAntTask group: 'org.apache.axis2', name: 'axis2', version: '1.6.3'
axisGenAntTask group: 'org.apache.axis2', name: 'axis2-tools', version: '1.1.1'
axisGenAntTask group: 'org.apache.neethi', name: 'neethi', version: '2.0.2'
axisGenAntTask group: 'org.apache.ws.commons.schema', name: 'XmlSchema', version: '1.4.7'
axisGenAntTask group: 'org.apache.ws.commons.axiom', name: 'axiom-api', version: '1.2.14'
axisGenAntTask group: 'org.apache.ws.commons.axiom', name: 'axiom-impl', version: '1.2.14'
compile group: 'org.apache.axis2', name: 'axis2-jibx', version: '1.6.3'
axisGenAntTask group: 'org.apache.axis2', name: 'axis2-jibx', version: '1.6.3'
axisGenAntTask group: 'org.jibx', name: 'jibx-run', version: '1.3.1'
axisGenAntTask group: 'org.jibx', name: 'jibx-tools', version: '1.3.1'
axisGenAntTask group: 'org.jibx', name: 'jibx-bind', version: '1.3.1'
axisGenAntTask group: 'org.jibx', name: 'jibx-extras', version: '1.3.1'
```  
__Command__
```groovy
task genJiBX {
    group 'WebServices'
    doFirst(){
        ant.java(classname: "org.jibx.schema.codegen.CodeGen", fork: true, classpath: "${configurations.axisGenAntTask.asPath}") {
            arg(line: "-b 'src/main/resources/binding.xml'")
            arg(line: "-c 'src/main/resources/jibx/customize-jibx.xml'")
            arg(line: "-t 'build/generated-sources'")
            arg(line: "src/main/resources/jibx/xsd/*.xsd")
        }
    }
}
```

## Generate Code from WSDL using Axis+JiBX  
__Config Setup__  
```groovy
configurations {
    axisGenAntTask
}
```
__Required Dependencies__  
```groovy
axisGenAntTask group: 'org.apache.axis2', name: 'axis2', version: '1.6.3'
axisGenAntTask group: 'org.apache.axis2', name: 'axis2-tools', version: '1.1.1'
axisGenAntTask group: 'org.apache.neethi', name: 'neethi', version: '2.0.2'
axisGenAntTask group: 'org.apache.ws.commons.schema', name: 'XmlSchema', version: '1.4.7'
axisGenAntTask group: 'org.apache.ws.commons.axiom', name: 'axiom-api', version: '1.2.14'
axisGenAntTask group: 'org.apache.ws.commons.axiom', name: 'axiom-impl', version: '1.2.14'
compile group: 'org.apache.axis2', name: 'axis2-jibx', version: '1.6.3'
axisGenAntTask group: 'org.apache.axis2', name: 'axis2-jibx', version: '1.6.3'
axisGenAntTask group: 'org.jibx', name: 'jibx-run', version: '1.3.1'
axisGenAntTask group: 'org.jibx', name: 'jibx-tools', version: '1.3.1'
axisGenAntTask group: 'org.jibx', name: 'jibx-bind', version: '1.3.1'
axisGenAntTask group: 'org.jibx', name: 'jibx-extras', version: '1.3.1'
```
__Command__
```groovy
task WSDL2Code {
    group 'WebServices'
    doFirst(){
        ant.echo(message:"Generating Classes for use with WSDL /main/webapp/WEB-INF/services/ARSService/META-INF/ARSService.wsdl")
        ant.java(classname: "org.apache.axis2.wsdl.WSDL2Java", fork: true, classpath: configurations.axisGenAntTask.asPath) {
            arg(line: "-uri 'src/main/webapp/WEB-INF/services/ARSService/META-INF/ARSService.wsdl'")
            arg(line: "-o '${buildDir}/generated-sources’")
            arg(line: "-S ")
            arg(line: "-d 'jibx’")
            arg(line: "-s")
            arg(line: "-or")
            arg(line: "-ss")
            arg(line: "-sd")
            arg(line: "-ssi")
            arg(line: "-g")
            arg(line: "-p 'com.corelogic.platform.services.corears.srvimpl’")
            arg(line: "-Ebindingfile 'src/main/resources/binding.xml’")
        }
    }
}
```
