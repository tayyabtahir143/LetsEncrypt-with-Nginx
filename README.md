# Virtual hosts with Nginx and LetsEncrypt

We will discuss 2 method to get SSL Certificate from LetsEncrypt.

http Challenge

Dns Challenge


  * For the HTTP Challenge, Certbot expects to handle the placement of a file at http://<YOUR_DOMAIN>/.well-known/acme-challenge/<TOKEN> on your web server.

  * Similarly, for the DNS Challenge, Certbot anticipates managing the addition of a TXT record to your DNS provider like cloudflare & namecheap, structured as _acme.<YOURDOMAIN> IN TXT record.



install the required packages:

    sudo dnf install certbot python3-certbot-nginx


create directories for each site:


    mkdir -p /usr/share/nginx/prod.aus.tayyabtahir.net/public_html
    mkdir -p /usr/share/nginx/stage.aus.tayyabtahir.net/public_html
    mkdir -p /usr/share/nginx/test.aus.tayyabtahir.net/public_html

create index files in site directories:


    echo "<h1> This is Production site.</h1>" > /usr/share/nginx/prod.aus.tayyabtahir.net/index.html
    echo "<h1>This is Stagging.</h1>" > /usr/share/nginx/stage.aus.tayyabtahir.net/index.html
    echo "<h1>This is Testing site.</h1>" > /usr/share/nginx/test.aus.tayyabtahir.net/index.html


create sites-available directory for nginx config file for each site:

    mkdir /etc/nginx/sites-available
    
    vim prod.conf
    server{
    	listen	80;
    	server_name	prod.aus.tayyabtahir.net;
    	location /{
    		root /usr/share/nginx/prod.aus.tayyabtahir.net/public_html;
    		index	index.html;
    		}
    }
    
    
    
    vim stage.conf
    server{
    	listen	80;
    	server_name	stage.aus.tayyabtahir.net;
    	location /{
    		root /usr/share/nginx/stage.aus.tayyabtahir.net/public_html;
    		index	index.html;
    		}
    }
    
    
    vim test.conf
    
    server{
    	listen	80;
    	server_name	test.aus.tayyabtahir.net;
    	location /{
    		root /usr/share/nginx/test.aus.tayyabtahir.net/public_html;
    		index	index.html;
    		}
    }
    
    
edit the default nginx.conf file and add the following line in httpd section:

    vim /etc/nginx/nginx.conf
    
    include /etc/nginx/sites-available/*.conf;

restart the nginx service:

    systemctl restart nginx

service should be started and site should be able to accessible with curl.
<pre>
[root@TayyabsFedora ~]# curl prod.aus.tayyabtahir.net
<h1>This is Production site.</h1>
[root@TayyabsFedora ~]# 
[root@TayyabsFedora ~]# curl stage.aus.tayyabtahir.net
<h1>This is Stagging.</h1>
[root@TayyabsFedora ~]# 
[root@TayyabsFedora ~]# curl test.aus.tayyabtahir.net
<h1>This is Testing site.</h1>
[root@TayyabsFedora ~]# 
</pre>

**Server should be accessible on port 80 and 443 publicly so that letsencrypt server can access the token on the server http://<YOUR_DOMAIN>/.well-known/acme-challenge/<TOKEN> .**

Run the certbot dry-run command to check if everything is configured fine:

    certbot certonly  --nginx  --dry-run --preferred-challenges=http
    
    [root@TayyabsFedora ~]# certbot certonly  --nginx  --dry-run --preferred-challenges=http
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    
    Which names would you like to activate HTTPS for?
    We recommend selecting either all domains, or all domains in a VirtualHost/server block.
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    1: prod.aus.tayyabtahir.net
    2: stage.aus.tayyabtahir.net
    3: test.aus.tayyabtahir.net
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    Select the appropriate numbers separated by commas and/or spaces, or leave input
    blank to select all options shown (Enter 'c' to cancel): 1
    Simulating a certificate request for prod.aus.tayyabtahir.net
    The dry run was successful.
    [root@TayyabsFedora ~]# 

As everything is configured well, dry-run is successful.


**Lets get the certificate from production server:**

    certbot run  --nginx --preferred-challenges=http


<pre>[root@TayyabsFedora ~]# certbot run  --nginx --preferred-challenges=http
Saving debug log to /var/log/letsencrypt/letsencrypt.log

Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: prod.aus.tayyabtahir.net
2: stage.aus.tayyabtahir.net
3: test.aus.tayyabtahir.net
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter &apos;c&apos; to cancel): 1
Requesting a certificate for prod.aus.tayyabtahir.net

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/prod.aus.tayyabtahir.net/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/prod.aus.tayyabtahir.net/privkey.pem
This certificate expires on 2024-07-24.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for prod.aus.tayyabtahir.net to /etc/nginx/sites-available/prod.conf
Congratulations! You have successfully enabled HTTPS on https://prod.aus.tayyabtahir.net

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let&apos;s Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[root@TayyabsFedora ~]# 
</pre>

certbot created the token in prod site document root to verify with letsencrypt servers as you can see the logs below.

<pre>server{rewrite ^(/<font color="#C01C28"><b>.well</b></font>-known/acme-challenge/.*) $1 break; # managed by Certbot
location = /<font color="#C01C28"><b>.well</b></font>-known/acme-challenge/nYrgL8mribCrLCp1wu23eq3WyY38d60-wvhNOCzvcOM{default_type text/plain;return 200 nYrgL8mribCrLCp1wu23eq3WyY38d60-wvhNOCzvcOM.dai-B21Fg83igmXSzG5EQ8iDcRzTAuN_MUzw8CPY0nU;} # managed by Certbot
          &quot;url&quot;: &quot;http://prod.aus.tayyabtahir.net/<font color="#C01C28"><b>.well</b></font>-known/acme-challenge/nYrgL8mribCrLCp1wu23eq3WyY38d60-wvhNOCzvcOM&quot;,
server{rewrite ^(/<font color="#C01C28"><b>.well</b></font>-known/acme-challenge/.*) $1 break; # managed by Certbot
location = /<font color="#C01C28"><b>.well</b></font>-known/acme-challenge/-My3HkADjLyX6_gdvAkqx1qqqcSMwSr4QTvHQf6h994{default_type text/plain;return 200 -My3HkADjLyX6_gdvAkqx1qqqcSMwSr4QTvHQf6h994.rTuh0avniw6GgfXPOukub8x6DHBz9y-CHeSdxYqOxzg;} # managed by Certbot
          &quot;url&quot;: &quot;http://prod.aus.tayyabtahir.net/<font color="#C01C28"><b>.well</b></font>-known/acme-challenge/-My3HkADjLyX6_gdvAkqx1qqqcSMwSr4QTvHQf6h994&quot;,
</pre>


certbot run is successful and ssl certificate has been received and saved in the local directory at /etc/letsencrypt/live/.
certbot nginx plugin also has configured the ssl certificate path and ssl configurations in nginx configuration file.


<pre>[root@TayyabsFedora ~]# cat /etc/nginx/sites-available/prod.conf 
server{
	server_name	prod.aus.tayyabtahir.net;
	location /{
		root /usr/share/nginx/prod.aus.tayyabtahir.net;
		index	index.html;
		}

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/prod.aus.tayyabtahir.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/prod.aus.tayyabtahir.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server{
    if ($host = prod.aus.tayyabtahir.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	listen	80;
	server_name	prod.aus.tayyabtahir.net;
    return 404; # managed by Certbot


}[root@TayyabsFedora ~]#</pre>

![Alt text](URL)

# Now lets get the wildcard certificate with dns challenge.
We required dns provider token to allow certbot to do add the temporary record in public dns.
As I am using cloudflare dns provider, so, I have generated the api token. we need to create a file and pass it to certbot while running the certbot command.

    vim .cloudflare.ini
    dns_cloudflare_api_token=safkhakd;fjahlsdfkhalkjfhalkjhfdljkas

set the following permissions on the file:

    chmod 600 .cloudflare.ini
  * With the DNS challenge, the "run" command isn't applicable; instead, you're limited to using the "certonly" command.



then run the following command to generate the wild card certificate:


    certbot certonly --dns-cloudflare --dns-cloudflare-credentials .cloudflare.ini   -d '*.aus.tayyabtahir.net'

<pre>[root@TayyabsFedora ~]# certbot certonly --dns-cloudflare --dns-cloudflare-credentials .cloudflare.ini   -d &apos;*.aus.tayyabtahir.net&apos;
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for *.aus.tayyabtahir.net

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/aus.tayyabtahir.net/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/aus.tayyabtahir.net/privkey.pem
This certificate expires on 2024-07-24.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let&apos;s Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[root@TayyabsFedora ~]# </pre>

wildcard ssl certificate generated, certificate and key are saved in  /etc/letsencrypt/live/ directory.


  * When opting for the DNS challenge to secure a wild-card SSL certificate, you'll have to configure Apache or Nginx settings manually post obtaining the certificates.
  * So we need to create the modify the .conf files of every site and we need to mentioned ssl certificates path manually there.

Note: we can copy the configuration from prod.conf file which is already configured by nginx module while getting the certificate with http challenge.

<pre>[root@TayyabsFedora ~]# cat /etc/nginx/sites-available/prod.conf 
server{
	server_name	prod.aus.tayyabtahir.net;
	location /{
		root /usr/share/nginx/prod.aus.tayyabtahir.net/public_html;
		index	index.html;
		}

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/aus.tayyabtahir.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/aus.tayyabtahir.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server{
    if ($host = prod.aus.tayyabtahir.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	listen	80;
	server_name	prod.aus.tayyabtahir.net;
    return 404; # managed by Certbot


}
</pre>
<pre>
[root@TayyabsFedora ~]# cat /etc/nginx/sites-available/stage.conf 
server{
	server_name	stage.aus.tayyabtahir.net;
	location /{
		root /usr/share/nginx/stage.aus.tayyabtahir.net/public_html;
		index	index.html;
		}

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/aus.tayyabtahir.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/aus.tayyabtahir.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server{
    if ($host = stage.aus.tayyabtahir.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	listen	80;
	server_name	stage.aus.tayyabtahir.net;
    return 404; # managed by Certbot


}
</pre>
<pre>
[root@TayyabsFedora ~]# cat /etc/nginx/sites-available/test.conf 
server{
	server_name	test.aus.tayyabtahir.net;
	location /{
		root /usr/share/nginx/test.aus.tayyabtahir.net/public_html;
		index	index.html;
		}

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/aus.tayyabtahir.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/aus.tayyabtahir.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server{
    if ($host = test.aus.tayyabtahir.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	listen	80;
	server_name	test.aus.tayyabtahir.net;
    return 404; # managed by Certbot


}
[root@TayyabsFedora ~]# 

</pre>



lets access the site on the web and verify the certificate.

