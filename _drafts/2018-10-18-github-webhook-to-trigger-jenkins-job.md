### Setup Jenkins GitHub webhook
> GitHub repository integration with Jenkins is made with the **GitHub Plugin** - installed in Jenkins by default.

The **GitHub Plugin** has three major functionalities:
1. Create hyperlinks between your Jenkins projects and GitHub
2. Trigger a job when you **push** to the repository by groking HTTP POSTs from post-receive hook and optionally auto-managing the hook setup
3. Report build status result back to github as [Commit Status](https://github.com/blog/1227-commit-status-api) which is [documented on SO](http://stackoverflow.com/questions/14274293/show-current-state-of-jenkins-build-on-github-repo/26910986#26910986)

#### GitHub hook trigger for GITScm polling
> This feature is in **Build Triggers** section in Jenkins **job configuration**.

It enables builds after [post-receive hooks in your GitHub repositories](http://help.github.com/post-receive-hooks/). For every incoming **PUSH** event this trigger executes **git-plugin** internal polling to check for changes.

To use **GitHub hook trigger for GITScm polling** feature choose the webhook adding option:
* **Manual**: you'll be responsible for registering the Jenkins hook URLs to GitHub
* **Automatic**: Jenkins will automatically add/remove hook URLs to GitHub repo projects based on Jenkins job configuration

#### Manual Mode for adding GitHub webhook
> Choose this option if you don't want Jenkins to add/remove webhooks to your GitHub account

Webhook must be manually registered to GitHub's repository.

* Get webhook URL for your Jenkins server
  - Click the (?) icon (under Manage Jenkins > Configure System > GitHub)
  - check the URL in Jenkins that receives the post-commit POSTs
  - example URL is `$JENKINS_BASE_URL/github-webhook/`
* Once you have the URL, add it as a webhook to the GitHub repository
  - Open the repository URL in browser https://github.com/nodematiclabs/QA-AutomationEngine
  - Click on `Settings` (gear icon)
  - Click on `Webhooks` (left menu)
  - Click `Add webhook` (top right)
  - Paste the URL into `Payload URL`
  - Choose `Which events would you like to trigger this webhook?` (default is Just push event)
  - Click `Add webhook` button

#### Automatic Mode for adding GitHub webhook
> Choose this option if you want Jenkins to manage webhooks (add/remove) for jobs by itself.

Jenkins will automatically add/remove hook URLs to GitHub based on the project configuration in the background. You'll specify GitHub OAuth token so that Jenkins can login as you to do this.

* Create your [personal access token in GitHub](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
  - In the upper-right corner of any page, click your profile photo, then click `Settings`.
  - In the left sidebar, click `Developer settings`
  - In the left sidebar, click `Personal access tokens`
  - Click `Generate new token`
  - Give your token a descriptive name
  - Select the scopes, or permissions, you'd like to grant this token. To use your token to access repositories from the command line, select repo.
  - Click `Generate token`
  - Copy the token to your clipboard
    - For security reasons, after you navigate off the page, you will not be able to see the token again
* Go to the Jenkins global configuration and add GitHub Server Config
  - Name (doesn't matter)
  - API URL (default `https://api.github.com`)
  - Credentials -> Add
    - Domain: Select `Global credentials`
    - Kind: Select `Secret text`
    - Scope: Select `Global (Jenkins, nodes...)`
    - Secret: `PASTE HERE YOUR GITHUB TOKEN`
    - ID: (doesn't matter)
    - Description: (doesn't matter)
  - Select your credentials to GitHub from drop down menu by your `secret id` given in previous step
  - Click `Test Connection`
    - Output i.e.: `Credentials verified for user nodematiclabs, rate limit: 4998`
  - Select `Manage hooks` [x]
  - Apply and Save

<p align="center"> <img src="jenkins-add-github-server.png" alt="Jenkins GitHub Server configuration"> </p>

#### Webhook verification in GitHub
> Check if the webhook is properly registered in GitHub repository.

* Open the repository URL in browser https://github.com/nodematiclabs/QA-AutomationEngine
* Click on `Settings` (gear icon)
* Click on `Webhooks` (left menu)

<p align="center"> <img src="jenkins-add-github-webhook.png" alt="GitHub - list webhooks"> </p>

* From the list of webhooks click on the newly created one

<p align="center"> <img src="jenkins-github-check-webhook.png" alt="GitHub - select webhook"> </p>

* Scroll down to see `Recent Deliveries`

<p align="center"> <img src="jenkins-github-check-webhook-deliveries.png" alt="GitHub - webhook recent deliveries"> </p>

* On the webhook recent delivery check the `Response` (200 is OK) and click on `Redeliver`

<p align="center"> <img src="jenkins-github-check-webhook-redeliver.png" alt="GitHub - webhook redeliver"> </p>

* Watch the Jenkins logs:
```
$ docker logs continuousintegration_jenkins_1 -f
Oct 18, 2018 11:09:40 AM org.jenkinsci.plugins.github.webhook.subscriber.PingGHEventSubscriber onEvent
INFO: PING webhook received from repo <https://github.com/nodematiclabs/QA-AutomationEngine>!
```

> Please note that the example webhook delivery was only **PING** event - it doesn't trigger the Jenkins job to be build.

> Push any change the the `develop` branch (branch must match the job trigger configuration in Jenkins) to trigger building process.

#### **Note!** Firewall and/or Jenkins port exposing
* Jenkins webhook HTTP(s) URL **must be** reachable from the GitHub Server (reachable from the Internet)
* Webhook feature with GitHub Plugin is aware of possible fake post-receive POSTS
  - Upon receiving a POST event, Jenkins will talk to GitHub to ensure the push was actually made

#### **Note!** Jenkins behind a firewall
If Jenkins server is behind the firewall and not directly reachable from the Internet, GitHub Plugin allows to specify an arbitrary endpoint URL as an override in the automatic mode.
The GitHub Plugin works with reverse proxy or other solution so that the POST from GitHub will be routed to the Jenkins.

#### Troubleshooting webhooks
* If the webhook is configured but builds aren't triggered, check the following things:
  - Click the "admin" button on the GitHub repository in question and make sure post-receive hooks are there
  - If webhook is not there, make sure you have proper credential set in the Jenkins system config page
  - Click `Test connection`
* Enable logging for the class names
  - `com.cloudbees.jenkins.GitHubPushTrigger`
  - `org.jenkinsci.plugins.github.webhook.WebhookManager`
  - `com.cloudbees.jenkins.GitHubWebHook`
  - to see the log of Jenkins trying to install a post-receive hook.
* Click "Test hook" button from the GitHub UI and see if Jenkins receive a payload
  - Verify the branch that recent push was made to
  - `command-core` job trigger is configured for the `develop` branch
  - only **PUSH** event to `develop` branch will trigger the job, even that GitHub sends webhook for push to any branch
* Watch the Jenkins server logs
  - `docker logs [JENKINS DOCKER CONTAINER NAME] -f`