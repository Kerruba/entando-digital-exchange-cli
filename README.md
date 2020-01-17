# DigitalExchange CLI

## Purpose
This CLI application has the purpose of generate Entando Bundles for the Digital-Exchange using NPM as supporting technology for modules and registry

## Getting started

### Setup a local registry with Nexus from scratch

As a registry you can use whatever technology you prefer. Some examples are the [NPM official registry](https://npmjs.org), [Verdaccio](https://github.com/verdaccio/verdaccio) or [Nexus](https://github.com/sonatype/nexus-public)

For development purposes, let's start a local Nexus repository and set it up as NPM registry

#### Start local nexus as a docker container

Start by running Nexus as a docker container using the docker command or the docker-compose.yml file available in the docker folder:
```
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

Nexus should be available at your localhost at  port 8081

Now you need to sign-in as an admin to configure Nexus and make it usable as a private npm repository. To do so, you need to get the admin credential from inside the container.

```
docker exec -it nexus cat /nexus-data/admin.password
```

Now you can use the password to access your private nexus instance as an admin and change the admin password to something easier for you to work with

#### Setup a private npm registry

> **NOTE**: Nexus allows you to setup both a private registry and a proxy to an external registry. 
For development purposes, having only a private registry could make sense in order to retrieve only local modules and not modules available on remote registries, though feel free to setup also a proxy if you want to get access to npm modules outside of the private registry. 
Check out the [documentation](https://help.sonatype.com/repomanager3/formats/npm-registry#NpmRegistry-ProxyingnpmRegistries) on nexus website for further details.

To setup a  local repository:
1. Go to the `Server administration and configuration` page
2. Go to repositories
3. Create a new repository
4. Choose the **npm (hosted)**
5. Provide a name and save

> **NOTE**: If you want you can also create a group repository to support search from multiple sources (local/proxies) at the same time.

#### Setup npm-realm and user for publishing

In order to be able to login and publish into a repository you need to
1. Create a role to enable user publishing
2. Create a user and assign such roles to him
3. Enable the NPM realm to support `npm adduser` or `npm login` commands

##### Create the role 
- Go to `Security > Roles > Create role > Nexus Role`
- Choose a role ID and name
- In the privileges, add the one required for publishing, e.g. `nx-repository-view-npm-<your-repo>-*`
- Save

##### Create the user
- Go to `Security > Users > Create local user`
- Add the relevant informations for your user, set the user `Active` and add the role you created in the previous step

##### Enable npm realm to support `npm adduser` or `npm login`
- Go to `Security > Realms`
- Add `the npm Bearer Token Realm` to the active column


### Setup a local registry with Nexus using provided volume

You can also decide to start using the provided `base-nexus-data` folder available in the `docker` folder. That contains a private npm registry (npm-internal), has role and a user already defined.

> **Note**: Make a copy of the `base-nexus-data` folder and name it `nexus-data` in the docker folder. That folder is already ignored in git and this way you can continue to work on nexus without updating the current repository. Or copy the entire docker folder somewhere

#### Docker command to include the volume

```
docker run -d -p 8081:8081 --name nexus -v "$(pwd)/nexus-data":/nexus-data sonatype/nexus3
``` 

or if you prefer using docker-compose 

```
cd docker
docker-compose up
```

#### Nexus dashboard
- **Username**: admin
- **Password**: qwe123

#### Npm user to publish
- **Username**: entando-dev
- **Password**: entando
- **Email**: entando-dev@entando.com

#### Local repository

- **Repo name**: http://localhost:8081/repository/npm-internal

### Configure NPM

#### Config npm to use the local repository
In order to use the private repository as default repository you need to configure npm accordingly (or use the `--registry=` option with all your commands)

```
npm config set registry http://localhost:8081/repository/<repo-name>/
```
> **Note A**: The trailing slash at the end of the repository is required for the repository to work

> **Note B**: This repository will be used for all the npm methods, so bare in mind that changing the global repository will potentially break other projects. If you want to avoid this continue to use the `--registry` option.

##### Login to the registry
You should be able now to login into the registry using the login command

```
npm login --registry=http://localhost:8081/repository/<repo-name>/
```

#### Good to go
You should now be able to publish your own npm modules to the private registry
using the `npm publish --registry=http://localhost:8081/repository/<repo-name>` command.

##### Set the publish repository at package.json level

In your npm module you can also add to the `package.json` an entry to 
make the private repository the default for publishing. Add this to your package.json file

```
  "publishConfig": {
    "registry": "http://localhost:8081/repository/<repo-name>/"
  }
```


## To review
(Automate nexus)[https://blog.mimacom.com/automate-nexus/]