FROM nginx:1.23.2

RUN apt-get update -yqq && \
  apt-get install -yqq vim curl

# See https://certbot.eff.org/instructions?ws=nginx&os=pip
# install system dependencies
RUN apt-get install -yqq python3 python3-dev python3-venv libaugeas-dev gcc

# setup pyton env
RUN python3 -m venv /opt/certbot/
RUN /opt/certbot/bin/pip install --upgrade pip

# install certbot
RUN /opt/certbot/bin/pip install certbot certbot-nginx certbot-dns-digitalocean certbot-dns-namecheap

# prepare the certbot command
RUN ln -s /opt/certbot/bin/certbot /usr/bin/certbot

# copy credentials
COPY credentials/digitalocean.ini /etc/letsencrypt/digitalocean.ini
COPY credentials/namecheap.ini /etc/letsencrypt/namecheap.ini

RUN chmod 600 /etc/letsencrypt/*.ini


# install certificates

# run inside container, manually for now
# RUN certbot certonly --nginx --agree-tos --no-eff-email --email admin@masajid.world -d masajid.world
# RUN certbot --server https://acme-v02.api.letsencrypt.org/directory \
# 	 -d www.ikg-zeven.de -d ikg-zeven.de -d *.masajid.world -d masajid.world \
# 	 --manual --preferred-challenges dns-01 certonly

# run automatically
# RUN certbot certonly \
#   --dns-digitalocean \
#   --dns-digitalocean-credentials /etc/letsencrypt/digitalocean.ini \
#   --dns-digitalocean-propagation-seconds 1800 \
#   -d masajid.world -d '*.masajid.world' \
#   --agree-tos --no-eff-email --email masajid.world@gmail.com \
#   --non-interactive --expand

# run manually because namecheap tokens are paid
# RUN certbot certonly --manual --preferred-challenges=dns \
#   -d ikg-zeven.de -d www.ikg-zeven.de \
#   --agree-tos --no-eff-email --email masajid.world@gmail.com \
#   --non-interactive --expand

# run for different providers
# RUN certbot certonly \
#   --dns-digitalocean \
#   --dns-digitalocean-credentials /etc/letsencrypt/digitalocean.ini \
#   --dns-digitalocean-propagation-seconds 1800 \
#   --dns-namecheap \
#   --dns-namecheap-credentials /etc/letsencrypt/namecheap.ini \
#   --dns-namecheap-propagation-seconds 600 \
#   -d masajid.world -d '*.masajid.world' \
#   -d www.ikg-zeven.de -d ikg-zeven.de \
#   --agree-tos --no-eff-email --email masajid.world@gmail.com \
#   --non-interactive --expand


# automatic renewal
RUN echo "0 0,12 * * * root \
  /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' \
  && sudo certbot renew -q" \
  | sudo tee -a /etc/crontab > /dev/null

# check cronjob is running
RUN systemctl list-timers | grep certbot

# configure and run nginx
COPY nginx.conf /etc/nginx/nginx.conf

RUN nginx -t
RUN service nginx status
RUN service nginx reload
