# How to make own local certificate?

I recommend, if you have not using global domain name and you want to use local domain names instead.

Or you can use this quick guide for your development environment. I do not recommend for use in production environments.

Open terminal.

Move to ssl folder:

`cd /to/your/ssl/folder`

Please check folder permissions for web server, you can search for information about it on Google, for example.

Edit this with your own details, then run it: 

`openssl req -x509 -newkey rsa:2048 -nodes -keyout YOUR-LOCAL-DOMAINNAME.key -out YOUR-LOCAL-DOMAINNAME.crt -days 365 -subj "/C=COUNTRYCODE/O=YOURSERVERNAME/OU=YOURORGANIZATION/CN=YOUR-LOCAL-DOMAINNAME" -addext "subjectAltName=DNS:YOUR-LOCAL-DOMAINNAME,DNS:*.YOUR-LOCAL-DOMAINNAME"`

Configurate certificates to your web server...
