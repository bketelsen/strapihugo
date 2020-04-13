---
title: Azure DevOps CI Setup
slug: Azure-DevOps-CI-Setup
date: '2018-09-11'

---

Microsoft just announced a killer continuous integration setup for Github, and I was really anxious to give it a try.  This post documents my setup for an [Elixir/Phoenix application](https://github.com/gophersnacks/site) I'm building with [Aaron Schlesinger](https://twitter.com/arschles).

### Set Up Azure Pipelines

Step one is to head to the [Github Marketplace](https://github.com/marketplace/azure-pipelines) and install the Azure Pipelines service.

![Azure Pipelines on Github](https://content.brian.dev/uploads/3d79ad4e82704fa381a8f32af094b91f.png)

I clicked on the "Set up a plan" button, and then selected the Free option, and "Install it for free" link.  This allows Linux, Mac and Windows build machines, with 10 free parallel jobs.  That's plenty for most open source projects.

![Confirmation Dialog](https://content.brian.dev/uploads/84e39a4b6bc54504b6c7a9661d211c4d.png)

Next I confirmed the Marketplace change by choosing the "Complete order and begin installation" button.

![Review Order](https://content.brian.dev/uploads/3261b755243c466ba2072d618f7e0aa7.png)

After authentication with my Azure account, I needed to create a Project.  Projects in Azure DevOps are conceptual containers that host a single repository and it's build procedures.

![Setup Project](https://content.brian.dev/uploads/e2c4441bab0540e9bcbefc2b52b15c3e.png)

After creating the project, I needed to pick a repository:

![Choose A Repository](https://content.brian.dev/uploads/d2cb837e0c7d4eac98c6eb2f0fd9d3a0.png)

### Configuration

The Azure DevOps pipeline wizard suggested a Docker pipeline for this Elixir project, which suits me fine.  We had already created appropriate Dockerfile and docker-compose.yml files.  The last step was to create an azure-pipelines.yml file and put it in the root of the repository.

```javascript
# Docker image
# Build a Docker image to run, deploy, or push to a container registry.
# Add steps that use Docker Compose, tag images, push to a registry, run an image, and more:
# https://docs.microsoft.com/vsts/pipelines/languages/docker

pool:
  vmImage: 'Ubuntu 16.04'

variables:
  imageName: 'gophersnacks:$(build.buildId)'

steps:
- script: |
    docker-compose -p site up -d
    docker build --network=site_default --build-arg MIX_ENV=test -f Dockerfile.test .
    docker-compose down
  displayName: 'test'
- script: docker build -f Dockerfile -t $(imageName) .
  displayName: 'docker build'
 ```

 After some experimentation I created two steps in my pipeline, the first runs tests, and the second builds the Docker container for production usage.  As is the case every time I create a new CI environment, it took 40 commits to get the Docker networking and database configuration setup right.  Eventually, though, I succeeded:
 
 ![FAIL](https://content.brian.dev/uploads/512fbfeeead148ada3b15c495993b143.png)

 These results were correct; I hadn't updated the tests since adding authentication to the web app yesterday.  *So I did what any good developer would do – I deleted the failing tests.*  Just temporarily, of course, because I wanted to focus my time on configuring Azure Pipelines, not learning how to test authenticated HTTP requests in Phoenix.

### Conclusion

The process to set up Azure DevOps Pipelines was really simple, and mostly involved clicking on a series of green buttons and integrating the Docker setup we had already created.  I was impressed by the speed of the builds after I pushed my commits to Github.  I think we have a winner with the new DevOps releases.  You can find the documentation on [docs.microsoft.com](https://cda.ms/F8).

In future posts, we'll get this Pipeline configured to push to a Kubernetes cluster to add Continuous Deployment to this application.
