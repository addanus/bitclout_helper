
# Introduction

This document is intended to show you how to setup SSL in Docker for a BitClout a node. Example: Running on the Google Cloud Platform (GCP.)

I'm using cloutini.com as the sample domain. Replace with your own domain any instances of cloutini.com in configuration files as needed.

## Steps

1. Setup BitClout Node

    - Follow the instructions described in <https://github.com/bitclout/run>
    - Ensure the node runs in Docker and the frontend is available on port 80. This will require enabling http (and https) firewall access, plus setting up an external IP address. That's out of scope (for now)
    - Sample application available at <http://cloutini.com/>

2. Get Let's Encrypt

      `cd bitclout/run/`

      `git clone https://github.com/wmnnd/nginx-certbot.git`

      `cd nginx-certbot`

    > **_NOTE:_**  The nginx-certbot folder might already be there from the bitclout node setup.

3. Configure Let's Encrypt

    - Open script file for edit

      `nano ./init-letsencrypt.sh`

    - Add email address

      `email="youremail@cloutini.com"`

    - Replace `example.org` with your own domain

    > **_NOTE:_** List of domains should be separated using spaces, and no commas, as in this example: `domains=(cloutini.com www.cloutini.com api.cloutini.com)`.

4. Configure `app.conf`

    - Find all files that contain `example.org`
  
      `grep -r "example.org" ./`

    - Open conf file for edit

      `nano ./data/nginx/app.conf`

    - Replace `example.org` with your own domain, except the last `proxy_pass example.org` for now (proly)

    - See [app.conf](app.conf) for reference.

    > **_NOTE:_** The reference [app.conf](app.conf) file has two server sections, one to listen on port 80 and one for port 433.

5. Stop the nginx web server, so that port 80 becomes available
  
    `docker stop nginx` or `../docker-compose -f docker-compose.dev.yml down`

6. Run a generic web server

    ```yaml
    # comment/replace this line in init-letsencrypt.sh    
    # docker-compose up --force-recreate -d nginx    
    
    # replace it with this    
      docker stop web; docker run -it --rm -d -p 80:80 --name web -v $PWD/data/certbot/www/:/usr/share/nginx/html nginx;
    ```

7. Create the multiple certs

    - Run `init-letsencrypt.sh`

      `./init-letsencrypt.sh # respond with y to replace if asked`

8. Finish `app.conf` configuration
  
    - Open conf file for edit

      `nano ./data/nginx/app.conf`

    - Edit and replace the last `example.org` with <http://frontend:8080>;

    - The final file should look like [app.conf](app.conf)

9. Configure `docker-compose-sample.yml`

    - Open docker compose file for edit

      `cd ..`

      `sudo nano docker-compose.dev.yml`

    - Add the nginx-certbot code

    - See [docker-compose-sample.yml](docker-compose-sample.yml) for reference.

    > **_NOTE:_** The reference [docker-compose-sample.yml](docker-compose-sample.yml) file uses a custom Docker network. Follow the comments in the file regarding that.

10. Bring services up

      `docker-compose -f docker-compose.dev.yml build`

      `docker-compose -f docker-compose.dev.yml down`

      `docker-compose -f docker-compose.dev.yml up -d`

11. Verify that the site is up and running on SSL

    - <https://cloutini.com/>

> **_NOTE:_** Bonus: Your turn. Cloutini actually has a unique config and there wasn't enough time to properly check all the steps above from end to end. So be sure to submit a pull request or reach out on Bitclout [Addanus](https://bitclout.com/u/Addanus) or [Discord](https://discord.com/invite/bitclout)
