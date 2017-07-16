#how to build apache2.4 and php7 environment on windows10

##source
[php7.1.7 vc14 x64 non thread Safe](http://windows.php.net/downloads/releases/php-7.1.7-nts-Win32-VC14-x64.zip)
[apache2.4 vc14 x64](https://www.apachelounge.com/download/VC14/binaries/httpd-2.4.27-win64-VC14.zip)
[mod_fcgid vc14 x64](https://www.apachelounge.com/download/VC14/modules/mod_fcgid-2.3.9-win64-VC14.zip)

##steps
suppose my root directory is d:/webservice
1. unzip httpd-2.4.27-win64-VC14.zip and copy the Apache24 directory to d:/webservice
2. unzip php-7.1.7-nts-Win32-VC14-x64.zip and copy it to d:/webservice, then rename the directory to php
3. unzip mod_fcgid-2.3.9-win64-VC14.zip and copy the mod_fcgid.so file to d:/webservice/Apache24/modules
4. cd d:/webservice/php, copy and rename the file 'php.ini-development' to 'php.ini'
5. edit d:/webservice/Apache24/conf/httpd.conf, search c:/Apache24 and change it to d:/webservice/Apache24
6. add 'LoadModule fcgid_module modules/mod_fcgid.so' to httpd.conf file
7. add flow lines to httpd.conf or copy it to a new file and include it in httpd.conf file, just like 'Include conf/extra/httpd-vhosts.conf'
   FcgidInitialEnv PATH "D:/webservice/php;C:/WINDOWS/system32;C:/WINDOWS;C:/WINDOWS/System32/Wbem;"
   FcgidInitialEnv SystemRoot "C:/Windows"
   FcgidInitialEnv SystemDrive "C:"
   FcgidInitialEnv TEMP "C:/WINDOWS/Temp"
   FcgidInitialEnv TMP "C:/WINDOWS/Temp"
   FcgidInitialEnv windir "C:/WINDOWS"
   FcgidIOTimeout 64
   FcgidConnectTimeout 16
   FcgidMaxRequestsPerProcess 1000 
   FcgidMaxProcesses 50 
   FcgidMaxRequestLen 8131072
   # Location php.ini:
   FcgidInitialEnv PHPRC "D:/webservice/php"
   FcgidInitialEnv PHP_FCGI_MAX_REQUESTS 1000

   <Files ~ "\.php$">
     AddHandler fcgid-script .php
     FcgidWrapper "D:webservice/php/php-cgi.exe" .php
   </Files>
8. go to line 262, and change 'Options Indexes FollowSymLinks' to 'options +ExecCGI'
9. go to line 282, append index.php to 'DirectoryIndex index.html'
10. cd d:/webservice/Apache24/htdocs, create a new php file with content '<?php phpinfo(); ?>', named 'index.php'
11. open your browser, and access 127.0.0.1/index.php, you will see the php settings
12. it`s over
##remind
if you add a new virtualhost, add 'Options +ExecCGI' to it; if not you will get 403 forbidden error