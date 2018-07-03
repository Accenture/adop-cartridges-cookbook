---
layout: docs
title: SCM
permalink: /docs/recipes/adding-a-pluggable-scm/
---

This section describes how to add a Pluggable SCM to your current cartridge.
It is done by using a custom groovy package _pluggable.scm.*_  to implement a chosen SCM provider based on Java reflection with the help of _SCMProvider_ class.

# How to add pluggable to your project 
Pluggable SCM allows to use below SCM providers but it can be extended

* Gerrit
* BitBucket

Using it requires an additional package and class

```   	
 import pluggable.scm.*;
 SCMProvider scmProvider = SCMProviderHandler.getScmProvider("${SCM_PROVIDER_ID}", binding.variables)
```

Add those at the very beginning of your _../jenkins/jobs/dsl/your-file-name.groovy_ file

As your chosen SCM will be used, no more gerrit project is needed, so this can be removed

```   	
 def referenceAppGitUrl = "ssh://jenkins@gerrit:29418/${PROJECT_NAME}/" + referenceAppgitRepo
```

Once you have done the above, you need to edit the job

## Using pluggable in Jenkins jobs declarations
To be able to use Pluggable SCM, it is needed to replace 

```   	
 scm {
     git {
         remote {
             url(referenceAppGitUrl)
             credentials("adop-jenkins-master")
         }
         branch("*/master")
     }
 }
```
	
with

```   	
 scm scmProvider.get(projectFolderName, referenceAppGitRepo, "master", "adop-jenkins-master", null)
```

where :

* **projectFolderName** - name of your project
* **referenceAppGitRepo** - name of the repository you want to clone i.e. _spring-petclinic_
* **master** - branch to be used
* **adop-jenkins-master** - the id of the Jenkins credentials to be used in the job in order to clone from SCM
* **null** - extras to be used, for example _{extensions{cleanAfterCheckout()}}_. More extensions can be found here - [Jenkins job DSL - extensions](https://jenkinsci.github.io/job-dsl-plugin/#method/javaposse.jobdsl.dsl.helpers.scm.GitContext.extensions)

_scmProvider.get()_ basically behaves similarly to default scm function (the same you are replacing) but instead of using _referenceAppGitUrl_, it constructs the URL automatically from provided: 

* SCM host
* SCM protocol
* SCM clone user (in case SSH protocol is used)
* SCM port
* project name
* repository name

Also _extensions_ can be included.

To be able to perform _triggering_ - automatically pull changes from repo as soon as they have been pushed - it is needed to replace 

```   	
 triggers {		  
     gerrit {		
         events {		
             refUpdated()		
         }		
         project(projectFolderName + '/' + referenceAppGitUrl, 'plain:master')		
         configure { node ->		
             node / serverName("ADOP Gerrit")		
         }		
     }		
 }
```

with

```   	
 triggers scmProvider.trigger(projectFolderName, referenceAppGitRepo, "master")
```

where :

* **projectFolderName** - name of your project
* **referenceAppGitRepo** - name of the repository you want to clone i.e. _spring-petclinic_
* **master** - the branch to be used.

_scmProvider.trigger()_ acts as the default scm function which you are replacing.

**Note:** Those changes require the _Load_Cartridge_ job to be updated, more details can be found [here](TODO:URL).

## Using pluggable SCM with multi-branch pipelines
Pluggable SCM can also be used for multi-branch pipelines in a similar way by replacing

```   	
branchSources {
        git {
            remote(referenceAppGitUrl)
            credentialsId("adop-jenkins-master")
        }
    }
```

with

```   	
branchSources scmProvider.getMultibranch(projectFolderName, referenceAppGitRepo, "adop-jenkins-master", null)

```
where :

* **projectFolderName** - name of your project
* **referenceAppGitRepo** - name of the repository you want to clone i.e. _spring-petclinic_
* **adop-jenkins-master** - the id of the Jenkins credentials to be used in the job in order to clone from SCM
* **null** - extras to be used, for example _{ignoreOnPushNotifications()}_. More extensions can be found for specific SCM providers here - [Jenkins job DSL - plugin](https://jenkinsci.github.io/job-dsl-plugin/#path/multibranchPipelineJob-branchSources)


# Example
```
import pluggable.scm.*;
SCMProvider scmProvider = SCMProviderHandler.getScmProvider("${SCM_PROVIDER_ID}", binding.variables)

def projectFolderName = "${PROJECT_NAME}"
def buildAppJob = freeStyleJob(projectFolderName + "/<JOB_NAME>")
def multibranchPipelineJob = multibranchPipelineJob(projectFolderName + "<MULTI_BRANBCH_JOB_NAME>")

def referenceAppgitRepo = "spring-petclinic"

buildAppJob.with {
    description("This job builds Java Spring reference application")
    wrappers {
        preBuildCleanup()
        injectPasswords()
        maskPasswords()
        sshAgent("adop-jenkins-master")
    }
    scm scmProvider.get(projectFolderName, referenceAppGitRepo, "master", "adop-jenkins-master", null)
    ...
    triggers scmProvider.trigger(projectFolderName, referenceAppGitRepo, "master")
    ...
}

multibranchPipelineJob.with{

   branchSources scmProvider.getMultibranch(projectFolderName, referenceAppGitRepo, "adop-jenkins-master", null)

   triggers {
      periodic(1)
    }
	
    orphanedItemStrategy {
        discardOldItems {
            numToKeep(20)
        }
    }
 }
```


---

More details here:

- [About Pluggable SCM](https://accenture.github.io/adop-pluggable-scm/docs/recipes/about-pluggable-scm/)
- [Cloning from Gerrit](https://accenture.github.io/adop-cartridges-cookbook/docs/recipes/git-clone/)
- [Creating a Jenkins job](https://accenture.github.io/adop-cartridges-cookbook/docs/recipes/creating-a-freestyle-job/)
