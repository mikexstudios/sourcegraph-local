FROM alpine:3.9

RUN apk --no-cache add git git-daemon

VOLUME /repos
WORKDIR /repos

EXPOSE 9418
# By default, only "upload-pack" is enabled, which makes this read-only.
CMD git daemon --reuseaddr --base-path=. --export-all --verbose