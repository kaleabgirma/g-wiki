<!-- TITLE: Apache Configuration With SSL -->
<!-- SUBTITLE: This document contains apache(httpd) configuration for SSL specially for Jhipster microservice -->

## Configuration
The source code is located under a dirctory /etc/httpd/conf.d/configname.conf

```yaml
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


<VirtualHost eshop-admin.preprod.orange-sonatel.com:443>
        SSLEngine on
       # SSLCertificateChainFile /etc/httpd/conf.d/keys/DigiCertCA.crt
        SSLCertificateFile /etc/httpd/conf.d/keys/eshop-admin.preprod.orange-sonatel.com.cer
        SSLCertificateKeyFile /etc/httpd/conf.d/keys/eshop-admin.preprod.orange-sonatel.com.key

        Header always append X-Frame-Options SAMEORIGIN
        Header set X-XSS-Protection 1;mode=block
        ServerName   eshop-admin.preprod.orange-sonatel.com
        ErrorLog /etc/httpd/logs/client_443_error.log
        CustomLog /etc/httpd/logs/access_443_log combined
        ProxyPass / http://10.137.18.239:8901/
        ProxyPassReverse / http://10.137.18.239:8901/

        <IfModule mod_reqtimeout.c>
                RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
        </IfModule>

</VirtualHost>
```
