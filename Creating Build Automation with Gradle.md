# Creating Build Automation with Gradle

Build Automation plays an important part in continuous integration. With build automation we can build necessary components of automated pipelines for the project. 

In this project, we're using Gradle as the build automation tool.

# The Scenario
Developers have created the first prototype of a train scheduling app. They need us to create a Gradle build for it. The developers need this build to execute some automated tests that they have written. If the automated tests pass, the build needs to produce a packaged archive that includes the application code and its dependencies. The completed archive should be located in the project folder as dist/trainSchedule.zip.

In short, this automation needs to:

- Have a build task that can be called to execute the entire build process
- Execute some automated tests (where any testing failures cause the build to fail)
- Package the code and its dependencies into a zip archive called `dist/trainSchedule.zip`

### A List of Steps
In order to accomplish this, you will need to perform several steps. We'll number them here, and follow this outline through the lab.

- Configure git for ssh authentication with GitHub.com
- Create a personal fork of the sample repository https://github.com/linuxacademy/cicd-pipeline-train-schedule-gradle
- Clone your personal fork from GitHub
- Initialize the project with Gradle
- Create a Gradle build task
- Include the id("com.github.node-gradle.node") plugin in the Gradle project
- Execute automated tests as part of the build with the npm_test task
- Create a task to generate a zip archive (dist/trainSchedule.zip) of the files that need to be depoyed to production
- Make sure task dependencies are set up so tasks execute in the correct order
- Commit and push these changes to your GitHub fork

## Configure git for ssh Authentication with GitHub.com

` git config --global user.name "Our Name"`
` git config --global user.email "ourname@ourdomain.com"`
` ssh-keygen -t rsa -b 4096`
`cat /home/cloud_user/.ssh/id_rsa.pub`

## Fork of the Sample Repository
Linux Academy GitHub repo at https://github.com/linuxacademy/cicd-pipeline-train-schedule-gradle. 

## Clone Our Personal Fork from GitHub
Copy the SSH Clone URL

`cd ~/`
`git clone git@github.com:kriteja/cicd-pipeline-train-schedule-gradle.git`

## Initialize the Project with Gradle
Once we have the Git repository cloned to our home directory, let's cd into the directory and initialize the Gradle build. We'll check afterward with an ls to see if we've got all the source code:

`cd ~/cicd-pipeline-train-schedule-gradle`
`./gradlew init`
`ls -la`

## Create a Gradle Build
To create a Gradle build, we have to install Gradle if this is the first time installation. 

However, as there's a Gradle wrapper included in the repository installing Gradle won't be necessary everytime. Gradle wrapper is nothing but a collection of scripts that will install Gradle all by itself. As long as we have Java 7 or later installed, we're good to go. 

There's a file in this repository called gradlew, and that is what we'll use to install Gradle and run gradle commands. To get that process moving, let's build Gradle by executing that script:

To install Gradle run, `./gradlew build` 

Once Gradle is installed, initialize the Gradle build by `./gradlew init`

Check the files by `ls -la` and you'll see a build.gradle file. 

Since this project is built on node.js, we're installing a supporting Plugin for node.js builds.

```bash
plugins {
  id("com.github.node-gradle.node") version "2.2.4"
}

// Confgure the plugin so that the system downloads and installs node.js and npm locally but not system-wide. 

directory.
node {
  version    = '9.11.2'
  download = true
}

// By calling the task named "Build task" it will run any and all other tasks that are required to actually call the build "built."

// All we need to do, in order for this build to actually run, is to call some of the tasks
// built in to the node.js and npm plugins. We're going to call them here.
build.dependsOn npm_build
```
**/////////////////////////////////////////////////////**

Without comments:

plugins {
  id("com.github.node-gradle.node") version "2.2.4"
}

node {
  version    = '9.11.2'
  download = true
}

task build

build.dependsOn npm_build
Save that file out, and we can build again.

Create the Gradle Build Again
We're going to build again, but this time including all of the things we just put in gradle.build. Gradle won't be downloaded again, because we got it the first time we ran build. Run the same command as before:

[cloud_user@$host]$ ./gradlew build
This time we'll see the node plugin getting downloaded though. We should land back at a command prompt, after a successful build.

Now let's edit build.gradle again. We've got to tack something on to the end of the file.

...
npm_build.dependsOn npmInstall
This will ensure that we run npmInstall. It ensures that we download any shared library dependencies that this particular node.js application requires.

Just like our we set our build to depend on npm_build in the line before this new one, we're going to make our npm.build depend on npmInstall. If npmInstall fails, then our npm.build will fail, and that will cause our build to fail.

Now let's run the build again:

[cloud_user@$host]$ ./gradlew build
Notice this time that there's no node getting downloaded, since it happened during our last build, but this time we're downloading npmInstall. This may take some time (several minutes) so be patient.

If we run the same build again, we'll see that it's finished in just a couple of seconds. Gradle is very good at detecting when build steps don't need to be run again, and doesn't perform them when it doesn't need to.

Execute Automated Tests as Part of the Build with the npm_test Task
We want to ensure that our build runs our automated tests. These are simple things, like sanity tests that confirm whether or not the application will start up and respond to requests. We're going to add another couple of lines at the bottom of our build.gradle file to do this:

...
npm_build.dependsOn npm_test
npm_test.dependsOn npmInstall
The first sets up those tests. The second though makes sure that npm_test depends on npmInstall. Since Gradle is capable of running tasks in parallel (two or more tasks at the same time), we need to set up these dependencies to run in a specific order. We want to make sure that npmInstall is finished before npm_test starts up, and we want to make sure npm_test is done before npm_build happens. Now if we run the build again, we'll get some output related to the testing. We should see that GET / and GET /trains both run successfully, showing us a 2 passing (XXXms) line near the end of the build command's output.

The way things work now, the automated tests get run as part of the build. If we (or developers) make a code change that causes one of these tests to fail, the build itself will fail. And we'll get immediate feedback, letting us know that we broke something.

Create a Task to Generate a zip Archive of the Files that Need to be Deployed to Production
We need to create a single zip file containing all of the source code. Deploying the code will then a simple matter of unzipping the archive. The Gradle build should generate an archive of the application in dist/trainSchedule.zip.

Gradle comes with some built-in functionality to facilitate this. We've just got to add a task that utilizes it in order to create this archive. Between task build and build.dependsOn npm_build, we're going to write this zipping task: With Comments

// Create a task that is the Zip type
    task zip(type: Zip) {
// Find the files at . (dot) which means "right here in the root of the directory"
              from ('.') {
// Grab all of the files in the root directory, not the files AND subdirectories
// with their files, just the files sitting right in the root directory.
      		    include "*"
// These next few lines have double asterisks, which means EVERYTHING. For each line, we want to grab the
// directory that we've named, and everything (files and subdirectories) below it in the file tree.
      		    include "bin/**"
      		    include "data/**"
      		    include "node_modules/**"
      		    include "public/**"
      		    include "routes/**"
      		    include "views/**"
      	    }
// This is the destination where the zip file is going to land. We're putting it in a directory names dist,
// which is going to be a subdirectory of the root directory.
      	    destinationDir(file("dist"))
// We're naming the zip file itself. When we're done, there should be a trainSchedule.zip sitting in the ./dist
// directory.
      	    baseName "trainSchedule"
          }

// This will make the build task call on the zip task. If zip doesn't finish, the build fails.
          build.dependsOn zip
// Here, we want to make sure our zip task runs after npm_build    
          zip.dependsOn npm_build
Without comments:

task zip(type: Zip) {
            from ('.') {
                include "*"
                include "bin/**"
                include "data/**"
                include "node_modules/**"
                include "public/**"
                include "routes/**"
                include "views/**"
            }
            destinationDir(file("dist"))
            baseName "trainSchedule"
        }

        build.dependsOn zip
        zip.dependsOn npm_build
With those additions made, let's execute the Gradle build again to generate the archive:

[cloud_user@$host]$ ./gradlew build
Oh look! The tests passed and we can see at the end of the output that our zip got built. Let's be sure though. Run an ls, and we should see a dist directory now that wasn't there before. If we do an ls on that dist directory, we should see a trainSchedule.zip file sitting in there.

Commit and Push These Changes to Our GitHub Fork
The first step in this process is to see what's actually going to get pushed up to GitHub when we commit. To do that, we'll run git status. We're going to get a list of "Untracked files" that include the dist directory. We want to avoid committing the archive, so we're going to edit .gitignore and add dist to it. The file ought to look like this when we're done:

node_modules
.gradle
dist
Now when we build, Git will ignore that dist directory whenever we run a build. If we run another git status, we won't see dist listed in the "Untracked files" output.

Now, to add all of our changes to our fork, run a git add ., and commit it with:

[cloud_user@$host]$ git commit -m "add gradle build"
Everything is ready, now we can push our fork back up to Git with git push.

Conclusion
That's it. We've complete the task set before us, which was to set up build automation in Gradle. We created a fork of a GitHub project, created a Gradle build that included some automated tests, and pushed it back up to GitHub. Congratulations. We're done.

Addendum
Here's the full build.gradle file, minus the comments:

plugins {
  id("com.github.node-gradle.node") version "2.2.4"
}

node {
  version    = '9.11.2'
  download = true
}

task build

    task zip(type: Zip) {
        from ('.') {
		    include "*"
		    include "bin/**"
		    include "data/**"
		    include "node_modules/**"
		    include "public/**"
		    include "routes/**"
		    include "views/**"
	    }
	    destinationDir(file("dist"))
	    baseName "trainSchedule"
    }
    build.dependsOn zip
    zip.dependsOn npm_build

build.dependsOn npm_build
npm_build.dependsOn npmInstall
npm_build.dependsOn npm_test
npm_test.dependsOn npmInstall
