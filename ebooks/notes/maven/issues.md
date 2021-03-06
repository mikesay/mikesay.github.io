## Error installing artifact's metadata: Error installing metadata: Error updating group repository metadata
> Error installing artifact's metadata: Error installing metadata: Error updating group repository metadata in epilog non whitespace content is not allowed but got d \(position:END_TAG seen ...\<\/metadata\>\\nd... @183:2\)

The metadata xml file (.m2/repository/xxxx/plugins/plugins/maven-metadata-local.xml) was corrupted, correct the xml sytanx to fix it.

## Attach additional artifacts to be installed and deployed in maven pom 
Usually one pom represents one maven module, and can only generate one artifact to be installed and deployed. You can use build-helper plugin's "attach-artifact" goal to make a second artifac created from the pom to be installed and deployed when during install and deploy phase.

## Issue caused by putting private repositories in POM file

![maven issue1](../_media/maven_issue1.png)

If this pom file was released outside and neither repository info nor mirror info setted in settings.xm, the maven build of others will fail at parsing POM file "org.eclipse.jetty:jetty-parent:19", because it can't connect the private repository in the project "org.eclipse.jetty:jetty-project:7.5.4.v20111024". So always put private repository into local settings.xml.

## Issue may happen for one pom file to build several artifacts
Sometimes, one maven project (pom.xml) can be made to build out serveral artifacts by using maven-assembly-plugin, or maven-helper-plugin (attach goal), besides the default artifact.

If there were three build process or jobs to build out the three snapshot version of artifacts, one build process or job build and deploy the default artifact to remote repository server, the second build process or job build and install second artifact to local repository(~/.m2/repository/), and the third build process or job will build and deploy the third artifact to remote repository server, and pom code for third build process or job will have dependency on the second artifacts. 
> Don't deploy the second artifact to remote repository server is because its size is big and only be used by third build process and job.

Sometimes, the third build process or job will fail, because it can't find the second depended artifact. The root cause is:

All the artifacts generated by the same pom will share the same pom, so when the third build process or job try to parse the latest snapshot version of second artifact by pom file, it finds out that the latest snapshot version of second artifact come from remote repository server instead of from local repository, because the first build process or job continues building and deploying the defaut artifact and pom file, so it will always try to download the second artifacts remotely, however the second artifact is never deployed by sencond build process or job, so it failed.

**Solution**
The best practice is to create serveral pom files like pom.xml, pom1.xml, pom2.xml in the same maven project, each pom file has different artifact id. However the disadvantage is that you need to update several times for the common build logic.

## Issue caused by sibling dependency
![maven issue2](../_media/maven_issue2.png)

This problem can also occur if you have some child projects that refer to a parent pom and you have not installed from the parent pom directory (run mvn install from the parent directory). One of the child projects may depend on a sibling project and when it goes to read the pom of the sibling, it will fail with the error mentioned in the question unless you have installed from the parent pom directory at least once.

I just ran into this problem when moving a project to a new computer. I was in the habit of running commands from the child project and didn't run install on the parent.

**Solution**
Always try to build submodules from parent folder by using maven's parameter (-pl) like:
```sh
mvn clean install -pl client-eclipse-image
```