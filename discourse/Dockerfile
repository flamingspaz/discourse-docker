# -*- sh -*-

# Base image for all Discourse related processes.

FROM ubuntu:12.04
MAINTAINER srid

ENV DISCOURSE_VERSION 0.9.7.8
ENV RAILS_ENV production

ENV LANGUAGE en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

ADD install /

RUN bash -xe /install


RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -q -y language-pack-en
RUN update-locale LANG=en_US.UTF-8

RUN echo mail.discourse.mozilla-community.org > /etc/hostname
ADD etc-hosts.txt /etc/hosts
RUN chown root:root /etc/hosts

RUN apt-get install -q -y vim nano libsasl2-modules

# Install Postfix.
RUN echo "postfix postfix/main_mailer_type string Internet site" > preseed.txt
RUN echo "postfix postfix/mailname string mail.discourse.mozilla-community.org" >> preseed.txt
# Use Mailbox format.
RUN debconf-set-selections preseed.txt
RUN DEBIAN_FRONTEND=noninteractive apt-get install -q -y postfix

# Set up the relay

ADD mandrill.txt /etc/postfix/sasl_passwd
RUN chown root:root /etc/postfix/sasl_passwd
RUN chmod 600 /etc/postfix/sasl_passwd

RUN postconf -e relayhost=[smtp.mandrillapp.com]
RUN postconf -e smtp_sasl_auth_enable=yes
RUN postconf -e smtp_sasl_password_maps=hash:/etc/postfix/sasl_passwd 
RUN postconf -e smtp_sasl_security_options=noanonymous
RUN postconf -e smtp_sasl_mechanism_filter=plain,login
RUN postmap /etc/postfix/sasl_passwd

ADD etc-aliases.txt /etc/aliases
RUN chown root:root /etc/aliases
RUN newaliases

expose 25

ADD	enter /
RUN	chmod +x /enter
WORKDIR   /discourse
ENTRYPOINT ["/enter"]
