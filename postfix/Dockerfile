FROM ubuntu:12.04

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list # 20103-08-24
RUN apt-get update
RUN apt-get install -q -y language-pack-en
RUN update-locale LANG=en_US.UTF-8

RUN echo mail.discourse.mozilla-community.org > /etc/hostname
add etc-hosts.txt /etc/hosts
RUN chown root:root /etc/hosts

RUN apt-get install -q -y vim nano libsasl2-modules

# Install Postfix.
RUN echo "postfix postfix/main_mailer_type string Internet site" > preseed.txt
RUN echo "postfix postfix/mailname string mail.discourse.mozilla-community.org" >> preseed.txt
# Use Mailbox format.
RUN debconf-set-selections preseed.txt
RUN DEBIAN_FRONTEND=noninteractive apt-get install -q -y postfix

# Set up the relay

add mandrill.txt /etc/postfix/sasl_passwd
RUN chown root:root /etc/postfix/sasl_passwd
RUN chmod 600 /etc/postfix/sasl_passwd

RUN postconf -e relayhost=[smtp.mandrillapp.com]
RUN postconf -e smtp_sasl_auth_enable=yes
RUN postconf -e smtp_sasl_password_maps=hash:/etc/postfix/sasl_passwd 
RUN postconf -e smtp_sasl_security_options=noanonymous
RUN postconf -e smtp_sasl_mechanism_filter=plain,login
RUN postmap /etc/postfix/sasl_passwd

add etc-aliases.txt /etc/aliases
RUN chown root:root /etc/aliases
RUN newaliases

# Use syslog-ng to get Postfix logs (rsyslog uses upstart which does not seem
# to RUN within Docker).
RUN apt-get install -q -y syslog-ng

expose 25
cmd ["sh", "-c", "service syslog-ng start ; service postfix start ; tail -F /var/log/mail.log"]