# Load Balancer Solution With Nginx and SSL/TLS
## Project  10

**Part 1 - Configure Nginx As A Load Balancer**
1. Create the Servers: Web Servers (2) and LB Server (1)
    - Creaate 2 EC2 Instances based on Red Hat Enterprise Linux 8 (HVM)
    ![image](https://user-images.githubusercontent.com/53876750/112306142-c94eb780-8c9f-11eb-96b2-1d9902f463a4.png)
    - Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections)
    ![image](https://user-images.githubusercontent.com/53876750/112306222-df5c7800-8c9f-11eb-90f5-3e77ac34473b.png) <br>
    ![image](https://user-images.githubusercontent.com/53876750/112306243-e71c1c80-8c9f-11eb-9042-d2d6a3d4e8e6.png) 


2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
    ```
    sudo vi /etc/hosts

    <WebServer1-Private-IP-Address> Web1
    <WebServer2-Private-IP-Address> Web2

    172.31.91.107 Web1
    172.31.86.49 Web2
    ```
    ![image](https://user-images.githubusercontent.com/53876750/112306350-0155fa80-8ca0-11eb-8ad8-9b40d4de8790.png)

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
    - Install Nginx
    ```
    sudo apt update
    sudo apt install nginx
    ```
    ![image](https://user-images.githubusercontent.com/53876750/112306493-25194080-8ca0-11eb-9f74-25e8932a130c.png)

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
    ![image](https://user-images.githubusercontent.com/53876750/112306646-5265ee80-8ca0-11eb-9dc1-db78ccad9c4d.png)

    - Restart Nginx and make sure the service is up and running
    ```
    sudo systemctl restart nginx
    sudo systemctl status nginx
    ```
    ![image](https://user-images.githubusercontent.com/53876750/112306693-5d208380-8ca0-11eb-9b81-72f380b5ff9a.png)
    
    * Checking Web Servers 1 and 2 individually (using the Public IP Address):
    ![image](https://user-images.githubusercontent.com/53876750/112307257-04051f80-8ca1-11eb-81fe-39fc1c8b9bc3.png) <br>
    ![image](https://user-images.githubusercontent.com/53876750/112307289-0ff0e180-8ca1-11eb-9ed6-f456e8395d33.png)
    
    * Checking Web Servers 1 and 2 through the NGINX Load Balancer (using the Public IP Address):
    ![image](https://user-images.githubusercontent.com/53876750/112307514-55adaa00-8ca1-11eb-961d-79dddb0cba9e.png) <br>
    ![image](https://user-images.githubusercontent.com/53876750/112307545-5fcfa880-8ca1-11eb-87fa-caf9d548de64.png)



**Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates**

1. Register a new domain name with any registrar of your choice in any domain zone (e.g. `.com, .net, .org, .edu, .info, .xyz` or any other)
    <br> Use freenom.com for a free domain.
    ![image](https://user-images.githubusercontent.com/53876750/112307831-b5a45080-8ca1-11eb-80fb-032588b461a7.png)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
    ![image](https://user-images.githubusercontent.com/53876750/112307899-cc4aa780-8ca1-11eb-9571-b00f42bb5ca0.png)
    <br>
    ![image](https://user-images.githubusercontent.com/53876750/112307941-d8366980-8ca1-11eb-98bc-dc4a8bfc34f9.png)

3. Update A record in your registrar to point to Nginx LB using Elastic IP address
    ![image](https://user-images.githubusercontent.com/53876750/112308029-f13f1a80-8ca1-11eb-97c7-318164c43aa8.png)

4. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol - http://<your-domain-name.com>
    <br> http://lbproject.tk
    ![image](https://user-images.githubusercontent.com/53876750/112308238-37947980-8ca2-11eb-88ba-6d23ceb80ab6.png)
    <br>
    ![image](https://user-images.githubusercontent.com/53876750/112308293-4713c280-8ca2-11eb-9757-da6531031bc5.png)


5. Configure Nginx to recognize your new domain name
Update your `nginx.conf` with server_name www.<your-domain-name.com> instead of server_name www.domain.com

```
sudo vi /etc/nginx/nginx.conf

server_name www.lbproject.tk
```
![image](https://user-images.githubusercontent.com/53876750/112308636-a4a80f00-8ca2-11eb-8191-effc44f1613e.png)

6. Install certbot and request for an SSL/TLS certificate
    - Make sure snapd service is active and running
        ```
        sudo systemctl status snapd
        ```
        ![image](https://user-images.githubusercontent.com/53876750/112308715-bd182980-8ca2-11eb-9b91-ee6f9e649cbe.png)

    - Install certbot
        ```
        sudo snap install --classic certbot
        ```
        ![image](https://user-images.githubusercontent.com/53876750/112308812-d7ea9e00-8ca2-11eb-8551-38cdfc0d25b4.png)

    - Request your certificate
        ```
        sudo ln -s /snap/bin/certbot /usr/bin/certbot
        sudo certbot --nginx
        ```
        ![image](https://user-images.githubusercontent.com/53876750/112308859-e3d66000-8ca2-11eb-935f-4f263aa49349.png)
        <br>
        ![image](https://user-images.githubusercontent.com/53876750/112308900-f51f6c80-8ca2-11eb-9ae1-e90d32621406.png)

7. Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>
    <br> https://lbproject.tk
    ![image](https://user-images.githubusercontent.com/53876750/112309053-27c96500-8ca3-11eb-8e59-b607a7a7c30a.png)
    <br>
    ![image](https://user-images.githubusercontent.com/53876750/112309084-33b52700-8ca3-11eb-8175-2364a80f310f.png)

8. Set up periodical renewal of your SSL/TLS certificate
    - You can test renewal command in `dry-run` mode
    ```
    sudo certbot renew --dry-run
    ```
    ![image](https://user-images.githubusercontent.com/53876750/112309214-5e9f7b00-8ca3-11eb-8487-2ac72422a3b5.png)

    - Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.
    <br> * Edit the crontab file <br>
            ```
            crontab -e
            ```
            ![image](https://user-images.githubusercontent.com/53876750/112309253-6c550080-8ca3-11eb-9f00-a7786b29b4e1.png)

    <br> * Add following line <br>
            ```
            * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
            ```
            ![image](https://user-images.githubusercontent.com/53876750/112309345-85f64800-8ca3-11eb-9df0-5df18939e971.png)
