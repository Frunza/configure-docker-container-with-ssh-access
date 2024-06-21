# Configure docker container with SSH access

## Scenario

Often you want to make some changes on a virtual machine or remote machine from a pipeline script; this is usually done by using `Ansible`, but even before that, SSH access to the target machine must be available.

Assuming that you have a main script that prepares a docker container to do the workload, how do you ensure that the docker container has access to a target machine that it wants to access?

## Prerequisites

A Linux or MacOS machine for local development. If you are running Windows, you first need to set up the *Windows Subsystem for Linux (WSL)* environment.

You need `docker cli` and `docker-compose` on your machine for testing purposes, and/or on the machines that run your pipeline.
You can check both of these by running the following commands:
```sh
docker --version
docker-compose --version
```

Set the following environment variable for SSH access:
- TARGET_MACHINE_SSH_PRIVATE_KEY
You can name it however you want. For convenience, you can add it directly to your profile. This can look like this, for example:
```sh
export TARGET_MACHINE_SSH_PRIVATE_KEY="-----BEGIN OPENSSH PRIVATE KEY-----
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaa
-----END OPENSSH PRIVATE KEY-----"
```
Note the syntax: it needs double quotation marks at the beginning and the end of the ssh key value.

## Implementation

When building the docker image, pass an argument to it to provide the value of the `TARGET_MACHINE_SSH_PRIVATE_KEY` environment variable:
```sh
--build-arg SSH_PRIVATE_KEY="$TARGET_MACHINE_SSH_PRIVATE_KEY"
```

Add the following in your docker file:
```sh
ARG SSH_PRIVATE_KEY
RUN mkdir -p /root/.ssh
RUN echo "$SSH_PRIVATE_KEY" | tr -d '\r' > /root/.ssh/id_rsa && chmod 600 /root/.ssh/id_rsa
```
This will copy the value of `SSH_PRIVATE_KEY` to the standard location for SSH keys.

## Usage

Navigate to the root of the git repository and run the following commands:
```sh
sh run.sh 
```

The following happens:
1) the first command builds the docker image, passing the private key value as an argument and tagging it as *sshaccess*
2) the docker image sets up the SSH access by copying the value of the `SSH_PRIVATE_KEY` argument to the standard location for SSH keys
3) the second command uses docker-compose to create and run the container. The container runs an SSH check against a target machine and prints some output to inform whether it was successful or not
