# docker-cartodb
This is a dockerized production Carto(DB) installation, based on the official [Carto Installation Documentation](http://cartodb.readthedocs.io/en/latest/install.html), using the latest dependencies including ogr2ogr2.1.  

You should be able to map most of the steps in the `Dockerfile` with those on the install documentation. The steps below were written with a Digital Ocean droplet (docker pre-installed) in mind.

![dockerizecarto](https://cloud.githubusercontent.com/assets/1833820/22806442/1703bcac-eef0-11e6-8826-33a126932d23.png)


##Production vs Development
This Dockerfile assumes you want to run Carto with SSL, which requires a few configuration steps beyond just building the docker container.  The biggest difference from other Dockerfiles is setting the `production` environment variables, and then adding ssl certificates to the nginx configuration.

This build also makes use of Docker volumes, so that the the data can persist.  This should allow for an easier upgrade path when new versions of the core components are release.

##How to Build

###Get yourself a Droplet
We have had success installing Carto in a Docker container on a Digital Ocean droplet with 8GB of Ram, 80GB Storage and 4 cores, which runs $80/month.  You can choose an Ubuntu 16.04 image with Docker preinstalled and save yourself some time.  SSH into that bad boy and get ready to Docker.

###Edit the sample app_config.yml
You will need a FQDN with an A record pointing to your server's IP address.  You need to edit `config/app_config.sample.yml`, and replace every instance of 'yourdomain.com' with your domain name and save the file as `app_config.yml`.  The dockerfile will insert this into the new container during the build process. 


###Build the Container
Go make a sandwich and watch some youtube videos, this will take a while... (About an hour as of the time I wrote this)
```
git clone https://github.com/chriswhong/docker-cartodb.git
sudo docker build -t="cartodb" docker-cartodb/
```
###Run the Container
This command runs the container with the three main services mapped to the host machine.  `3000` for the frontend, `8080` for the Windshaft Map Tiler (Maps API), and `8181` for the SQL API.
Use `docker run -d -p 3000:3000 -p 8080:8080 -p 8181:8181 cartodb` the first time you run it.

If you are replacing an existing container built with this image, add the `--volumes from` flag like `docker run --volumes-from {id of old container} -d -p 3001:3000 -p 8081:8080 -p 8182:8181 cartodb`  

You can't connect from the outside world until you setup nginx to forward specific URLs to the three ports

###Configure Nginx

Install nginx `apt-get install nginx` and then copy `config/cartodb.nginx.proxy.conf` from this repo to `/etc/nginx/conf.d/`. 

Use `certbot-auto` to quickly install SSL certificates for your domain.  Take note of where it tells you it stored the new SSL certificates.  http://letsencrypt.readthedocs.io/en/latest/install.html#certbot-auto

Edit the file to include your production domain name and the paths to the SSL certificates.

Restart nginx `service nginx restart`

###Users
The Dockerfile will create a user named `admin` with a default password of `pass1234`.  You should change this when you first login.

###To enable GZIP
This can also be done in nginx, you just need to enable gzip and add the mime types that carto uses.  

###To use carto-served vector tiles
This is unrelated to carto in Docker, but It's exciting so I thought I would share it.  [See this gist](http://bl.ocks.org/chriswhong/2695b75fd1936bd034df83c91738648d) 

##Attribution
First and foremost, thank you to the Carto team for such a great product that is also open source!
This Dockerfile was inspired by and largely based on [Stefan Verhoeven's image](https://github.com/sverhoeven/docker-cartodb). I could not get it working and descided to start over using the official installation docs.  All of the configs are borrowed directly from his. 

##Pull requests and issues welcomed!
If this doesn't work for you, please open an issue and let's figure out why.
