# Wildcard certificate
`*.vice.cyverse.at`

1. create a wildcard DNS for `vice-haproxy.cyverse.at` like `*.vice.cyverse.at` & `vice.cyverse.at`

![Alt text](cname.png?)


## STEPS
```bash
# install snap
yum install epel-release
yum install snapd
systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap

# Ensure that your version of snapd is up to date
sudo snap install core; sudo snap refresh core

# install certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo snap set certbot trust-plugin-with-root=ok

# install plugin
sudo snap install certbot-dns-digitalocean

# Set up credentials
mkdir certbo
cd certbo/
vi dg.ini
# cat dg.ini
# dns_digitalocean_token = 442b8d1c1e2a230b6879f3b99233cd844ea5bdea48975a8339808405546d33f2


# get wildcard cert
certbot certonly \
  --dns-digitalocean \
  --dns-digitalocean-credentials dg.ini \
  -d vice.cyverse.at \
  -d '*.vice.cyverse.at'

# combine the certs 
cat /etc/letsencrypt/live/vice.cyverse.at/fullchain.pem /etc/letsencrypt/live/vice.cyverse.at/privkey.pem > /etc/ssl/certs/update.vice.cyverse.at


# update haproxy.cfg
bind *:443 ssl crt /etc/ssl/certs/update.vice.cyverse.at

```

## manually renewing 

```bash
# copy the dg.ini file to the letsencript directory
cp dg.ini /etc/letsencrypt/live/

# one time command to renew certificates
sudo certbot renew

cat /etc/letsencrypt/live/vice.cyverse.at/fullchain.pem /etc/letsencrypt/live/vice.cyverse.at/privkey.pem > /etc/ssl/certs/update.vice.cyverse.at
```
