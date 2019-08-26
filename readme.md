### How to setup

Copy `docker` folder and `docker-compose.yml` file to the root folder of your rails app.

Our environment is configurable from the outside and **Dockerfile** is a sort of a template.
Thats why in **docker-compose.yml** we set arguments with exact versions of `ruby`, `node`, `postgres` etc.
Provide those arguments in the `args:` section of the `backend` service.

Set image name and version in your **docker-compose** file for [versioning](#image-versioning).
Default is `image: rails-app:1.0.0`


### How to use

To log into the containers shell run `docker-compose run --rm shell`. Used to run tests, migrations, rake tasks etc.

Run `docker-compose up rails` to start your rails server.

To check what services are running run `docker ps`.

#### Image versioning

Every time you change the **Dockerfile** content you must increase the version in the **docker-compose** file.

Proposed rule to follow:

- increase the patch version (1.0.x) if the bug was fixed;
- increase the minor version (1.x.0) if some dependencies were changed;
- increase the major version (x.0.0) if some major changes were provided (ex. app stack was changed);


### What is inside Dockerfile and docker-compose files

Several tricks (like use `:cached` folders, store assets/bundles in volumes, etc) are used to make our dockerized rails app be faster and more comfortable to use while developing.
Brief explanation of some of those tricks comes below.


in a **Dockerfile**:

At first we specify that we want to receive exact versions of `ruby`, `bundler`, `node`, `yarn`, `postgres` as arguments from the outside of our **Dockerfile**:

```bash
ARG RUBY_VERSION
ARG BUNDLER_VERSION
ARG YARN_VERSION
ARG NODE_MAJOR
ARG PG_MAJOR
```

We add required deb packages repositories to the sources list and then install the dependencies.
Each RUN instruction can be seen as a cacheable unit of execution.
Thats why we chain several commands to install those dependencies into one RUN instruction to form one cacheable unit.
To make sure our Docker layer doesn’t contain any garbage in the last part of this RUN we clear out the local repository of retrieved package files and all created during the installation temporary files and logs:
`apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && truncate -s 0 /var/log/*log`

`DEBIAN_FRONTEND=noninteractive` - this is the anti-frontend. It never interacts with you  at  all, and  makes  the  default  answers  be used for all questions. ([link to the manual](https://manpages.debian.org/stretch/debconf-doc/debconf.7.en.html))

Line `LANG=C.UTF-8` sets the default locale to UTF-8.
Otherwise Ruby uses US-ASCII for strings (no support for emojis).

We set the path for gem installations via `GEM_HOME=/bundle`, where `bundle` is a volume we set in our **docker-compose** file.
This allow us to store the dependencies on our local machine.

`ENV PATH /app/bin:$BUNDLE_BIN:$PATH` this line expose Ruby and our application binaries globally, which means we shouldn't prefix our commands with `bundle exec`.

in a **docker-compose.yml**:

[Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications.
With Compose, you use a YAML file to configure the application’s services.

**The key tricks** are: use `:cached` to mount source files and use volumes for generated content (assets, bundle, etc.).

We define 7 services:

- `backend` service define all shared logic for our rails application;
- `shell` service starts `/bin/bash` by default;
- `web` service runs rails server;
- `webpacker` service runs webpack server;
- `postgres` service starts the database;
- `redis` and `sidekiq` services allow us to use Sidekiq as a background job processor;


#### Resources

Docker [blog post](https://blog.docker.com/2019/07/intro-guide-to-dockerfile-best-practices/) with some **Dockerfile** best practices.

Great explanation of some tricks can be found in [the Evil Martians blog post](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development).
