FROM ruby:2.3-alpine

MAINTAINER jicki

ENV TZ 'Asia/Shanghai'

RUN apk upgrade --no-cache && \
    apk add --no-cache bash tzdata nodejs py-pygments git gcc musl-dev make && \
    git clone https://github.com/jicki/jicki.github.io.git /root/jicki.github.io && \
    cd /root/jicki.github.io && \
    gem install --no-document bundle && \
    bundle install && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    apk del gcc musl-dev make && \
    rm -rf /var/cache/apk/*

ADD cron/15min/* /etc/periodic/15min

ADD entrypoint.sh /root/entrypoint.sh

RUN chmod +x /root/entrypoint.sh

RUN chmod +x /etc/periodic/15min/*

WORKDIR /root

ENTRYPOINT ["/root/entrypoint.sh"]
