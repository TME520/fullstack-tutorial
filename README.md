# Apollo full-stack GraphQL tech challenge

This is the documentation for the Apollo full-stack GraphQL tech challenge.

## Build and deployment instructions

Here are the instructions to follow if you want to deploy a stack for the Apollo full-stack GraphQL tech challenge in AWS.
You are likely to find the process cumbersome, I don't think we will be friends after you went through that little ordeal.
Anyway, here we go.

### Preparation: get your launchpad ready

1. You will need a Hosted Zone in AWS Route 53: create one that suits your needs and budget. I got *schneiderzupper.de* for 9$AUD a year; not fancy, but cheap,
2. Clone [this repo](https://github.com/TME520/fullstack-tutorial.git),
3. Setup your own [Jenkins](https://www.jenkins.io/),
4. Create a Jenkins pipeline (type: Pipeline) named `orchestrated`,
5. In the Jenkins home directory (usually /var/lib/jenkins), locate its `config.xml` file (should be `/var/lib/jenkins/jobs/orchestrated/config.xml`) and [update it with this content](https://github.com/TME520/fullstack-tutorial/blob/master/jenkins/config.xml),
6. Don't forget to select the proper credentials so Jenkins can access your Git repository if required,
7. In Jenkins, configure the AWS credentials: `aws_access_key` (username with password) - Store: Jenkins, Domain: global,
8. Check your Jenkins plugins list:

```
Ant Plugin, Authorize Project, Build Name and Description Setter, Build Timeout, Command Agent Launcher Plugin, Config File Provider Plugin, Configuration as Code, Copy Artifact, Dashboard View, Docker Pipeline, Email Extension, Embeddable Build Status Plugin, Git Parameter, GitHub Branch Source, Gradle Plugin, LDAP Plugin, Oracle Java SE Development Kit Installer Plugin, OWASP Markup Formatter Plugin, PAM Authentication plugin, Pipeline, Pipeline: GitHub Groovy Libraries, Publish Over SSH, Rebuilder, Role-based Authorization Strategy, SSH Agent Plugin, SSH Build Agents plugin, SSH Credentials Plugin, SSH plugin, Throttle Concurrent Builds, Timestamper, Workspace Cleanup Plugin
```

### Deployment

1. Run Jenkins pipeline `orchestrated`: give it the name you want (no forbidden characters though), leave the *authorized_ip* field empty if you want your current external IP to be used, set the hosted zone to the one defined in step 1 and click on `Build`,
2. Open the [AWS console](https://console.aws.amazon.com), go to CloudFormation | Stacks | click on your stack,
3. In the Outputs tab, refer to the *SSHConnectionShellCommand* field in order to get the command to type in a terminal when you want to SSH into the EC2 instance that has just been deployed.

### Troubleshooting

1. In case of trouble, SSH into the EC2 instance and look into `/var/log/cfn-init-cmd.log` and `/var/log/cloud-init-output.log`,
2. You can also run `netstat -atnup | grep 4000` as root in order to check if port 4000 is being listened to.

### Missing elements

- I went through 70% of the [tutorial](https://www.apollographql.com/docs/tutorial/introduction/). I was able to get the server and the client running, and to run queries from Apollo Studio.
- I didn't go all the way through for 2 reasons: first, I learned enough at that point, second, I found the way modifications to bring to code files were explained to be unclear at times, making it tedious to follow the tutorial. That being said, I'm glad such a tutorial existed and found it informative overall. I gave me enough knowledge to understand what I had to do.
- I have been successful in setting up and starting both server and client, and calling `http://localhost:4000/dev/graphql` locally from the EC2 instance itself shows that things are in working order. I was surprised to find calling `http://mirmidon.schneiderzupper.de.:4000/dev/graphql` from my web browser doesn't work, despite Node listening on 0.0.0.0/0 port 4000.
- My intuition tells me it has to do with the fact we start the server in offline mode, but I would need to investigate that and I suspect it might take me a while.
- Decision was made to send it as is; this work in its current state is enough to demonstrate I have the ability to get things done using GitHub, AWS, Jenkins, CloudFormation and Linux.
- I would be interested in spending some time with you trying to get it to 100%.

## Approach

### Dev env setup

- My personal laptop was already setup with Jenkins and a running instance of DynamoDB, plus I already had some CloudFormation code ready to spin up an EC2 + R53 + local DDB in AWS, so I went with that,
- I used a mix of *vi* and *Visual Studio Code* to edit my code,
- I used the *git* command line tool to interact with GitHub.

### Principles applied

1. Get it running first, improve later,
2. Use CloudFormation Outputs to communicate useful information to end user,
3. Check log files even when everything seems fine,
4. Understand what you're doing, don't blindly copy/paste stuff from StackOverflow (one of the reasons why I went through the tutorial first),
5. On GitHub, a good product without proper documentation will always look bad.
