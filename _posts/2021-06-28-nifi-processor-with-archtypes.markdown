---
layout: post
title:  "Creating Custom NiFI Processors with Maven Archetypes"
date:  2021-06-28 18:33:28 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["#BE", "#DataEngineering", "#Java"]
---

This post assumes you're familiar with Apache NiFi and its concepts and architecture.
Currently, Apache NiFi comes with almost 300 built in processors and more than 50 controller services.
But in case you need to implement a processor with a propriety, or an industry specific logic,
NiFi provides a convenient scaffolding capabilities via Maven Archetypes.

### Apache NiFi extensions
Apache NiFi extensions are packed in NAR files (NiFi archives).
In a very high level, NAR files allow multiple components and dependencies to be packaged into a single package.
A more details explanation about NARs can be found [here](https://nifi.apache.org/docs/nifi-docs/html/developer-guide.html#nars){:target="_blank"}.

### Structure of a processor project
Processor projects usually organized in bundle which composed of:

* A Maven project which generates JAR of processors
* A Maven project responsible for packaging the processors into a NAR.
* A pom for the entire bundle that generates both of the projects above.

### Processor Archetype
NiFi supplies a Maven Archetype to easily generate the bundle structure and a lot of boilerplate code,
allowing you to start writing your processor code without having to worry about the boilerplate.

For the step-by-step guide I'll be using Intellij Idea, but feel free to use an IDE of your choice, or do it directly via CLI.


#### Step 1: Create an empty project
* Go to <i>File -> New -> Empty Project</i>.
* Make sure to define an SDK via <i> File -> Project Structure </i>

#### Step 2: Generate an Archetype
{% highlight shell %}
mvn archetype:generate -DarchetypeGroupId=org.apache.nifi -DarchetypeArtifactId=nifi-processor-bundle-archetype -DarchetypeVersion=1.11.4 -DnifiVersion=1.11.4
{% endhighlight %}

You can access Maven CLI via a double click on Ctrl.



#### Step 3: Fill in properties
After running the generation command, you will be prompted to fill in the bundle properties,

![Maven Prompt](/assets/post-images/2021-06-28-nifi-processor/2021-06-28-nifi-processor.JPG)

Let's assume your bundle name is <i>customprocessor</i>, then I'll suggest filling the values as follows:

* <b>artifactBaseName</b> - Artifact base name, should be <i>customprocessor</i>
* <b>gropuId</b> - Maven groupId for the bundle
* <b>version</b> - Maven version for the bundle, depends on how you choose to version your project.
* <b>artifactId</b> - <i>nifi-customprocessor-bundle</i>
* <b>package</b> - The Java package for the processor

#### Project Structure
As described above, the output will be 2 projects and a POM file, and look like this:

![Project Output](/assets/post-images/2021-06-28-nifi-processor/output.JPG)

#### Boilerplate Code
If you'll inspect the processor class that was generated for you in the processor project, you'll notice a lot of boilerplate code provided for you,
so all you have to do is start implementing your propriety business logic.

![Project Output](/assets/post-images/2021-06-28-nifi-processor/code-boilerplate.JPG)

### Build and Deploy
Next, run `mvn clean install` and copy the NAR file to NiFi `/lib` directory.
After this, restart NiFi and your processors should be available.