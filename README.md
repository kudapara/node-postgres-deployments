# Local Development

## Dev environment setup
To develop locally make sure you have the following installed
- [nodejs](https://nodejs.org)
- [vscode](https://code.visualstudio.com)
- [git](https://git-scm.com)
- [postgresql](https://postgresql.org)

## Git
To effectively manage and track changes to th code base you need to efficiently use git. Download this [git cheatsheet](https://www.atlassian.com/dam/jcr:e7e22f25-bba2-4ef1-a197-53f46b6df4a5/SWTM-2088_Atlassian-Git-Cheatsheet.pdf)

# Deployment

When the app has been completed locally it's now time to deploy it to your remote servers so that the app can be accessible to the whole world.

## Server setup
To get started you have to create a digitalocean/linode server. Make sure to add your computer's ssh keys so that you can remotely access it when perfoming the deploymnent.

- [what are ssh keys](https://jumpcloud.com/blog/what-are-ssh-keys)
- [generrating ssh keys on windows](https://phoenixnap.com/kb/generate-ssh-key-windows-10)
- [how to add ssh keys to linode](https://www.linode.com/docs/guides/use-public-key-authentication-with-ssh/)

## Connecting to server & deployment
Once you got the server up and running, now is the time to install the software that is required to run your application.

There are two ways to go about this.
1. Install every package by hand
2. Use a prebuild heroku-like deplyment server, that comes with some of those tools

Go with option 2. Install dockku on your droplet/vps.

To do that
-  Connect to your server via ssh. [Instructions here.](https://www.linode.com/docs/guides/getting-started/)
- Install [dokku](https://dokku.com)
```bash
  > wget https://raw.githubusercontent.com/dokku/dokku/v0.24.4/bootstrap.sh
  > sudo DOKKU_TAG=v0.24.4 bash bootstrap.sh

  # go to your server's IP and follow the web installer
```
- Follow [these instructions](https://github.com/amannn/dokku-node-hello-world) to get your node app up and running
- If you want to install postgres [follow this guide](https://github.com/dokku/dokku-postgres)
- To intstall postgres with postgis, fork [this repo](https://github.com/dokku/dokku-postgres) and replace the [POSTGRES_IMAGE name here](https://github.com/dokku/dokku-postgres/blob/1a0f6815ca2ccb4c0fe4fbf749ff3473dae09d83/config#L2) from `postgres` to `postgis/postgis` and the [POSTGRES_IMAGE_VERSION here](https://github.com/dokku/dokku-postgres/blob/1a0f6815ca2ccb4c0fe4fbf749ff3473dae09d83/config#L3) from `11.3` to `latest`. This has to be done in your forked repo.
- If you go with the option to install postgis there is no need to also install postgres because the postgis docker image is based on the [postgres](https://registry.hub.docker.com/_/postgres/) image and will install postgres automatically for you.
- Now when you install the postgres plugin on your VPS, instead of using the command
```bash
  sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
```
- Use the one from your repository
```sh
 sudo dokku plugin:install https://github.com/<your git username>/dokku-postgres.git postgres
```

### Migrations
These are some ways migrations are done and they both have pros and cons.
1. Dumping local database schema and importing it on the remote server
  - PRO: The easiest option to get a copy of the local database online, for the first deployment
  - CON: only valid for the first deployment, after that if the remote app is used the database deviates from the original and doing that process again will result in loss of data
2. Running sql scripts to add/remove tables/columns in the remote database everytime a new change has been made
  - PRO: You are interacting with the database directly so there is flexibility
  - CON: Manual process. Tedius and error prone.
  - CON: Not really possible to rollback if you mess up
3. Writing migrations for your database changes. Like git, database migrations track the changes in your database over time and can be run everytime the app is deployed and they ensure the remote database is consistent with the local database. For postgresql and nodejs check out [postgres-migrations](https://www.npmjs.com/package/postgres-migrations)
  - PRO: Code based. can be version controlled and reviewed before deployment
  - PRO: No need to write sql statements manually everytime you deploy
  - PRO: Consistency. Migrations are guaranteed to run in the same order everytime
  - CON: More code to maintain
