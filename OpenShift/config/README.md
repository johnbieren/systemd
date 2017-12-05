## Configuring your OpenShift environment to run the Systemd Pipeline

Adding the systemd pipeline to your OpenShift environment is very simple with the help of a few scripts present in this directory. Please note that there is a much more complete and automated setup of the entire environment at http://github.com/CentOS-PaaS-SIG/ci-pipeline/tree/master/dev_setup, this is just the basics specific to the systemd pipeline. The first step is to simply execute the jenkins/systemd-create-openshift-project script. This script will add a new project to your OpenShift tenant with the name systemd. In this tenant, it will create both a Jenkins master and a Jenkins slave. The master will have most of the necessary plugins preinstalled and will be configured properly.

The next step will be to execute create-containers.sh script in this directory. This script creates all the necessary containers for the pipeline run in the systemd OpenShift project. Note that after executing this script, you should monitor the Builds tab in OpenShift to wait for the builds to finish. If any fail, try pressing the Start Build button from the build's page to retry.<br><br>

There are two commands left to run. They both require the oc command to be available. Before running either, you should login to Openshift. To do this, type 
````
oc login -u system:admin
````
into your prompt. Now, you can start by tagging all of the previously created containers with the stable tag. This can be done with
````
oc get is | awk '{print $2}' | grep -v DOCKER | sed 's/.*5000\///g' | xargs -i oc tag {}:latest {}:stable
````
Finally, you must modify the security context constraints to allow privileged container runs. To do this, simply run
````
oc create -f ./systemd-scc.yaml
````
from this directory.<br>

## Jenkins Setup

You will now want to navigate to your Jenkins master, which should be listed as a route in OpenShift. Begin by navigating to Manage Jenkins -> Manage Plugins -> Available. Type GitHub Pull Request Builder into the Filter box in the top right. Select the plugin and choose to install it and restart at the bottom of the page.<br><br>
Next, you will have to add a GitHub account to your master with read/write rights to the repo you plan to monitor (quite likely, systemd/systemd). In order to accomplish this, select Manage Jenkins -> Configure System. Find the section titled "GitHub Pull Request Builder." Here, click the Add button next to credentials and add the account. Make sure to take note of the description.<br>

## Add the pipeline job

The final step is to add the literal pipeline job. To do this, select New Item, fill in the item name, select Pipeline, and click OK. There are three sections to manually fill in here. The first is the GitHub project field in the General tab. Check this, and fill in the Project url with the url to the GitHub repo you would like to trigger pipeline builds from when new PRs are raised.<br><br>
Second, the trigger must be configured. To do this, go to the Build Triggers tab. Select GitHub Pull Request Builder. For the GitHub API credentials, you will want to select the account you previously configured. Click advanced. You are free to setup things like Trigger phrase for retriggering, but what is very important is that you add to the White list, whether that be normal usernames or organizations (different box for this right below the White list one). If no one is white listed, no one will be able to trigger your pipeline, so it is important to add to the white list.  <br><br>
Third is to simply add the pipeline. In the Pipeline tab, select "Pipeline script from SCM" as your definition. The SCM will be Git, with the Repository URL pointed at the location of the github repo which holds your Jenkinsfile. If you were to use this repo, this would be http://github.com/johnbieren/systemd. Next, change the branch from master to whatever branch you are using, if desired. Finally, for Script Path, enter the path in the repo to the Jenkinsfile (example here would be OpenShift/Jenkinsfile). Click save and setup is complete!
