---
title: User Docker Compose for A Rails App with Postgres and Sidekiq
categories:
- DevEnv
- Shell
tags:
- docker
- docker-compose
- rails
- postgres
- sidekiq
- redis
- images
- containers
description: User Docker Compose for A Rails App with Postgres and Sidekiq
date: 2019-02-24
author_profile: true
classes: wide
---

I mentioned `docker compose` to  manage multiple container docker applications in this post, and now I am going to use it to solve a problem I have: recently I've been trying to build different small experimental rails apps, those lifespan of those small apps are short, and I usually start and continue those apps on different machines, every time I start a new app in a new machine I need to set up the app, set up the database the their connection, and sometimes it might affect other parts of the setup for long term projects. So what will be convenient is to be able to start up the local dev environment quickly in a separate environment. Docker compose seems to be the perfect tool for this scenario. What I want to achieve is to have a docker application with rails and postgresql running and talking to each other, and add redis and sidekiq. There is an official tutorial [here](https://docs.docker.com/compose/rails/), this note is mainly following along, but also add sidekiq and add more explanations.

## Rails + Postgresql

The first step is to have the basic rails app running with database. I created a new directory on my local machine called *quickstart*. I will start with the rails app image by creating a `Dockerfile`:

```bash
FROM ruby:2.4
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN mkdir /myapp
WORKDIR /myapp # make /myapp the working directory so that the RUN command happens there
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp
# create and copy necessary files into the container

# Add a script to be executed every time the container starts.
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Start the main process.
CMD ["rails", "server", "-b", "0.0.0.0"]
```


Since we will copying the *Gemfile* and *Gemfile.lock* to the myapp directory in the container, let's create them in local machine first:

*Gemfile*

```bash
source 'https://rubygems.org'
gem 'rails', '~>5'
```

For now, this gemfile will only install rails, but once we run `rails new` it will be overwritten with all the rails app dependencies. And an empty *Gemfile.lock file with `touch Gemfile.lock`*

We also need to create a script `entrypoint.sh` and copy it to the `/usr/bin/` directory of the the container to fix a rails specific issue:

```bash
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```


This will be executed every time the container gets started, to fix the issue that prevents the server from restarting when a certain `[server.pid](http://server.pid)` file pre-exists.

We will use those files to build the rails app image to run the web application container.

For the postgres container, we will just use the latest postgres image to start the container.

At this point, if you want to get it working, you can start those two container separately and exposing the right ports to each other, but docker compose make it easier with a config file `docker-compose.yml`:

```bash
version: '3'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

A few thing worth noting are:

1. we use volumes to mount local machine directory in docker, this way, you don't have to rebuild docker image when you modify local code
2. the command in the web section will override the CMD in Dockerfile
3. the web service will happen after db service gets up and running
4. `-b` option binds rails to the specified IP, by default it's localhost, usually 127.0.0.1, if the web server is listening for connections on 127.0.0.1, it will only accepts requests coming from the same host, `0.0.0.0` means rails is listening on all interfaces.

Now we'll start the web service with `docker-compose run`, it runs a one time command against a service, it will start a new container with the service configuration, and the command you passed by `run` overrides the command defined in the service configuration:

    docker-compose run web rails new . --force --no-deps --database=postgresql

The `--force` flag will overwrite the existing Gemfile.

After the above command, `Gemfile` is updated, so we will need to build the image again with `docker-compose build`.

The last step is to connect rails with the postgresql database, we will need to change the `config/database.yml` file:

```bash
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres # the default postgres user
  password:
  pool: 5

development:
  <<: *default
  database: myapp_development


test:
  <<: *default
  database: myapp_test
```


If you run `docker-compose up` now you should see in the terminal `Listening on tcp://0.0.0.0:3000` , however if you go to `0.0.0.0:3000` it will throw an error "myapp_development" does not exist, that's because we haven't created the database, let's create the db in rails service, run `docker-compose run web rake db:create` in another tab. Depends on the use case, you can add `bundle exec rails db:migrate` into the docker compose configuration if you want to commit the database schema change into version control, but in my case, when I use this compose configuration I usually want to start from scratch.

Then refresh `0.0.0.0:3000` you should see the rails welcome page. You can stop the application with `docker-compose down`.

### When to rebuild

If you modify the Gemfile or the compose file, you need to rebuild `docker-compose up —-build`, or `docker-compose run web bundle install` to sync changes in Gemfile.lock, the rebuild.

## Add Sidekiq and Redis

1. Modify the Gemfile to add sidekiq gem:

    `gem 'sidekiq'`

2. Update Gemfile.lock:

    `docker-compose run web bundle install`

3. Modify `docker-compose.yml`, add redis, add sidekiq and run `bundle exec sidekiq` when starting the sidekiq container

```bash
version: '3'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
  redis:
    image: redis
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
  sidekiq:
    build: .
    command: bundle exec sidekiq
    depends_on:
      - redis
      - db
    volumes:
      - .:/myapp
```

4. Modify sidekiq and redis config in rails app
    - [create](https://github.com/mperham/sidekiq/wiki/Using-Redis#using-an-initializer) an `sidekiq.rb` initializer in `initializers` folder with:

      {% highlight bash %}
      Sidekiq.configure_server do |config|
        config.redis = { url: 'redis://redis:6379/0' }
      end

      Sidekiq.configure_client do |config|
        config.redis = { url: 'redis://redis:6379/0' }
      end
      {% endhighlight %}


        Note that by default sidekiq tried to connect to `127.0.0.1:6379`, but since sidekiq is now in a different container than redis, we should use `redis:6379` [as the redis host](https://github.com/mperham/sidekiq/wiki/Using-Redis).

    - Also change the active job queue config in environments folder, I added this line in development.rb since I am only using it for local development:

        `config.active_job.queue_adapter = :sidekiq`

    5. [Add a simple job](https://github.com/mperham/sidekiq/wiki/Getting-Started#rails) so that we can test if it's working:

    `rails g sidekiq:worker Hard`

    modify the sample worker file:

    {% highlight bash %}
    class HardWorker
      include Sidekiq::Worker
      def perform(name, count)
        # do something
        puts "test worker"
      end
    end
    {% endhighlight %}


    6. You can create a job within rails console `docker-compose run web rails c`, to add a job `HardWorker.perform_async('bob', 5)`

    7. rebuild and restart since the `docker-compose.yml` file has changed:

         `docker-compose up -—build`

    8. once all containers are up and running, you should also see the job has been processed by sidekiq, `test worker` is printed, and if you add a job again, it will be immediately process by sidekiq.
