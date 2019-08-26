ARG RUBY_VERSION
FROM ruby:$RUBY_VERSION

ARG BUNDLER_VERSION
ARG YARN_VERSION
ARG NODE_MAJOR
ARG PG_MAJOR

# Add NodeJS deb packages repos to the sources list
RUN curl -sL https://deb.nodesource.com/setup_$NODE_MAJOR.x | bash -

# Add Yarn deb packages repos to the sources list
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo 'deb http://dl.yarnpkg.com/debian/ stable main' > /etc/apt/sources.list.d/yarn.list

# Add PostgreSQL deb packages repos to the sources list
RUN curl -sSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
  && echo 'deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main' $PG_MAJOR > /etc/apt/sources.list.d/pgdg.list

# Install dependencies
RUN apt-get update -qq && DEBIAN_FRONTEND=noninteractive apt-get -yq dist-upgrade && \
  DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    build-essential \
    postgresql-client-$PG_MAJOR \
    nodejs \
    yarn=$YARN_VERSION-1 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    truncate -s 0 /var/log/*log

# Create and set home directory
RUN mkdir /app
WORKDIR /app

ENV LANG=C.UTF-8 \
  GEM_HOME=/bundle \
  BUNDLE_JOBS=4 \
  BUNDLE_RETRY=3
ENV BUNDLE_PATH $GEM_HOME
ENV BUNDLE_APP_CONFIG=$BUNDLE_PATH \
  BUNDLE_BIN=$BUNDLE_PATH/bin
ENV PATH /app/bin:$BUNDLE_BIN:$PATH

# Upgrade RubyGems, install bundler
RUN gem update --system && \
    gem install bundler:$BUNDLER_VERSION
