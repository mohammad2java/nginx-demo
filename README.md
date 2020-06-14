# Definiton
	nginx is modernEra web server,vhost,reverse proxy,load balancer with highest performance.
	
#Topic to be cover
----------------
	1. installation guide
	2. directory structure
	3. command to start/stop/reload/
	4. command to compile conf file
	5. vhosts concept (block) ( mutliple site in single severs)
	6. reverse proxy concept ( hiding main site with domain)
	7. load balacing concept ( to improve performance using upstream directive)

#installation guide
---------------------

#1) if OS is OpenSuse Linux 
------------------------------	
	1) zypper seach nginx      (for searching)
	2) sudo zypper install nginx	(for installing)
	3) which nginx or sudo which nginx	(for verifying)
	root conf file diectory is : /etc/nginx  and root conf file is - /etc/nginx/nginx.conf
	/etc/nginx  contain following important dir
	conf.d/   	( almost with evry os)  (basically contain conf file with .conf extension)
	vhost.d/  	( with some OS like opensuse) (basically contain conf file with .conf extension)
	sites-available/ ( with some OS like ubuntu) 	( basically contain conf file without extension)
	sites-enabled/   ( with some OS like ubuntu)  	( basically contain softlink of sites-available files)
		
	Notes: if you open /etc/nginx/nginx.conf you will association with above dir files
 

	contain following 2 main block
	1) events
	example:
	events {
	 worker_connections  1024;
	use epoll;
	}
				
	
	2) http	
	example:
	http {
		    include       mime.types;
		    default_type  application/octet-stream;
		
		    keepalive_timeout  65;
		
		    #gzip  on;
		
		    include conf.d/*.conf;
		
		    server {
		      	listen       80;
		       server_name  localhost;
		       access_log  /var/log/nginx/host.access.log  main;
		       
			   location / {
		            root   /srv/www/htdocs/;
		            index  index.html index.htm;
		           }
		
				location /proxy {           
					proxy_pass https://www.digitalocean.com/;
		        }
		       
		
			error_page  404              /404.html;
		        error_page   500 502 503 504  /50x.html;
		        location = /50x.html {
		            root   /srv/www/htdocs/;
		        }
		
		    }
		
		    include vhosts.d/*.conf;
		}
	
		Notes:  inside http block you will see include directive or statements that are responsible to call other configuration.
				  like  include vhosts.d/*.conf;  include conf.d/*.conf;
			default root dir for html content: /srv/www/htdocs
		
#basic commands
------------------
	1) Use the systemctl command 	in opensuse
	
	$ sudo systemctl start nginx ## <-- start the service ##
	$ sudo systemctl restart nginx ## <-- restart the service ##
	$ sudo systemctl stop nginx ## <-- stop the service ##
	$ sudo systemctl status nginx ## <-- Get the status of the service ##
	$ sudo systemctl reload nginx ## <-- Get update conf code into running ##
	$ sudo systemctl enable nginx.service  ## enable for boot time running.
	2) use nginx command
	 sudo nginx -t     ## compile configuration code

	

#Basic notes about configuration
----------------------------------
	1) every block must be enclose with {}
	2) every statement must be ended with ; 
	3) every keyword like http or events is known as directive
	4) every nginx variable can be access using $VarName
	5) every config file must end with extension .conf
	5) root directive is http for all sub conf files because nginx.conf include others file inside http block.
	6) http directive can have multiple server and upstream directive
	7) server directive can have multiple location directive
	
	
	
#vhost setting 
------------------

	1) for 1st site  ( create site1.conf)
		
		server {
			listen  80;
	       server_name  		first.amirapp.com;
	       
		   location / {
	            root   /srv/www/htdocs/first;
	            index  index.html index.htm;
	           }
           }
           


	1) for 2nd site ( create site2.conf)
		
		server {
			listen  80;
	       server_name  		second.amirapp.com;
 
		   location / {
	            root   /srv/www/htdocs/second;
	            index  index.html index.htm;
	           }
      }
           
	 Notes: 
	 server match listen port and server_name and find root directory to send response back.           



##Reverse PROXY setting
-------------------------------
	There are two way to configuration the reverse proxy severs
	-----------------------------------------------------------
	1) without upstream
	2) with upstream

	##1) without upstream
		
		server {
			listen  80;
	       server_name init.amirapp.com www.init.amirapp.com;
		   location / {
	            proxy_pass https://www.digitalocean.com/;
	           }
      }

	##2) with upstream
		
		upstream backend_server {
		server localhost:8088;
		
		}
		server {
			listen  80;
	       server_name init2.amirapp.com www.init2.amirapp.com;
		   location / {
	            proxy_pass http://backend_server/;
	           }
      }
		
		

    
 
##Load balancing with nginx setting
-----------------------------------

	NGINX Open Source supports 4 load‑balancing methods, and and NGINX Plus adds two more methods.
	
	1) Round Robin  ( default one --no need to mentioned and use to distributed load evenly across the servers
	2) least_conn   ( A request is sent to the server with the least number of active connections)
	3) ip_hash      ( taking ip of client to find server names)
	4) Generic Hash (



	upstream backend_server {
		//specified load balance methd
		server localhost:8087;
		server localhost:8088;
		server localhost:8089;
		}
		
		server {
			listen  80;
	       server_name init2.amirapp.com www.init2.amirapp.com;
		   location / {
	            proxy_pass http://backend_server/;
	           }
      }

   
   
     example of upstream with method
     
     upstream backend_server {
		least_conn;
		server localhost:8087;
		server localhost:8088;
		server localhost:8089;
		}
 
 
 #SSL in NGINX
----------------------
            0) check openssl

            1) sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096

            2) keep  cert path

            /etc/apache2/ssl/apache.crt
            /etc/apache2/ssl/apache.key

            3) 
            sudo vim /etc/nginx/snippets/self-signed.conf
            ssl_certificate /etc/apache2/ssl/apache.crt;
            ssl_certificate_key /etc/apache2/ssl/apache.key;

            4)
            sudo vim /etc/nginx/snippets/ssl-params.conf
            ssl_protocols TLSv1.2;
            ssl_prefer_server_ciphers on;
            ssl_dhparam /etc/nginx/dhparam.pem;
            ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
            ssl_ecdh_curve secp384r1; # Requires nginx >= 1.1.0
            ssl_session_timeout  10m;
            ssl_session_cache shared:SSL:10m;
            ssl_session_tickets off; # Requires nginx >= 1.5.9
            ssl_stapling on; # Requires nginx >= 1.3.7
            ssl_stapling_verify on; # Requires nginx => 1.3.7
            resolver 8.8.8.8 8.8.4.4 valid=300s;
            resolver_timeout 5s;
            # Disable strict transport security for now. You can uncomment the following
            # line if you understand the implications.
            # add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
            add_header X-Frame-Options DENY;
            add_header X-Content-Type-Options nosniff;
            add_header X-XSS-Protection "1; mode=block";

            5) goto vhost/server block
                include snippets/self-signed.conf;
                include snippets/ssl-params.conf;

                like :
                listen 443 ssl;
                     listen [::]:443 ssl;
                     include snippets/self-signed.conf;
                     include snippets/ssl-params.conf;

 
