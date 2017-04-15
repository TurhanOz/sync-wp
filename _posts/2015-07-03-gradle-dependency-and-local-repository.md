---
ID: 278
post_title: Gradle, dependency and Local Repository
author: turhan
post_date: 2015-07-03 18:13:54
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/gradle-dependency-and-local-repository/
published: true
dsq_thread_id:
  - "3902023826"
---
<p style="text-align: justify;">If you are an android developer, working on non collaborative projects on your own computer, you have definitely faced the situation where you need to use some piece of software among many projects (mainly utils).</p>
<p style="text-align: justify;">This situation is nothing else than a dependency requirement. So what are our choices to make it smooth ?</p>

<h2>step 1 : create an Android library module</h2>
<p style="text-align: justify;">I am sure many of you are making copy/paste of piece of sofware from one project to another. Who didn't, right ?
First of all, get rid of this archaic habit and simply put this reusable software into an <a href="https://developer.android.com/tools/projects/index.html#LibraryModules">android Library module</a> . That's easy and cheap. Moreover, that gives you the power to make your software evolve peacefully, with adding quality and consistency through time.</p>

<h2>step 2 (optional) : make it public and deploy it on maven central</h2>
<p style="text-align: justify;">This module is hosted on your computer. You can make the choice to publish your library into github and make the source available to the world. Then, you can simply deploy your artifact to maven central and easily fetch it into any of your project (and let others use it too). To do so, please read <a href="http://turhanoz.com/reactive-directory-chooser-an-open-source-journey/">my previous article</a> where I detailed the entire process of publishing and deploying an open source project.</p>
<p style="text-align: justify;">But you probably won't go that far. Your need is to keep your software private but benefit from the gradle dependency mechanism.</p>

<h2>step 2 : keep it private</h2>
<p style="text-align: justify;">You will probably keep this peace of software private for any valid reason (high end technology you haven't patented yet oO ... or ... software not high quality enough ...or... pure selfishness...) Whatever the reason is, it's ok, it's your decision :)
However, even if it is private, do not forget to version your source code into private repository.</p>

<h2>step 3 : publish on local directory</h2>
<p style="text-align: justify;">The idea here is to publish your artifact into a local directory of your file system. Bear in mind that this directory (used as repository) will host many versions of the same library, and even multiple private libraries, so don't forget to backup it (using dropbox or google drive for instance) in case you switch computer.</p>
<p style="text-align: justify;">Based on the <a href="https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.RepositoryHandler.html">Gradle Repository Handler documentation</a>, gradle supports many repositories where your project can fetch artifacts. Among them, you can find jCenter(), mavenCentral(), mavenLocal() or any maven() custom repository.</p>
<p style="text-align: justify;"><em>Why don't we use directly mavenLocal() as our local repository ?</em>
We could of course install our artifact into mavenLocal (/home/.m2/repository) but this repository has another goal. It's mainly used for caching artifacts to save network requests. Moreover, we loose our sync-ing requirement. Indeed, there is no need to backup the the .m2 folder, it's just non sense.</p>
<p style="text-align: justify;">So we need to choose a local folder of our choice (for instance /Users/turhan/LocalRepository/).</p>

<h3>step 3.1 : configure your library's build.gradle</h3>
We have to configure our library's build.gradle to specify the directory where to deploy our artifact
<pre class="lang:default mark:1,3,4,5,10 decode:true ">apply plugin: 'maven'

group = "com.yourgroupid"
archivesBaseName = "your-library-name"
version = "1.0.0"  //customize this for new release

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: 'file://' + new File(System.getProperty('user.home'),'LocalRepository').absolutePath) { }

            snapshotRepository(url: 'file://' + new File(System.getProperty('user.home'),'LocalRepository/snapshots').absolutePath) { }

            pom.project {
                name 'Your Library Name'
                packaging 'aar'
                description 'My very sensitive Library'
                url 'http://private-git-url'

                //optional
                scm {
                    connection 'scm:pathtoyour.git'
                    developerConnection 'scm:pathtoyour.git'
                    url 'url of your project'
                }

                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }

                developers {
                    developer {
                        id 'YourId'
                        name 'yourName'
                        email 'your@email.com'
                    }
                }
            }
        }
    }
}</pre>
<p style="text-align: justify;">We have here created the <em>uploadArchives</em> task by defining the repository url where the artifact will be deployed by maven(here it is a local <em>file://</em> path).
As you can see, we have to include the maven plugin  (highlighted on the first line) so the mavenDeployer can work.
Finally, don't forget to specify a group, an archiveBaseName as well as a version. And for each release, do not forget to increment the version  (better put it into a variable inside grade).</p>

<h3>step 3.2 : upload your artifact</h3>
So now, to deploy our artifact into our local directory called LocalRepository, we simply call the task
<pre class="lang:default decode:true ">$ ./gradlew uploadArchives</pre>
<h2>step 4 : fetch your artifact into your project</h2>
<h3>step 4.1 : configure the repository url</h3>
On any other other project of yours, open the top build.gradle. We will specify there the path to our local repository
<pre class="lang:default decode:true">def localMavenRepository = 'file://' + new File(System.getProperty('user.home'), 'LocalRepository').absolutePath

allprojects {
    repositories {
        mavenCentral()
        maven {url localMavenRepository}
    }
}</pre>
<h3>step 4.2 : import the dependency</h3>
Now, we can easily define our library as a dependency, such any other dependency. Gradle will look into each repositories we have defined (mavenCentral() and or custom local repository)
<pre class="lang:default mark:4 decode:true">dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.2.0'
    compile 'com.yourgroupid:your-library-name:1.0.0@aar'
}</pre>
<h2>Conclusion</h2>
<p style="text-align: justify;">We could have installed a Nexus repository locally, but to be honest with you, this is overkill (credentials + web server &amp; so) for our need (non collaborative projects). We need something that could be setup quickly, something that could be backuped easily.
The setup detailed here is not complicated at all and gives you the opportunity to manage your private dependencies easily.
I hope you could save some time with that and make your development process more smooth.</p>
<p style="text-align: justify;">Enjoy :)</p>
&nbsp;