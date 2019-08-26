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


#### Resources

Docker [blog post](https://blog.docker.com/2019/07/intro-guide-to-dockerfile-best-practices/) with some **Dockerfile** best practices.

Great explanation of some tricks can be found in [the Evil Martians blog post](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development).
