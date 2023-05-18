FROM ruby:2.7.6-slim

# Install dependencies
RUN \
  apt-get update && \
  apt-get install -y --no-install-recommends \
    build-essential \
    default-libmysqlclient-dev \
    libpq-dev \
    libsqlite3-dev \
    wget \
    apt-transport-https \
    git \
    openssh-client \
    curl \
    gnupg2 \
    nodejs \
    npm && \
  rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://get.docker.com | bash -

WORKDIR /app

# Mostly static
COPY .ruby-version config.ru Rakefile .env.virtualbox ./
COPY bin bin
COPY public public
COPY db db
COPY .env.bootstrap .env
COPY .env.test .env.test

# NPM
COPY package.json ./
RUN npm install --silent >/dev/null

# Gems
COPY Gemfile Gemfile.lock ./
COPY plugins plugins
RUN bundle install --quiet --jobs 4

# Code
COPY config config
COPY app app
COPY lib lib

# Tests
COPY test test

# Assets
COPY vendor/assets vendor/assets

# # Drop privs to samson user as some tests expect non-root
# RUN groupadd -r samson && useradd -r -g samson -d /app samson \
#   && chown -R samson:samson /app

# USER samson

RUN ./bin/decode_dot_env .env
RUN PRECOMPILE=1 bundle exec rake assets:precompile

# # Copy source code
# COPY --chown=samson:samson . /app/
# COPY --chown=samson:samson .env.bootstrap /app/.env
