Example Voting App
=========

Getting started
---------------

Download [Docker](https://www.docker.com/products/overview). If you are on Mac or Windows, [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

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
4. Run `az --version` to verify the installation.

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.