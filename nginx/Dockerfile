FROM nginx

RUN apt-get update -yqq && \
  apt-get install -yqq vim curl certbot python-certbot-nginx nginx-extras

# To run inside container, manually for now
# RUN certbot --server https://acme-v02.api.letsencrypt.org/directory
#	 -d www.rubyaumaroc.com -d rubyaumaroc.com -d *.masajid.world -d masajid.world \
#	 --manual --preferred-challenges dns-01 certonly
# RUN certbot \
#   --server https://acme-v02.api.letsencrypt.org/directory \
#   -d *.masajid.world -d masajid.world \
#   --manual \
#   --preferred-challenges dns-01 certonly

COPY nginx.conf /etc/nginx/nginx.conf
