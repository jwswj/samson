FROM ruby:2.7.6-slim

# Install dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      build-essential \
      default-libmysqlclient-dev \
      libpq-dev \
      libsqlite3-dev \
      wget \
      apt-transport-https \
      git \
      curl \
      gnupg2 \
  && curl -sL https://deb.nodesource.com/setup_6.x | bash - \
  && apt-get install nodejs -y \
  && curl -fsSL https://get.docker.com | bash - \
  && wget -qc https://github.com/betalo-sweden/await/releases/download/v0.4.0/await-linux-amd64 \
  && install await-linux-amd64 /usr/local/bin/await \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Drop privs to samson user as some tests expect non-root
RUN groupadd -r samson && useradd -r -g samson -d /app samson \
  && chown -R samson:samson /app

USER samson

# Copy in npm deps
COPY package.json /app/package.json
RUN npm install --silent

# Copy in ruby deps
COPY .ruby-version Gemfile Gemfile.lock /app/
COPY plugins /app/plugins
RUN bundle install --jobs 8

# Copy source code
COPY --chown=samson:samson . /app/
COPY --chown=samson:samson .env.bootstrap /app/.env
RUN ./bin/decode_dot_env .env

ARG RAILS_ENV=development
RUN bundle exec rake assets:precompile
