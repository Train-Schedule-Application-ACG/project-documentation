# Source Control Management (SCM)

Objective: As per the client request, the app's main page header should change from "Train Schedule" to "Find your train!”
- Therefore, we are making changes to project file views/index.jade on GitHub.

-----------

#### Install Git
Install Git using below command 
`sudo yum -y install git`

#### Configure the name and email address associated with git:

`git config --global user.name "<YOUR NAME>"`

`git config --global user.email "<YOUR EMAIL>"`

#### Create an SSH key, using the default settings:

`ssh-keygen -t rsa -b 4096`

*Display the contents of our new public key (be sure to copy the key to your clipboard):*

Go to `cat /home/cloud_user/.ssh/id_rsa.pub`

Log in to [GitHub](https://github.com/ "GitHub"), navigate to **Settings**, click on **SSH and GPG keys**, and click on **New SSH Key**. Paste your public key information into this field and then click **Add SSH key**.

#### Create a Fork

Create a fork from [Linux Academy GitHub repository](https://github.com/linuxacademy/cicd-pipeline-train-schedule-git "Linux Academy GitHub repository").

#### Copy the SSH Clone URL

In the terminal, run the following command:

`cd ~/`

`git clone git@github.com:YOURUSERNAME/cicd-pipeline-train-schedule-git.git`

`cd cicd-pipeline-train-schedule-git/`

#### Create a feature branch to contain the change

`git checkout -b myBranch`

#### Change the header text in views/index.jade from “Train Schedule” to “Find your train!”:

`vim views/index.jade`

`:wq `to save and quit.

#### Add the change in views/index.jade to the next commit:

`git add .`

#### Commit the change:

`git commit -m "<YOUR COMMENT>"` to push the commit. 

#### Push the change to the remote SCM repository:

`git config --global push.default simple`

`git push --set-upstream origin myBranch`

#### Create a pull request to merge the feature branch into the master branch:

- On the GitHub fork page, click Branches, locate myBranch under Your branches, and click New pull request.

- In the top-left of the new pull request, change the base fork dropdown menu to your personal fork.

- Click Create pull request

- Merge the pull request

After a few moments, click the Merge pull request button
- Click Confirm merge
-* The change is pushed.*

## Conclusion
We have installed Git, created a fork of a file, edited to fulfill our needs, and have pushed the change. Congratulations on completing this lab!
