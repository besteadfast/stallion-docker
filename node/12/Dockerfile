FROM node:12.18-alpine

RUN set -ex \
	&& apk update \
	# Install dependencies
	&& apk add --no-cache --virtual .build-deps \
		python \
		make \
		g++

WORKDIR /app

CMD npm install \
	&& \
	npm run hot