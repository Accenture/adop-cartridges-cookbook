---
layout: docs
title: SCM - Pipelines
permalink: /docs/recipes/adding-a-pluggable-scm-for-pipelines/
---

# Using pluggable SCM with Jenkins pipelines

In order to consume the library in your Jenkins pipeline, ensure you have first followed [these instructions](https://jenkins.io/doc/book/pipeline/shared-libraries/#global-shared-libraries) to install the library in your Jenkins instance.

At the top of your pipeline script/Jenkinsfile, you can then import the library as such:
```
@Library('adop-pluggable-scm-jenkinsfile') _
```
Instead of annotating with import statements, the symbol _ is annotated, according to the [annotation pattern](https://jenkins.io/doc/book/pipeline/shared-libraries/#loading-libraries-dynamically).

To consume the scm object, insert the following method call in your Pipeline script:
```
checkout scmGet(scmUrl, projectName, repoName, credentialsId, branch, additionalMapExtensions, scmClass)
```
where

  * scmUrl is the base URL of the SCM provider you want to use with a trailing slash e.g. https://github.com/
  * projectName is the name of the project/namespace in your SCM provider
  * repoName is the name of the repository to checkout
  * credentialsId is the id of the credential in the Jenkins credential manager to use
  * branch is the name of the branch to checkout on ref updates, defaults to master
  * additionalMapExtensions is an additional arrayList of scm extensions which will be appended to the returned scm object, defaults to empty
  * scmClass is the name of the parent SCM class to implement, defaults to Git