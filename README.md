this is rubbish

## This repository

This contains lots of different kinds of builds as listed below. Just follow their instructions to try them out.

After you have forked this repository.

There are a couple ways to try it out.

  1. Run this on your installation of Drone. You will be able to add your forked repository, and then run builds.
  2. Run this on [cloud.drone.io](cloud.drone.io) You will need to have an established github account. Once you log into cloud.drone.io you will be able to add your forked repository, and then run builds.
  3. Follow the instructions for setting up drone in aws, at the bottom of this readme. You will be able to add your forked repository, and then run builds.

## Drone Basics

### Conditionals

Here is a concrete example of conditional usage in drone. For more complex usage look [here](https://docs.drone.io/pipeline/conditions/)
To try this build. In the settings of this repo in you drone ui. Set the path for the drone file to `./basic/conditional.yml`

### Parallelism

Here is a concrete example of parallel builds in drone. For more complex usage look [here](https://docs.drone.io/pipeline/docker/syntax/parallelism/)
To try this build. In the settings of this repo in you drone ui. Set the path for the drone file to `./basic/parallel.yml`

### Triggers and multiple pipelines

Here is a concrete example of triggers and multiple pipelines in drone.
To try this build. In the settings of this repo in you drone ui. Set the path for the drone file to `./basic/triggers_and_pipelines.yml`

### Further reading into what is possible

For more complex usage docker usage look [here](https://docs.drone.io/quickstart/docker/)

## Language specific build examples

### Golang build

Code lives here in the `golang` folder, it contains an example go project and a basic `.drone.yaml` file that:

- go vet
- go test
- builds a binary

To try this build. In the settings of this repo in you drone ui. Set the path for the drone file to `./golang/.drone.yml`

For more advanced information on golang builds go [here](https://docs.drone.io/pipeline/kubernetes/examples/language/golang/)

### Java build

Code lives here in the `java` folder, it contains an example project and a basic `.drone.yaml` file that:

- builds a class
- runs the class

To try this build. In the settings of this repo in you drone ui. Set the path for the drone file to `./java/.drone.yml`

For more advanced information on Java builds go [here](https://docs.drone.io/pipeline/kubernetes/examples/language/java/)

## Running and installing Drone in AWS using Github as the git provider

**THIS IS NOT RECOMMENDED FOR PRODUCTION** **THIS IS NOT RECOMMENDED FOR PRODUCTION**

The setup here is for testing purposes only, there are settings here that are inherently insecure (the open ssh/http ports).

### AWS setup

- Go to your EC2 [dashboard](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Home:) we are using Ohio region us-east-2
- Click on the Launch Instance button
- On screen `1` Ubuntu 20.04 - You may need to click the subscribe button to get the latest image.
- On screen `2` t2.medium 4gb mem
- On screen `3` In advanced details - inside 'user data' paste in the following

```BASH
#cloud-config
users:
- default
- name: root
  sudo: ALL=(ALL) NOPASSWD:ALL
  groups: sudo

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88
      
packages:
- docker-ce
- vim

write_files:
- path: /root/docker-compose.yml
  content: |
    version: "3.8"
    services:
        drone:
            image: drone/drone:latest
            ports:
            - "80:80"
            environment:
            - DRONE_SERVER_HOST=${AWS_HOSTNAME}
            - DRONE_SERVER_PROTO=http
            - DRONE_RPC_SECRET=bea26a2221fd8090ea38720fc445eca6
            - DRONE_COOKIE_SECRET=e8206356c843d81e05ab6735e7ebf075
            - DRONE_COOKIE_TIMEOUT=720h
            - DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}
            - DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}
            - DRONE_LOGS_DEBUG=true
            - DRONE_CRON_DISABLED=true
            volumes:
            - ./data:/data
        runner:
            image: drone/drone-runner-docker:latest
            environment:
            - DRONE_RPC_HOST=drone
            - DRONE_RPC_PROTO=http
            - DRONE_RPC_SECRET=bea26a2221fd8090ea38720fc445eca6
            - DRONE_TMATE_ENABLED=true
            volumes:
            - /var/run/docker.sock:/var/run/docker.sock
runcmd:
  - [ systemctl, daemon-reload ]
  - [ systemctl, restart, docker ]
  - sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  - sudo chmod +x /usr/local/bin/docker-compose

output: {all: '| tee -a /var/log/cloud-init-output.log'}
```

This script installs docker, docker-compose, vim and creates '/root/docker-compose.yml' for you.

- On screen `4` select the storage size
- On screen `5` add any neccessary tags
- On screen `6` set up a security group to allow ports 80 and 22 (http/ssh) from anywhere.
- Review your settings, finally select your key pair you have created / or create a new key pair.

### Setting up Github Oauth

- Setup [Github Oauth](https://github.com/settings/developers)
- Set the home page to `http://localhost` * this is not important
- Set the `Authorization callback URL` to `http://${AWS_HOSTNAME}/login`
- Create your client id and secret. Keep these safe !! this is the only time you will see these.

### Running Drone

- SSH into the box as ubuntu using your aws pem file `ssh -i key_pair.pem ubuntu@${AWS_HOSTNAME}`. Use whatever your key pair is called.
- Change to the root user `sudo su`
- Change directory into the root folder `cd /root`
- Edit the docker compose file `vim docker-compose.yml`

Changing the following 3 settings

```BASH
DRONE_SERVER_HOST=${AWS_HOSTNAME}                          # your aws instance hostname
DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}           # taken from your Github oauth application
DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}   # taken from your Github oauth application
```

- Run the following command to start the Docker containers `docker-compose up`. You should see the Drone server and Drone runner start.
- Open a browser and go to `http://{AWS_HOSTNAME}`

## TODO's

- Add workspace setting for golang .drone.yml, to remove cd hack.
