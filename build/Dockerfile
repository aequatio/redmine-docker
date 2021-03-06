FROM ruby:2.3-alpine

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN addgroup -S redmine && adduser -S -g redmine -H -D -s /bin/false redmine

RUN apk add --no-cache \
		openssl \
		imagemagick \
		mariadb-client-libs \
		libpq \
		sqlite-libs \
		bash \
		tzdata \
		\
		bzr \
		git \
		mercurial \
		openssh-client \
		subversion

#	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys 6380DC428747F6C393FEACA59A84159D7001A4E5 \
#	&& gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini \
# grab tini for signal processing and zombie killing
ENV TINI_VERSION v0.13.2
RUN set -x \
	&& apk add --no-cache --virtual .gnupg gnupg \
	&& wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini-static-amd64" \
	&& wget -O /usr/local/bin/tini.asc "https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini-static-amd64.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& apk del .gnupg \
	&& rm -r "$GNUPGHOME" /usr/local/bin/tini.asc \
	&& chmod +x /usr/local/bin/tini \
	&& tini -h

#	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
#	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.10
RUN set -x \
	&& apk add --no-cache --virtual .gnupg gnupg \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& apk del .gnupg \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true

ENV RAILS_ENV production
# Set RubyGems mirror site if official site is too slow

ENV REDMINE_VERSION 3.3.1
ENV REDMINE_DOWNLOAD_MD5 cb8aab3e03cae7d21d003a307e51c176

#COPY ./redmine2 .
#RUN rm files/delete.me log/delete.me \
#    && mkdir -p tmp/pdf public/plugin_assets \
#    && chown -R redmine:redmine ./

RUN cd /usr/src \
    && git clone https://github.com/aequatio/redmine.git \
    && cd redmine \
    && rm -rf .git \
    && mkdir -p tmp/pdf public/plugin_assets \
    && chown -R redmine:redmine ./ \
    && ls

WORKDIR /usr/src/redmine

RUN echo "<<<<<<<<<<<<<<<<<<<<<< %PWD >>>>>>>>>>>>>>>>>>>>>>"; \
    pwd; \
    ls

RUN buildDeps=' \
		gcc \
		imagemagick-dev \
		libc-dev \
		zlib-dev \
		mariadb-dev \
		postgresql-dev \
		sqlite-dev \
		make \
		patch \
	' \
	&& set -ex \
	&& apk add --no-cache --virtual .build-deps $buildDeps \
	&& bundle install --without development test \
	&& for adapter in mysql2 postgresql sqlite3; do \
		echo "$RAILS_ENV:" > ./config/database.yml; \
		echo "  adapter: $adapter" >> ./config/database.yml; \
		bundle install --without development test; \
	done \
	&& rm -r ./config/database.yml \
	&& apk del .build-deps
#	"${HOME}/.bundle" "${HOME}/.gem" \


VOLUME /usr/src/redmine/files

COPY ./docker-entrypoint.sh /
RUN chmod a+x /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]

