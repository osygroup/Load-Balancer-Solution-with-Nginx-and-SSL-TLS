# Project 3: Load Balancer Solution with Nginx and SSL/TLS

SSL stands for Secure Sockets Layer. It\'s the standard technology for
keeping an internet connection secure and safeguarding any sensitive
data that is being sent between two systems, preventing criminals from
reading and modifying any information transferred, including potential
personal details. The two systems can be a server and a client (for
example, a shopping website and browser) or server to server (for
example, an application with personal identifiable information or with
payroll information).

It does this by making sure that any data transferred between users and
sites, or between two systems remains impossible to read. It uses
encryption algorithms to scramble data in transit, preventing hackers
from reading it as it is sent over the connection. This information
could be anything sensitive or personal which can include credit card
numbers and other financial information, names and addresses.

TLS (Transport Layer Security) is just an updated, more secure, version
of SSL. Security certificates are still referred to as SSL because it is
a more commonly used term, but when buying SSL from a Certificate
Authority (DigiCert, Entrust, Comodo, GlobalSign etc.), the most up to
date TLS certificates is bought with the option of ECC, RSA or DSA
encryption.

An SSL certificate is installed on the server side but there are visual
cues on the browser which can tell users that they are protected by SSL.
Firstly, if SSL is present on the site, users will see https:// at the
start of the web address rather than the http:// (the extra \"s\" stand
for \"secure\"). Depending on what level of validation a certificate is
given to the business, a secure connection may be indicated by the
presence of a padlock icon or a green address bar signal. The details of
the certificate, including the issuing authority and the corporate name
of the website owner, can be viewed by clicking on the lock symbol.

SSL starts to work after the TCP connection is established, initiating
what is called an SSL handshake. The server sends its certificate to the
user along with a number of specifications (including which version of
SSL/TLS and which encryption methods to use, etc.).The user then checks
the validity of the certificate, and selects the highest level of
encryption that can be supported by both parties and starts a secure
session using these methods. There are a good number of sets of methods
available with various strengths - they are called cipher suites.

To guarantee the integrity and authenticity of all messages transferred,
SSL and TLS protocols also include an authentication process using
message authentication codes (MAC).

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image1.jpeg)

In this project we will enhance the Tooling Website solution by adding
an Nginx Load Balancer to distribute traffic between the web Servers and
setting up SSL/TLS.

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image2.png)

## Prerequisite
Completion of CI-with-Jenkins (Project 2)

## Step 1: Install Nginx and configure as a load balancer to web servers

Create an Ubuntu 20.04 virtual machine named *nginx-lb* and install the
Nginx Open Source edition on it.

*\$ sudo apt-get update*

*\$ sudo apt-get install nginx*

*\$ sudo systemctl enable nginx*

*\$ sudo systemctl status nginx*

To confirm that the Nginx server is installed, open the load balancer
server's public IP address in a web browser to view the Nginx Welcome
page:

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image3.png)

If the welcome page is not seen, check that a firewall is not blocking
the connection. Disable the firewall if enabled:

\$ sudo ufw status

\$ sudo ufw disable

Also check that port 80 is open in the Inbound port rules of the Network
Security Group of the VM.

To configure Nginx for load balancing, set up Nginx with instructions
for which type of connections to listen to and where to redirect them.
Create a new configuration file using a text editor, for example with
nano:

\$ sudo nano /etc/nginx/conf.d/load-balancer.conf

<script src="https://gist.github.com/osygroup/b6ea814d80490d56e85194624daf04b2.js"></script>


In the load-balancer.conf, define the following two segments, *upstream*
and *server*, as in the example below:

*\# Define which servers to include in the load balancing scheme.*

*\# It\'s best to use the servers\' private IPs for better performance
and security.*

*upstream backend {*

*server 10.0.0.6;*

*server 10.0.0.7;*

*server 10.0.0.8;*

*}*

*\# This server accepts all traffic to port 80 and passes it to the
upstream.*

*\# Notice that the upstream name and the proxy_pass need to match.*

*server {*

*listen 80;*

*location / {*

*proxy_pass http://backend;*

*}*

*}*

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image4.png)

This configuration defaults to round-robin as no load balancing method
is defined. There are other Nginx load balancing methods like Least
connections, IP hashing and Server weights.

Save the file and exit the editor.

Next, disable the default server configuration. Remove the default
symbolic link from the sites-enabled folder.

*\$ sudo rm /etc/nginx/sites-enabled/default*

Ensure the Nginx configuration structure is correct by running the
following command.

*\$ sudo nginx -t*

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image5.png)

If the configuration is OK, restart the Nginx service to apply the
changes.

*\$ sudo systemctl restart nginx*

Open three SSH/Putty consoles for the three web Servers and run
following command:

*\$ sudo tail -f /var/log/httpd/access_log*

To test the Nginx load balancing, open the load balancer server's public
IP address in a web browser to view the Tooling website. Then
continuously refresh the page and make sure that the servers each
receive HTTP GET requests from the load balancer -- every time the page
is refreshed, new records will appear in a server's log file according
to the order in the configuration (round-robin).

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image6.png)

## Step 2: Configure Secure Connection to the Load Balancer with Certbot
and Let's Encrypt

To install Let's Encrypt certificate, a client tool to facilitate the
process. Certbot client will be used in this project. However, there are
other client tools that can be used e.g. ACMEv2.

Certbot is an extensible client that fetches a security certificate from
Let's Encrypt Authority and lets you automate the validation and
configuration of the certificate for use by the webserver.

Note that Let's Encrypt does not issue certificates for bare IP
addresses, only domain names. A domain name needs to be registered from
a Domain Name Registrar in order to get a Let's Encrypt certificate.
Create a DNS record of type 'A' on the domain name that points to the
public IP address of the load balancer:

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image7.jpeg)

Snapd is needed to install Certbot. Ubuntu 20.04 already comes with
snapd pre-installed, so check that the snapd version is up to date.

*\$ sudo snap install core; sudo snap refresh core*

Install Certbot

*\$ sudo snap install \--classic certbot*

Execute the following instruction on the command line on the machine to
ensure that the certbot command can be run

*\$ sudo ln -s /snap/bin/certbot /usr/bin/certbot*

Edit the /etc/nginx/conf.d/load-balancer.conf file and replace \'*listen
80*\' with \'*server_name \<domain_name\>*\' as seen below:

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image8.jpeg)

Verify that the Nginx configuration structure is correct.

*\$ sudo nginx -t*

Use certbot command to initialize the fetching and configuration of
Let's Encrypt security certificate.

*\$ sudo certbot \--nginx -d \<domain_name.com\>*

Follow the interactive prompt and answer the questions as requested. If
everything is well configured, a congratulatory message is shown at the
end. Open port 443 (port number for https) in the Inbound port rules of the Network Security Group of the VM.

To confirm that the Nginx site is encrypted, visit the tooling
website with the domain name and observe the padlock symbol at the
beginning of the URL. This indicates that the site is secured using an
SSL/TLS encryption.

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image9.png)

Further tests can be carried out on the site by pasting the site url
into a [testing site](https://www.ssllabs.com/ssltest/).

## Step 3: Set up Automatic Renewal

By default, Let's Encrypt certificates are valid for 90 days. It is
recommended to renew the certificate before it expires since an expired
certificate will give users a safety warning when they try to visit the
website.

The Certbot packages come with a cron job or systemd timer that will
renew the certificates automatically before they expire. There is no
need to run Certbot again, unless to change a configuration. Run this
command to test automatic renewal for the certificates:

*\$ sudo certbot renew \--dry-run*

The above command will automatically check the currently installed
certificates and will attempt to renew them if they are less than 30
days away from the expiration date.

Nginx recommends setting up an automatic renewal cron job.

First, open the crontab configuration file for the current user:

*\$ crontab -e*

Depending on text editor of choice, choose the corresponding number:

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image10.png)

Add a cron job that runs the certbot command, which renews the
certificate if it detects the certificate will expire within 30 days.
Schedule it to run daily at a specified time (in this example, it does
so at 05:00 a.m.):

*0 5 \* \* \* /usr/bin/certbot renew --quiet*

![](https://github.com/osygroup/Images/blob/main/LB-and-TLS-with%20Nginx/image11.png)

The cron job should also include the \--quiet attribute, as in the
command above. This instructs certbot not to include any output after
performing the task.

The interval of this cronjob can always be changed.

## Conclusion

Google now advocates that HTTPS, or SSL, should be used everywhere on
the web. Since 2014, the search engine has been rewarding secured
websites with improved web rankings, another great reason for any site
to install SSL.

If it\'s simply a blog or a standard \'info only\' kind of site, HTTPS
can help to protect the security of sites, reducing the risk or
tampering and intruders injecting ads onto the page to break user
experience. Plus, it really can\'t hurt in terms of search engine
rankings.

SSL works across all devices, servers and operating systems. Just as
sites are created to work on all browsers (Chrome, Edge, Firefox, Safari
etc.), SSL/TLS from a reputable provider will also work in 99% of cases.
Unless users are accessing the site from very niche browsers, all the
big names will be covered.

## Credits

<https://upcloud.com/community/tutorials/configure-load-balancing-nginx/>

https://www.websecurity.digicert.com/security-topics/what-is-ssl-tls-https

<https://certbot.eff.org/lets-encrypt/snap-apache>

<https://phoenixnap.com/kb/letsencrypt-nginx>

<https://www.youtube.com/watch?v=SJJmoDZ3il8>

<https://www.youtube.com/watch?v=T4Df5_cojAs>
