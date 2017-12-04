Example Voting App
=========

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

Setup environment
-----

### Create required accounts

1. Create a [Microsoft Azure account](https://azure.microsoft.com).
2. Create a [Docker Hub account](https://hub.docker.com/).

### Install Git

1. Install [Git](https://git-scm.com/download/win).

### Install docker client

1. Download the [docker client](https://download.docker.com/win/static/stable/x86_64/docker-17.09.0-ce.zip), extract the files and copy `docker.exe` to a separate folder.
2. Download [docker-compose](https://github.com/docker/compose/releases/download/1.17.1/docker-compose-Windows-x86_64.exe), rename it to `docker-compose.exe` and copy to the same folder.
3. Add the folder containing the docker binaries to PATH.

### Install Putty

Download [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

### Install azure-cli

1. Download [python](https://sourceforge.net/projects/winpython/files/WinPython_2.7/2.7.13.1/).
2. Run the installation, and add `python-2.7.13.amd64` and `python-2.7.13.amd64/Scripts` to PATH.
3. Run `pip install azure-cli`.
4. Login into Azure by running `az login` and following the procedure.

Create a Docker host
------

To be able to run the application we need first to set up a remote docker host. Open Git bash and execute the following commands.

1. Set a namespace variable to isolate your resources from others:
```bash
export NAMESPACE=<your name>
```
2. Create a resource group:
```bash
az group create --name $NAMESPACE --location westeurope
```
3. Create a Virtual Machine:

```
az vm create --resource-group $NAMESPACE --name $NAMESPACE-docker --image UbuntuLTS --admin-username dev --admin-password DockerDocker-1
```
**Note the public IP of the VM**. Also note the user name and the password.
4. Open the required ports on the VM:
```
az vm open-port -g $NAMESPACE -n $NAMESPACE-docker --port 80 --priority 900
az vm open-port -g $NAMESPACE -n $NAMESPACE-docker --port 2375 --priority 800
az vm open-port -g $NAMESPACE -n $NAMESPACE-docker --port 5000-5001 --priority 700
```
5. SSH into the VM to install and run the Docker daemon:
```
sudo apt install aufs-tools docker.io
systemctl stop docker
sudo dockerd -H tcp://0.0.0.0:2375
```
6. To verify the installation, on your local machine run:
```
export DOCKER_HOST=tcp://<ip address of the VM:2375
docker run hello-world
```

Run the application
---

1. Clone the project:
```
git clone https://github.com/ctco-dev/example-voting-app.git
```
2. Navigate to the project folder and run the application:
```
cd example-voting-app
docker-compose up
```
Wait until everything is downloaded.

The voting application will be available at `http://<your VM's IP>:5000` and the result application at `http://<your VM's IP>:5001`.

Change the application
----

The next step would be to change the application from a cat-dog vote to a Java-Node vote.

1. Stop the running application by pressing Ctrl+C.
2. Open `vote/app.py` and replace all occurrences of `Cats` with `Java` and `Dogs` with `Node`. Do the same for `result/views/index.html`.
3. Start the application by running:
```
docker-compose up --build
```
Note the `--build` argument.

Refresh the application in your browser and make sure, that the changes have been applied correctly.

Set up Redis and PostgreSQL on Azure
---

### Setup Redis
1. Create a Redis instance:
```
az redis create --location westeurope --name $NAMESPACE-redis --resource-group $NAMESPACE --sku Basic --vm-size C0
```
Note the value of the `hostName` property.
2. Retrieve the password to Redis:
```
az redis list-keys --name $NAMESPACE-redis --resource-group $NAMESPACE
```
Note the value of the `primaryKey` property.
3. Create a `.env` in the root of your project with the following contents:
```
REDIS_HOST=<host name>
REDIS_PASSWORD=<primary key>
REDIS_PORT=6380
```

### Setup PostgreSQL

1. Create a PostgreSQL instance:
```
az postgres server create --resource-group $NAMESPACE --name $NAMESPACE-postgresql  --location westeurope --admin-user dev --admin-password Postgres-1 --performance-tier Basic --compute-units 50 --version 9.6 --ssl-enforcement Disabled
```
Note the value of the `fullyQualifiedDomainName` and `name` properties.
2. Allow to connect from any IP:
```
az postgres server firewall-rule create -g $NAMESPACE -s $NAMESPACE-postgresql -n allowall --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```
3. Add the following contents to your `.env` file:
```
POSTGRES_HOST=<your fullyQualifiedDomainName>
POSTGRES_USER=dev@<your instance name>
POSTGRES_PASSWORD=Postgres-1
```

### Test

Restart your applications to verify the changes.

Publish the application
----

To deploy the applications to Azure we need to publish the images to Docker Hub first.
1. Login into Docker Hub by running:
```
docker login
```
Enter your user name and password.
2. Create a `docker-compose.override.yml` file:
```
version: "3"

services:
  vote:
    image: <docker user name>/voting-app-vote

  result:
    image: <docker user name>/voting-app-result

  worker:
    image: <docker user name>/voting-app-worker
```
3. Publish the images:
```
docker-compose push
```

Deploy the application
----

Now it's time to deploy the applications to Azure.

1. Create a service plan:
```
az appservice plan create --name $NAMESPACE-asp --resource-group $NAMESPACE --sku S1 --is-linux
```
2. Create three applications and configure the required settings:
```
az webapp create --resource-group $NAMESPACE --plan $NAMESPACE-asp --name $NAMESPACE-vote \
    --deployment-container-image-name <docker user>/voting-app-vote
az webapp config appsettings set -g $NAMESPACE -n $NAMESPACE-vote --settings \
    REDIS_HOST=<insert value from .env> \
    REDIS_PASSWORD=<insert value from .env> \
    REDIS_PORT=6380
az webapp config set -g $NAMESPACE -n $NAMESPACE-vote --always-on true
```
```
az webapp create --resource-group $NAMESPACE --plan $NAMESPACE-asp --name $NAMESPACE-result \
    --deployment-container-image-name <docker user>/voting-app-result
az webapp config appsettings set -g $NAMESPACE- -n $NAMESPACE-result --settings \
   POSTGRES_USER=<insert value from .env> \
   POSTGRES_PASSWORD=<insert value from .env>
az webapp config set -g $NAMESPACE -n $NAMESPACE-result --always-on true
```
```
az webapp create --resource-group $NAMESPACE --plan $NAMESPACE-asp --name $NAMESPACE-worker \
    --deployment-container-image-name <docker user>/voting-app-worker
az webapp config appsettings set -g $NAMESPACE -n $NAMESPACE-worker --settings \
    REDIS_HOST=<insert value from .env> \
    REDIS_PASSWORD=<insert value from .env> \
    REDIS_PORT=6380 \
    POSTGRES_HOST=<insert value from .env> \
    POSTGRES_USER=<insert value from .env> \
    POSTGRES_PASSWORD=<insert value from .env> \
    WEBSITES_CONTAINER_START_TIME_LIMIT=600
az webapp config set -g $NAMESPACE -n $NAMESPACE-worker --always-on true
```
3. After some time the two frontend applications will be available at `<namespace>-vote.azurewebsites.net` and `<namespace>-result.azurewebsites.net`

Clean up resources
----

Once you're done playing around with your application, cleanup all of the resources you've created in Azure:
```
az group delete --name $NAMESPACE
```

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.
