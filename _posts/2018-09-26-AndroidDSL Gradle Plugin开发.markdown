---
layout: post
title:  "AndroidDSL Gradel Plugin开发"
date:   2018-09-26 20:43:13 +0800
categories: Android  Groovy Gradle
---

<h3>环境配置<h3>
1. Intellij环境配置


<h3>gradle plugin开发简介</h3>

<h5>Extension</h5>
&emsp;&emsp;由于不同的module在使用自定义的plugin时可能要完成不同的工作，所以需要在module的build.gradle自定义一些参数，使得plugin能够根据不同的参数完成不同的工作。而Extension（以下简称ext）正是起这种作用。    


