# A self-contained rails development environment

# About

This setup lets you develop a rails application on any machine with Docker, [docker-compose](https://docs.docker.com/compose/install/) and Bash.
The technique is adaptable for other web stacks (think Node, PHP, ...).

# Setup

Clone the source, then

    bin/bootstrap
    bin/start development

You can now browse the app by visiting `http://localhost:3000/`

# Getting started / useful commands

| Command                                   | Purpose                                                                                  |
| -------                                   | -------                                                                                  |
| `bin/source_env development`              | Switch to the development environment                                                    |
| `bin/start development`                   | Start the development environment                                                        |
| `bin/stop development`                    | Stop the development environment                                                         |
| `bin/restart development`                 | Restart the development environment                                                      |
| `bin/test`                                | Run the tests                                         |
| `docker-compose ps`                       | Show the running containers                                                              |
| `docker-compose logs -f rails`            | Follow the rails log                                                                     |
| `docker-compose run rails rails c`        | Run the rails console in a new container (you can run all rails tasks like this)         |
| `docker-compose run rails bundle install` | Install the bundle                                                                       |
| `docker-compose exec rails bash`          | Start a bash inside the running rails container (you will see the rails process with ps) |
| `docker-compose run yarn yarn add moment` | Install a node module                                                                    |

# Concepts

## Environment

There is a *development* and a *testing* environment, implemented as separate sets of containers.

You can start and stop these environments. It's also possible to inspect each environment and perform 
commandline tasks inside the containers.

* To start an environment: `bin/start development` or `bin/start test`
* To stop an environment: `bin/stop development` or `bin/stop test`
* To point your docker-compose/Docker CLI at an environment: `. bin/source_env development` or `. bin/source_env test`

Have a look at what the scripts are doing, there's no rocket science involved.

## Images and file permissions

All files created by the containers will belong to you. This means you can do all editing and browsing with
your favorite tools that you install and configure locally.

To achieve correct ownership, most Docker images contain a user `app` whose id 
and primary group id are the same as your users'. Processes in those containers (such as `rails s`) will belong to `app`.
`app`, having the same uid/gid as you have, is effectively an alias of your user.

## Volumes

All state (runtime data managed by the containers) lives inside volumes. A volume is a directory under `volumes/`.

Volumes include

* Database files
* Gems, Node modules
* Separate history files for the shells of each container

Volumes need to have correct permissions set so both you and the containers can access them. 
This bootstrapping is done via `bin/bootstrap` -> `docker-compose.bootstrap.yml`

Having all state in volumes also means you can reliably recreate the situation on your device 
otherwhere by checking out the same commit on another device and copying your `volumes` 
directory there.

### Why not docker volumes?

Docker also has an [approach to volumes](https://docs.docker.com/storage/volumes/) - the implemented
alternative seems to facilitate inspecting volume contents, measuring volume size and cleaning up.
Maybe it would just be a matter of using docker volumes correctly...

# Caveats / further possibilities for improvement

* This setup prevents you from using spring. One approach to solving this would be to start a container 
  running spring for both environments, then rewire the scripts to run all rails commands in it,
  e.g. `docker-compose exec spring rails s`
* What will bundle open do? Can we extract the path to the relevant gem inside the container with bundler,
  then pass it locally to `xdg-open` to get the same behaviour?

# Big bang: Generating the rails app

This repo already contains a generated rails app. The following information is intended as an example
of how to start a project without ever installing the necessary dependencies locally.

To get things running without an existing app, do

    bin/bootstrap # this will fail when trying to install the gems
    . bin/source_env development
    
    docker-compose run rails bash
    
No you're in a bash shell inside a Ruby enabled container. On we go:

    gem install rails
    rails new . --database=postgresql --skip-git
    
Leave the shell (Ctrl + D)

Make sure the database connection infos are read from the environment variables (see [database.yml](src/config/database.yml)) and finish boostrapping:

Starting from there, you can use

    bin/start development
    
to start your environment. Have a look at the logs:

    . bin/source_env development
    docker-compose logs rails

Rails might complain about a missing Javascript engine. If so, add `therubyracer` to your Gemfile and 

    . bin/source_env development
    docker-compose run rails bundle install
    docker-compose run rails rails db:migrate
    bin/start development

As soon as the app boots, you can browse it by visiting `http://localhost:3000/`
