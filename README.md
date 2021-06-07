# Docker image for inbound Jenkins agents

## **Modified Fork**

This is a fork of https://github.com/kevit/docker-inbound-agent which is a fork of the original inbound agent.  Kevit's
fork contains a necessary bugfix so that one can pass the secret and name as positional parameters into Windows-based containers 
so as to align with the Linux-based containers.  This is necessary so the [Jenkins Amazon ECS plugin](https://plugins.jenkins.io/amazon-ecs/) can properly start Windows-based
Jenkins agents.

### Additional Build Tools

This fork also contains additional build tools (AWS CLI, MSBuild, Docker, Docker Compose, etc) so that Windows-based Galeo applications
can be compiled, tested, containerized.  An improvement of this approach would be to generate the Kevit Docker image and then
use it as a starting point in another project which adds the necessary tools.  

### Building/Deploying This Custom Fork

From a PowerShell window, navigate to the root directory of this repository.  Execute the following to build.
    
    .\make.ps1

Once images are created, you want to authenticate against the Galeo Group's AWS Root account and upload the `windowsservercore-1809` tag.  
The Jenkins build environment will automatically utilize the new image.

1. Retrieve an authentication token and authenticate your Docker client to your registry.
    ```
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 465728785746.dkr.ecr.us-east-1.amazonaws.com
   ```
2. Tag the image.
    ```
    docker tag jenkins/inbound-agent:windowsservercore-1809 465728785746.dkr.ecr.us-east-1.amazonaws.com/jenkins-inbound-agent:windowsservercore-1809
    ```
3. Push the image into the `jenkins-inbound-agent` repository in AWS ECR.
    ```
    docker push 465728785746.dkr.ecr.us-east-1.amazonaws.com/jenkins-inbound-agent:windowsservercore-1809
    ```

[![Join the chat at https://gitter.im/jenkinsci/docker](https://badges.gitter.im/jenkinsci/docker.svg)](https://gitter.im/jenkinsci/docker?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![GitHub stars](https://img.shields.io/github/stars/jenkinsci/docker-inbound-agent?label=GitHub%20stars)](https://github.com/jenkinsci/docker-inbound-agent)
[![Docker Pulls](https://img.shields.io/docker/pulls/jenkins/inbound-agent.svg)](https://hub.docker.com/r/jenkins/inbound-agent/)
[![GitHub release](https://img.shields.io/github/release/jenkinsci/docker-inbound-agent.svg?label=changelog)](https://github.com/jenkinsci/docker-inbound-agent/releases/latest)

:exclamation: **Warning!** This image used to be published as [jenkinsci/jnlp-slave](https://hub.docker.com/r/jenkinsci/jnlp-slave/) and [jenkins/jnlp-slave](https://hub.docker.com/r/jenkins/jnlp-slave/).
These images are deprecated, use [jenkins/inbound-agent](https://hub.docker.com/r/jenkins/inbound-agent/).

This is an image for [Jenkins](https://jenkins.io) agents using TCP or WebSockets to establish inbound connection to the Jenkins master.
This agent is powered by the [Jenkins Remoting library](https://github.com/jenkinsci/remoting), which version is being taken from the base [Docker Agent](https://github.com/jenkinsci/docker-agent/) image.

See [Jenkins Distributed builds](https://wiki.jenkins-ci.org/display/JENKINS/Distributed+builds) for more info.

## Running

To run a Docker container

  Linux agent:

    docker run --init jenkins/inbound-agent -url http://jenkins-server:port <secret> <agent name>
  Note: `--init` is necessary for correct subprocesses handling (zombie reaping)

  Windows agent:

    docker run jenkins/inbound-agent:windowsservercore-1809 -Url http://jenkins-server:port -Secret <secret> -Name <agent name>

To run a Docker container with [Work Directory](https://github.com/jenkinsci/remoting/blob/master/docs/workDir.md)

  Linux agent:

    docker run --init jenkins/inbound-agent -url http://jenkins-server:port -workDir=/home/jenkins/agent <secret> <agent name>

  Windows agent:

    docker run jenkins/inbound-agent:windowsservercore-1809 -Url http://jenkins-server:port -WorkDir=C:/Jenkins/agent -Secret <secret> -Name <agent name>

Optional environment variables:

* `JENKINS_URL`: url for the Jenkins server, can be used as a replacement to `-url` option, or to set alternate jenkins URL
* `JENKINS_TUNNEL`: (`HOST:PORT`) connect to this agent host and port instead of Jenkins server, assuming this one do route TCP traffic to Jenkins master. Useful when when Jenkins runs behind a load balancer, reverse proxy, etc.
* `JENKINS_SECRET`: agent secret, if not set as an argument
* `JENKINS_AGENT_NAME`: agent name, if not set as an argument
* `JENKINS_AGENT_WORKDIR`: agent work directory, if not set by optional parameter `-workDir`
* `JENKINS_WEB_SOCKET`: `true` if the connection should be made via WebSocket rather than TCP

## Configuration specifics

### Enabled JNLP protocols

As of version 3.40-1 this image only supports the [JNLP4-connect](https://github.com/jenkinsci/remoting/blob/master/docs/protocols.md#jnlp4-connect) protocol.
Earlier, long-unsupported protocols have been removed.
As a result, Jenkins versions prior to 2.32 are no longer supported.

### Amazon ECS

Make sure your ECS container agent is [updated](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-update.html) before running. Older versions do not properly handle the entryPoint parameter. See the [entryPoint](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions) definition for more information.
