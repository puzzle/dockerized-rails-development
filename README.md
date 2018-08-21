# A self-contained rails development environment

# About

This setup lets you develop a rails application on any machine with Docker, [docker-compose](https://docs.docker.com/compose/install/) and Bash.
The technique is adaptable for other web stacks (think Node, PHP, ...).

# Setup

Clone the source, then

    bin/bootstrap
    bin/start development

You can now browse the app by visiting `http://localhost:3000/`

# Generating the rails app

This repo contains a generated rails app. To generate the app without having to install rails on your machine, do

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
