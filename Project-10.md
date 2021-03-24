# Load Balancer Solution With Nginx and SSL/TLS
## Project  10

**Part 1 - Configure Nginx As A Load Balancer**
1. Create the Servers: Web Servers (2) and LB Server (1)
    - Creaate 2 EC2 Instances based on Red Hat Enterprise Linux 8 (HVM)
    - Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections)

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
    ```
    sudo vi /etc/hosts

    <WebServer1-Private-IP-Address> Web1
    <WebServer2-Private-IP-Address> Web2

    172.31.91.107 Web1
    172.31.86.49 Web2
    ```

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
    - Install Nginx
    ```
    sudo apt update
    sudo apt install nginx
    ```
    - Configure Nginx LB using Web Servers’ names defined in `/etc/hosts`
    ```
    sudo vi /etc/nginx/nginx.conf

    #insert following configuration into http section

    upstream myproject {
        server Web1 weight=5;
        server Web2 weight=5;
    }

    server {
        listen 80;
        server_name www.domain.com;
        location / {
            proxy_pass http://myproject;
        }
    }

    #comment out this line
    #       include /etc/nginx/sites-enabled/*;
    ```
    - Restart Nginx and make sure the service is up and running
    ```
    sudo systemctl restart nginx
    sudo systemctl status nginx
    ```

**Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates**

1. Register a new domain name with any registrar of your choice in any domain zone (e.g. `.com, .net, .org, .edu, .info, .xyz` or any other)
    <br> Use freenom.com for a free domain.

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

3. Update A record in your registrar to point to Nginx LB using Elastic IP address

4. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol - http://<your-domain-name.com>


5. Configure Nginx to recognize your new domain name
Update your `nginx.conf` with server_name www.<your-domain-name.com> instead of server_name www.domain.com

```
sudo vi /etc/nginx/nginx.conf

server_name www.lbproject.tk
```
6. Install certbot and request for an SSL/TLS certificate
    - Make sure snapd service is active and running
        ```
        sudo systemctl status snapd
        ```
    - Install certbot
        ```
        sudo snap install --classic certbot
        ```
    - Request your certificate
        ```
        sudo ln -s /snap/bin/certbot /usr/bin/certbot
        sudo certbot --nginx
        ```
7. Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

8. Set up periodical renewal of your SSL/TLS certificate
    - You can test renewal command in `dry-run` mode
    ```
    sudo certbot renew --dry-run
    ```
    - Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.
    <br> * Edit the crontab file <br>
            ```
            crontab -e
            ```
    <br> * Add following line <br>
            ```
            * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
            ```