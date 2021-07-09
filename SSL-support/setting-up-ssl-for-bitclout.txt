
Setting up SSL within Docker for a Bitclout node/service - ex: running on GCP
I'm using cloutini.com as the sample domain - replace with your own.

1. follow instructions to setup the Bitclout node. 
  https://github.com/bitclout/run
  Ensure it runs in docker and is available on port 80. This will require enabling http (and https) firewall access, plus setting up an external IP address. That's out of scope (for now)
  http://cloutini.com/


2. #get letsencrypt...
  cd bitclout/run/
  git clone https://github.com/wmnnd/nginx-certbot.git 
  # note that the nginx-certbot folder might already be there from the bitclout node setup
  cd nginx-certbot


3. # edit this file
  nano ./init-letsencrypt.sh
  # add email address
  email="cert@cloutini.com"

  # replace all example.org except the last one in app.conf - place these 3 in here - use spaces - no commas
    domains=(cert.cloutini.com cloutini.com www.cloutini.com api.cloutini.com)


4. # ** find all files with example.org and replace with your domain
  grep -r "example.org" ./

  # keep the last example.org for now
  nano ./data/nginx/app.conf
    server {
        ...
        listen 80;
        server_name cloutini.com;
        ...
    }
    server {
        ...
        listen 443 ssl;
        server_name cloutini.com;
        ...
        ssl_certificate /etc/letsencrypt/live/cloutini.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/cloutini.com/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
        ...
        location / {
            proxy_pass  http://example.org;
            ...
        }
    }
        

5. bring down nginx so that port 80 is available - there might be a different way to keep it going
  docker stop nginx # or: docker-compose down 


6. *** use this to run a generic web server! 
  # comment/replace this line in init-letsencrypt.sh
    #docker-compose up --force-recreate -d nginx
  # with this
    docker stop web; docker run -it --rm -d -p 80:80 --name web -v $PWD/data/certbot/www/:/usr/share/nginx/html nginx;


7. run init-letsencrypt to add the multiple certs
  ./init-letsencrypt.sh # respond with y to replace


8. edit and replace the last example.org -> cert.cloutini.com
  nano ./data/nginx/app.conf


9. bring nginx back up and check that the site is up - now on SSL
  docker start nginx # or: docker start $(docker ps -aq) # or: docker-compose up -d
  https://cloutini.com/

10. Add the cert bot stuff to the main docker file for bitclout - see docker-compose-sample.yml. 
  Recheck the site after re-running the build + deploy
  docker-compose build;    docker-compose down;    docker-compose up -d;
  https://cloutini.com/



                
