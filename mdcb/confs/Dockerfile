FROM debian:buster-slim

ENV TYKVERSION 1.7.5
ENV os=debian
ENV dist=jessie
ARG REPOKEY

LABEL Description="Tyk MDCB docker image" Vendor="Tyk" Version=$TYKVERSION

RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y --no-install-recommends \
            curl ca-certificates apt-transport-https gnupg \
 && curl https://packagecloud.io/gpg.key | apt-key add - \
 && apt-get purge -y gnupg \
 && apt-get autoremove -y \
 && rm -rf /root/.cache

RUN curl -s https://$REPOKEY:@packagecloud.io/install/repositories/tyk/tyk-mdcb/script.deb.sh | bash \
 && apt-get update \
 && apt-get install --allow-unauthenticated -f --force-yes -y tyk-sink=$TYKVERSION \
 && rm -rf /var/lib/apt/lists/*

COPY ./confs/sink.conf /opt/tyk-sink/tyk_sink.conf

WORKDIR /opt/tyk-sink

CMD ["/opt/tyk-sink/tyk-sink", "--c=/opt/tyk-sink/tyk_sink.conf"]
