---
title: Apache Configuration With SSL
description: 
published: true
date: 2019-10-22T15:58:07.641Z
tags: 
---

## Configuration
Create configuration file under the directory **/etc/httpd/conf.d/<confname>.conf** and paste the following source code

 Note: Don’t foregate to change the SSL certificate directory
``` yml
RewriteEngine on
RewriteCond %{SERVER_PORT} !443
RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
RequestHeader set "X-Forwarded-SSL" expr=%{HTTPS}
#RequestHeader set X-Forwarded-Proto "https"
#Header edit Set-Cookie ^(.*)$ $1;Secure;HttpOnly
SSLProtocol  all -SSLv2 -SSLv3
SSLProxyProtocol  all -SSLv2 -SSLv3
SSLHonorCipherOrder on
SSLCipherSuite  ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256:!RC4:HIGH:!MD5:!aNULL:!EDH
SSLProxyCipherSuite ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256:!RC4:HIGH:!MD5:!aNULL:!EDH

#KeepAlive On
<IfModule mpm_prefork_module>
   StartServers        3
   MinSpareServers     5
   MaxSpareServers     10
   MaxClients          250
   MaxRequestsPerChild 3000
</IfModule>

<VirtualHost eshop-admin.orange-sonatel.com:443>
        SSLEngine on
        SSLCertificateFile /etc/httpd/conf.d/keys/fibre_orange_sn.crt
        SSLCertificateKeyFile /etc/httpd/conf.d/keys/fibre_orange_sn.key

        Header always append X-Frame-Options SAMEORIGIN
        Header set X-XSS-Protection 1;mode=block
        ServerName   eshop-admin.orange-sonatel.com
        ErrorLog /etc/httpd/logs/client_443_error.log
        CustomLog /etc/httpd/logs/access_443_log combined
        ProxyPass / http://10.137.18.69:8901/
        ProxyPassReverse / http://10.137.18.69:8901/

        <IfModule mod_reqtimeout.c>
                RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
        </IfModule>

</VirtualHost>
```