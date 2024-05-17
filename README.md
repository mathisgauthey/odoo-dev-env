# odoo-dev-env

My Odoo dev env that I use in WSL. There's also some Docker to play around with containers. This repository is licensed under the [MIT license](https://mathisgauthey.mit-license.org/).

It is meant to allow for local dev on WSL and/or in Docker containers.

The project is a combination of

- [odoo-dev-env-WSL](https://github.com/mathisgauthey/odoo-dev-env-WSL)
- [odoo-dev-env-containers](https://github.com/mathisgauthey/odoo-dev-env-containers)

<!-- ----------------------------------------------------------------------- -->
<!--                                WSL Setup                                -->
<!-- ----------------------------------------------------------------------- -->

## VS Code Odoo Pytest setup

Follow the WSL setup instructions to get a working Odoo dev environment first.

Then, from your virtual env (remember to use `source venv/bin/activate` to get into your venv), install [pytest 7.4.3](https://pypi.org/project/pytest/7.4.3/) and [pytest-odoo](https://github.com/camptocamp/pytest-odoo) :

```bash
pip install pytest==7.4.3
pip install pytest-odoo
```

Then, install Odoo as a pip package :

```bash
cd ./src/odoo
pip install -e .
```

You should now see odoo when doing a `pip list`.

You can now configure PyTest on VS Code, but it should work just fine using the `settings.json` and `config/odoo.conf` files provided in this repository.

## WSL Setup

File structure :

```txt
.
└── odoo-dev/
    ├── .vscode/
    ├── odoo/
    ├── odoo_venv/
    ├── LICENSE
    ├── README.md
    └── setup.sh
```

### Install WSL

1. Install WSL by opening a `CMD` and using the following command : `wsl --install`. This will install the Ubuntu LTS[^1] version.
2. `sudo apt update && sudo apt upgrade`.

### Setup a database using PostgreSQL

1. Install PostgreSQL in WSL using `sudo apt install postgresql`.
2. Login to user postgres : `sudo -u postgres psql postgres`.
3. Create a user using the following command : `CREATE USER odoo WITH PASSWORD odoo`
4. Add a line to `pg_hba.conf` file to allow connections for the odoo user :

```conf
TYPE DATABASE USER ADRESS METHOD
local all odoo scram-sha-256
```

![pg_hba.conf](https://github.com/mathisgauthey/odoo-dev-env-WSL/assets/46576952/951ce90e-6306-437c-8a82-7d1168c9225a)

NOTE: You can also use a Docker container using Rancher Desktop installed on Windows for a more production-like environment and less struggle for setting up the database.

### Setup your dev environment

1. Clone this repository.
2. Go inside the repo using `cd` and use `bash setup.sh` to :
   1. Install the required Ubuntu dependencies.
   2. Clone Odoo source code.
   3. Setup a python virtual environment.
   4. Install Odoo specific dependencies inside the python venv.

You can now start the Odoo server using `./odoo/odoo-bin` and access it using `localhost:8069`.

If you're on VSCODE, simply press `CTRL+F5` to start the server, or `F5` to start the server in debug mode.

If you're using Jetbrains softwares such as PyCharm or Intellij IDEA, inspire yourself from the `.vscode/launch.json` file to setup your launch and debug configuration.

### Use and create Odoo modules

Based on [odoo docs](https://www.odoo.com/documentation/17.0/developer/tutorials/getting_started/01_architecture.html#module-structure), the following directory structure should be applied when developping odoo addons:

```txt
.
└── REPO_NAME/
    ├── .git/
    ├── MODULE_NAME/
    │   ├── data/
    │   ├── models/
    │   ├── controllers/
    │   ├── views/
    │   ├── static/
    │   ├── wizard/
    │   ├── report/
    │   ├── tests/
    │   ├── __init__.py
    │   ├── __manifest__.py
    │   └── ... # Other module files and folders (Detailled below)
    ├── .gitignore
    ├── LICENSE
    ├── README.md
    └── requirements.txt
```

Odoo.sh is handling `requirements.txt` files in [the parent folder of the module](https://www.odoo.com/documentation/17.0/administration/odoo_sh/advanced/containers.html#overview). We're not using Odoo.sh, but we could still use the same structure to keep things consistent.

If you don't use Odoo.sh, you might need to use a bash script that looks for `requirements.txt` files and install the required dependencies.

You can find informations about [coding guidelines in details here](https://www.odoo.com/documentation/17.0/contributing/development/coding_guidelines.html).

> A module is organized in important directories. Those contain the business logic; having a look at them should make you understand the purpose of the module.
>
> - data/ : demo and data xml
> - models/ : models definition
> - controllers/ : contains controllers (HTTP routes)
> - views/ : contains the views and templates
> - static/ : contains the web assets, separated into css/, js/, img/, lib/, …
>
> Other optional directories compose the module:
>
> - wizard/ : regroups the transient models (models.TransientModel) and their views
> - report/ : contains the printable reports and models based on SQL views. Python objects and XML views are included in this directory
> - tests/ : contains the Python tests

I'd recommend to add a `odoo-dev-env-WSL/odoo/dev/` folder to store your modules and use the `--addons_path` argument to add it to the Odoo server. You can inspire yourself from the `launch.json` file to do so.

### Configure Odoo

to configure Odoo, we get to choose if we prefer to use :

- [The `odoo.conf` file](https://www.odoo.com/documentation/17.0/administration/install/deploy.html)
- [The command line arguments](https://www.odoo.com/documentation/17.0/developer/reference/cli.html#cmdoption-odoo-bin-d) by replacing or adding to the default Docker `CMD` with `odoo --ARG_NAME ARG_VALUE` in the `Dockerfile` file.

A good practice is to **use only one** of the two methods to configure Odoo.

Note that some arguments are [named differently](https://www.odoo.com/documentation/17.0/developer/reference/cli.html#configuration-file) depending on the method used :

- `--db-filter` becomes `dbfilter` in `odoo.conf`
- `--database` becomes `db_name` in `odoo.conf`
- `--addons-path` becomes `addons_path` in `odoo.conf`
- `--data-dir` becomes `data_dir` in `odoo.conf`

We're using WSL here and will be using launch arguments in the `launch.json` file.

### How to debug

Press `F5` to start the server and `F5` to start it in debug mode. You can then set breakpoints and use the debug console to interact with the server.

### How to start fresh

1. Stop your server.
2. Use `bash drop_db.sh` to drop the database.
3. Relaunch the server to install and upgrade the plugins automatically.

[^1]: Long Term Support

<!-- ----------------------------------------------------------------------- -->
<!--                              Docker Setup                               -->
<!-- ----------------------------------------------------------------------- -->

## Docker Setup

This repo is meant to **simplify the dev environment for odoo development**.

The only prerequisite is to have [Docker](https://www.docker.com/) installed on your machine (or [Rancher Desktop](https://rancherdesktop.io/)), and Git to clone this repo. I'd still suggest to clone this repo inside WSL as Linux will always feel better than Windows for development, and both Windows Docker and Rancher desktop supports linking to WSL2.

Using Docker allows us to :

1. Have a simple onboarding process for new developers without having to struggle with unmet dependencies or different OS. It's literraly a matter of cloning the repo and running `docker-compose up`.
2. Have a consistent environment across all developers.
3. Have a similar environment for testing and production.

> No more manual set up of the database manipulating `pg_hba.conf`, no more manual set up of a python venv, no more manual set up of the odoo configuration and launch arguments, and no more struggling with unmet dependencies on different operating systems.

### What Directory Structure Should I Use to Develop Odoo Addons ?

Based on [odoo docs](https://www.odoo.com/documentation/17.0/developer/tutorials/getting_started/01_architecture.html#module-structure), the following directory structure should be applied when developping odoo addons:

```txt
.
└── REPO_NAME/
    ├── .git/
    ├── MODULE_NAME/
    │   ├── data/
    │   ├── models/
    │   ├── controllers/
    │   ├── views/
    │   ├── static/
    │   ├── wizard/
    │   ├── report/
    │   ├── tests/
    │   ├── __init__.py
    │   ├── __manifest__.py
    │   └── ... # Other module files and folders (Detailled below)
    ├── .gitignore
    ├── LICENSE
    ├── README.md
    └── requirements.txt
```

Odoo.sh is handling `requirements.txt` files in [the parent folder of the module](https://www.odoo.com/documentation/17.0/administration/odoo_sh/advanced/containers.html#overview). We're not using Odoo.sh, but we could still use the same structure to keep things consistent.

If you don't use Odoo.sh, you might need to use a bash script that looks for `requirements.txt` files and install the required dependencies.

You can find informations about [coding guidelines in details here](https://www.odoo.com/documentation/17.0/contributing/development/coding_guidelines.html).

> A module is organized in important directories. Those contain the business logic; having a look at them should make you understand the purpose of the module.
>
> - data/ : demo and data xml
> - models/ : models definition
> - controllers/ : contains controllers (HTTP routes)
> - views/ : contains the views and templates
> - static/ : contains the web assets, separated into css/, js/, img/, lib/, …
>
> Other optional directories compose the module:
>
> - wizard/ : regroups the transient models (models.TransientModel) and their views
> - report/ : contains the printable reports and models based on SQL views. Python objects and XML views are included in this directory
> - tests/ : contains the Python tests

### Database (PostgreSQL) Configuration

We create a database (you need to keep it named `postgres`, but more about this in a [later section](#some-notes-on-the-odoo-and-database-configuration)), a user ([which is not a superuser as required by odoo](https://www.odoo.com/documentation/17.0/administration/install/deploy.html#configuring-odoo)) and a password for the database in the `docker-compose.yml` file.

```yml
environment:
            - POSTGRES_DB=postgres
            - POSTGRES_USER=odoo
            - POSTGRES_PASSWORD=odoo
            - PGDATA=/var/lib/postgresql/data/pgdata
```

You can see `PGDATA=/var/lib/postgresql/data/pgdata` in the `docker-compose.yml` file. This is because of the following:

> [PGDATA](https://hub.docker.com/_/postgres)
>
> Important Note: when mounting a volume to /var/lib/postgresql, the /var/lib/postgresql/data path is a local volume from the container runtime, thus data is not persisted on the mounted volume.
>
> This optional variable can be used to define another location - like a subdirectory - for the database files. The default is /var/lib/postgresql/data. If the data volume you're using is a filesystem mountpoint (like with GCE persistent disks), or remote folder that cannot be chowned to the postgres user (like some NFS mounts), or contains folders/files (e.g. lost+found), Postgres initdb requires a subdirectory to be created within the mountpoint to contain the data.

Note that we used port `5432` for the host and for the container. It might lead to conflict if you have another PostgreSQL instance running on your host. You can change the host port to `5433` if this is the case and you want to access your database in your IDE seemlessly.

### Odoo Configuration

We need to specify the database container `db`, the database container port (`5432` by default with PosgreSQL), the database user and password in the `docker-compose.yml` file.

```yml
environment:
            - HOST=db
            - PORT=5432
            - USER=odoo
            - PASSWORD=odoo
```

Then, to configure Odoo, we get to choose if we prefer to use :

- [The `odoo.conf` file](https://www.odoo.com/documentation/17.0/administration/install/deploy.html)
- [The command line arguments](https://www.odoo.com/documentation/17.0/developer/reference/cli.html#cmdoption-odoo-bin-d) by replacing or adding to the default Docker `CMD` with `odoo --ARG_NAME ARG_VALUE` in the `Dockerfile` file.

A good practice is to **use only one** of the two methods to configure Odoo.

Note that some arguments are [named differently](https://www.odoo.com/documentation/17.0/developer/reference/cli.html#configuration-file) depending on the method used :

- `--db-filter` becomes `dbfilter` in `odoo.conf`
- `--database` becomes `db_name` in `odoo.conf`
- `--addons-path` becomes `addons_path` in `odoo.conf`
- `--data-dir` becomes `data_dir` in `odoo.conf`

As we're using Docker here, we're using the `odoo.conf` file as it is more convenient.

### Docker Volumes and networks

We use Docker volumes to persist the database data and the Odoo addons.

```yml
volumes:
    odoo-data:
    odoo-db-data:
```

Should you want to start fresh, you need to :

1. Stop the containers : `docker-compose down`
2. Remove the containers : `docker-compose rm`
3. Remove the volumes : `docker volume rm VOLUME_NAMES`

We also use a network to link the Odoo and the database containers.

### Some Notes on the Odoo and Database Configuration

We decided to use `odoo-db` as database name. By default, Odoo uses `postgres` as we can see on [DockerHub](https://hub.docker.com/_/odoo/) or on their docs.

If we decide to use ours, we need to be sure that the `odoo.conf` file or the command line arguments are correctly configured.

We need :

- The database host `db_host`
- The database port `db_port`
- The database filter `db_filter`
- The database name `db_name`
- The database user `db_user`
- The database password `db_password`

Based on the [Dockerhub instructions](https://hub.docker.com/_/odoo/), we need to setup the database using the `POSTGRES_DB=postgres` environment variable in the `docker-compose.yml` file.

This is because the database need to be initialiwed when Odoo first start. That's why we gave it a different name in the `odoo.conf` file. Hence when Odoo first starts, it will create the database `odoo-db` and initialise it.

If we'd used `POSTGRES_DB=odoo-db` in the `docker-compose.yml` file, the database would have been created with the name `odoo-db` and Odoo would have failed to start because it would have been unable to find the database `postgres`. It would have asked us to force the database initialisation using `--init base` flag on our first start-up.

### How to access Odoo source code from the container

You can access it by connecting to your container using the following command :

```bash
docker exec -it CONTAINER_NAME bash
```

And then you need to go to `/usr/lib/python3/dist-packages/odoo` to access the source code.

### How to debug

Using VSCODE, you can simply launch your containers using `docker compose up` and then press F5 to attach to it using **debugpy**. If you put some breakpoints inside files of the `addons` folder, you should be able to debug your code.

If you want more informations about this setup, I'd recommend [Khaled Said](https://dev.to/kerbrose/how-to-remote-debugging-odoo-docker-images-python-based-framework-4o2h) or [kerbrose](https://stackoverflow.com/a/63758922/18775070) posts about it.

The following picture comes from their writtings :

![cQ9AR](https://github.com/mathisgauthey/odoo-dev-env-containers/assets/46576952/92049812-2188-4558-b7f1-9fb93c440549)

There is no current way to debug the Odoo source code.

You can probably find a similar setup on Jetbrains IDEs.
