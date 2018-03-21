# CI/CD Automation in practice. Part 1

Not a secret that Continuous Integration and Continuous Deployment is the one of the very important things in any modern project. In this article I`ll try to describe my own CI/CD practice and why it important in our projects.

## Tracking

I`ll start from very begining - backlog. No difference what kind of methodology you use, all releases must be planned and tracked by issue tracker. It very helps to understand what deployed on servers. Common practice here is to use Jira (or GitHub Issues for some open-source projects). 
Each component in your project should has its own version and each commit in CVS should reflect to issue tracker and release planning is the best way to keep it clear.

Imagine in our project we have components `web` and `api` with next planned releases
```
web-1.0.0
  JIRA-0001 Feature 1
  JIRA-0002 Feature 2
api-1.0.0
  JIRA-0003 Feature 3
api-1.0.1
  JIRA-0004 Bug 4
```

As you can see, I used semantic versioning of each component and I highly recommend to use [semver](https://semver.org/) in your projects, further you can see why.

## Workflow

To provide simple CI/CD solution we have to choose issue workflow. It gives us ability to create different strategies for deployment pipelines. Gitflow is one of well-established flows that we applying in our projects. If you not well with gitflow recommend you this article [Succesful Git Branching Model](http://nvie.com/posts/a-successful-git-branching-model/). The same principles can be applied to Forking Workflow if your project is open-source project.

With GitFlow branches we can start with next simple CI/CD strategy:
```
feature -> build + test
develop -> build + test + deploy to dev/demo server
release -> build + test + integrational test + deploy to QA server
master -> build + test + integrational test + deploy to production server
```

Very good practice when you work with feature/hotfix branches is gave them ISSUE-ID, for example:
```
feature/JIRA-0001_feature_1
```
CI web tools such as Bamboo, Teamcity and etc. gives you ability to easy switch into issue details from build plan and visa-versa by detecting issue id. Another good practice is commitment format convention. Here ["How to Write a Git Commit Message"](https://chris.beams.io/posts/git-commit/) you found good article that explain how to write good commits. For our CI/CD stuff important part is message header and I offer to use next format:
```
JIRA-0001 Add Feature 1
```
So, again, each commit will be linked to build and each build will be linked to commit and to issue in issue tracker. 
 
> Keep your develop branch clean. Try to keep simple rule -- One feature - one commit in develop. Always squash your PR`s into develop branch. The same rule with master branch -- One release - one commit. But always merge into master to keep master and develop branches synchronized. 

## Continuous integration

The main idea of CI is as soon as posible recieve knowledge that our feature can be merged into our product without any side effects or even breakable changes. How do we know that our changes will not bring problems? The answer is -- tests. For CI we will divide all tests on unit and integrational. And here appears first metric -- code coverage. 
As a result we want to receive ready to deploy artifact. 

So, to be sure that feature can be implemented we want to know next information from our CI:
  - Build successful (all unit and integrational tests passed)
  - Code coverage satisfied ( ex. not less than 75% )

Different CI tools gives us different abilities but our CI/CD workflow based on branches and we need next from our GIT and CI instruments:
   - Do not allow commit into develop and master branches directly. Only using PR (pull requests).
   - Do not allow merge PR when build failed or not yet started
   - Do not allow merge PR when code coverage too small 

First build plan can consist of next steps:
```
1. Send git-hooks to GIT notified that build and code-coverage started
2. Update version meta information with build number
3. Compile and test 
4. Send git-hook to GIT with success/failure build status
5. Send git-hook with code-coverage result
```

Let see closer to version meta information. As we use semantic versioning we will produce the same artifact name. For ex. for java applications it will be something like `api-1.0.0-SNAPSHOT.jar` and only one way to determine what version was compiled is update `META-INF` with `Implementation-Version: build-123`. For ui applications it can be much simplier and more usefull -- artifact can be `web-1.0.0.zip` or even `web-1.0.0.123.zip` but display version will be `1.0.0.123` where 123 is build version from CI.
Good practice to have in your application place where you can see application version and all dependencies versions. It will save time to understand what builds deployed to environments. 

Step 3 Compile and test. This step requires some dependencies and requirements on build machine. For example - for java applications it is JDK and access to maven or other artifact repository. Clean build on agent has some restrictions:
- version of requirements can be different with app requirements and production environment. In some cases it can be unobvious problem.
- Not always possible to run integrational tests. Run integrational tests against real servers is bad practice in case if it is not part of acceptance tests 

## Docker and Continuous integration
[Docker](https://www.docker.com/) is a lightweight containers what can be very usefull in development, especially in CI.
Benefits:
  - Allow to create virtual test environment per project with necessary dependencies and requirements
  - Allow to emulate real servers for integrational test purposes

How we can integrate docker in our example? 
Our web app depends on api app. Also both app required some database(s) and may be other components.

##### Autonomous virtual environment
Let create dockerfile in each project to build runnable application with environment
Let create docker-compose file for web with next strucure:
```
services:
  database:
    image: microsoft/mssql-server-linux:2017-GA
  web:
    build: .
  api:    
    image: docker.registry.mycompany/myproject/api:1.0.0
```
When we run this environment we will recieve clean environment for integrational tests. It remains to fill the database with neccessary migrations and we can test build of web-app against of real virtual servers.
We see `docker.registry.mycompany/myproject/api:1.0.0` dependency. It means that this dependency must appears before we will run build plan of web app. This dependency relates to latest successfull build of api app with version 1.0.0.
Let add nexts step to api app build plan:
```
6. build docker image from dockerfile
7. push docker image to mycompany docker.registry
```
##### Artifact dependency environment
Most CI tools allow to share artifacts between projects. 
For our build plan we can setup to grab latest successfull api app artifact from master branch. In that case we can setup next docker-compose file strucure:
```
services:
  database:
    image: microsoft/mssql-server-linux:2017-GA
  web:
    build: .
  api:    
    image: openjdk:8
    volumes:
    - ./artifacts:/tmp/artifacts
    command: artifact_install_script.sh
```

To bee continued...
