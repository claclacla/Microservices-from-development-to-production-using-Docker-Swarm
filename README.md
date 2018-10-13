# Microservices from development to production using Docker Swarm

## Compose and deploy the application infrastructure

--------------------------------------------------------------------------------

### Prerequisites

What things you need to install the software

```
docker 18+
docker-compose 1.20.1+
docker-machine 0.14.0+

```

--------------------------------------------------------------------------------

### Development

```bash
# Move to the docker/dev folder
cd docker/dev

# Create a .env file with your local absolute path to the application folder
echo "APP_FOLDER=/your-path-to-the/application-folder" > .env

# Compose the docker containers
sudo docker-compose up -d

```

--------------------------------------------------------------------------------

### Production

#### Create the services image and push it to the Docker Hub repository 

```bash
# Move to the project main folder
cd /your-path-to-the/application-folder

# Build the docker image
docker build -f docker/production/Dockerfile . -t <username>/<application name>

# Define the organization user ID
export DOCKER_ID_USER=username

# Login with your Docker data
docker login

# Push the application image to the DockerHub
docker push <username>/<application name>

```

#### Create the Docker cluster

```bash
# Create the docker machines
docker-machine create --driver=virtualbox NodeManager
docker-machine create --driver=virtualbox NodeWorker1

# Use the following command to read the manager ip
docker-machine ls

# Instruct the manager to become a swarm manager
docker-machine ssh NodeManager "docker swarm init --advertise-addr <manager ip>"

# The response will look like this:

# Swarm initialized: current node (q4udo31nbhfp1zsi9h8bzfkgm) is now a manager.
#
# To add a worker to this swarm, run the following command:
#
#    docker swarm join --token SWMTKN-1-3mprdn0gjmijoauacc2m20ezanhpst3m8j3lfb7a3fkc4fwzri-48uizyf8dwuo7mrm3zanqd2ll 192.168.99.100:2377
#
# To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# Use the command above to have worker1 join the swarm 
docker-machine ssh NodeWorker1 "docker swarm join --token <token> <ip>:2377"

# Use this command to control the swarm status
docker-machine ssh NodeManager "docker node ls"

```

#### Deploy the app to the Docker cluster

```bash
# Get the command to talk with NodeManager
docker-machine env NodeManager

# The response will look like this:

# export DOCKER_TLS_VERIFY="1"
# export DOCKER_HOST="tcp://192.168.99.100:2376"
# export DOCKER_CERT_PATH="/root/.docker/machine/machines/NodeManager"
# export DOCKER_MACHINE_NAME="NodeManager"
# # Run this command to configure your shell: 
# # eval $(docker-machine env NodeManager)

# Use the previous command to configure the shell
eval $(docker-machine env NodeManager)

# Use the following command to verify your shell
docker node ls

# Go to the docker production folder
cd docker/production

# Use the following command to deploy the app
docker stack deploy --with-registry-auth -c docker-compose.yaml App

# Check the stack status
docker stack ps App

# Check the processes status
docker service ls

# Show the process logs
docker service logs <service name> --raw

# If needed remove the stack
docker stack rm NodeApp

```

--------------------------------------------------------------------------------

## Authors

- **Simone Adelchino** - [_claclacla_](https://twitter.com/_claclacla_)

--------------------------------------------------------------------------------

## License

This project is licensed under the MIT License

--------------------------------------------------------------------------------

## Acknowledgments

- [Get started with Docker](https://docs.docker.com/get-started)
- [Install Docker](https://docs.docker.com/install)
- [Install Docker compose](https://docs.docker.com/compose/install)
- [Install Docker machine](https://docs.docker.com/machine/install-machine)