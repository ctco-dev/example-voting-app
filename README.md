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

### Windows

#### Install docker client
1. Download the [docker client](https://download.docker.com/win/static/stable/x86_64/docker-17.09.0-ce.zip), extract the files and copy `docker.exe` to a separate folder.
2. Download [docker-compose](https://github.com/docker/compose/releases/download/1.17.1/docker-compose-Windows-x86_64.exe), rename it to `docker-compose.exe` and copy to the same folder.
3. Add the folder containing the docker binaries to PATH.

#### Install Putty
Download [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

#### Install azure-cli

1. Download [python](https://sourceforge.net/projects/winpython/files/WinPython_2.7/2.7.13.1/).
2. Run the installation, and add `python-2.7.13.amd64` and `python-2.7.13.amd64/Scripts` to PATH.
3. Run `pip install azure-cli`.
4. Login into Azure by running `az login` and following the procedure.

Run application
------

To be able to run the application we need first to set up a remote docker host.

1. Set a namespace variable to isolate your resources from others:
```bash
export NAMESPACE=<your name>
```

1. Create a resource group:
```bash
az group create --name $NAMESPACE-docker --location westeurope
```
2. Create a Virtual Machine:

```
az vm create --resource-group $NAMESPACE-docker --name $NAMESPACE-docker --image UbuntuLTS --admin-username dev --admin-password DockerDocker-1
```
**Note the public IP of the VM**. Also note the user name and the password.

3. Open the required ports on the VM:
```
az vm open-port -g $NAMESPACE-docker -n $NAMESPACE-docker --port 80 --priority 900
az vm open-port -g $NAMESPACE-docker -n $NAMESPACE-docker --port 2375 --priority 800
az vm open-port -g $NAMESPACE-docker -n $NAMESPACE-docker --port 5000-5001 --priority 700
```
4. SSH into the VM to install and run the Docker daemon:
```
sudo apt install aufs-tools docker.io
systemctl stop docker
sudo dockerd -H tcp://0.0.0.0:2375
```

5. To verify the installation, on your local machine run:
```
export DOCKER_HOST=tcp://<ip address of the VM:2375
docker run hello-world
```

Change the application
----

TODO

Publish the application
----

TODO

Deploy the application
----

TODO

Clean up resources
----

TODO

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.