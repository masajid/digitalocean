FROM nginx

RUN apt-get update -yqq && \
  apt-get install -yqq vim curl certbot python3-certbot-nginx nginx-extras

# To run inside container, manually for now
# RUN certbot certonly --nginx --agree-tos --no-eff-email --email admin@masajid.world -d masajid.world
# RUN certbot --server https://acme-v02.api.letsencrypt.org/directory \
#	 -d www.al-nour.de -d al-nour.com -d *.masajid.world -d masajid.world \
#	 --manual --preferred-challenges dns-01 certonly

# RUN certbot certonly \
#   --manual \
#   --preferred-challenges=dns \
#   --email masajid.world@gmail.com \
#   --server https://acme-v02.api.letsencrypt.org/directory  \
#   --agree-tos \
#   -d *.masajid.world -d masajid.world

# COPY nginx.conf /etc/nginx/nginx.conf
