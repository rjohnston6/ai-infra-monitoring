Minimum configuration for Dial-In Configurations

feature grpc
feature openconfig

grpc use-vrf default (not required if using mgmt0 interface)

crypto ca trustpoint grpc_trustpoint
 crypto ca import grpc_trustpoint pkcs12 grpc_selfsign2048.pfx C1sco123!

grpc certificate grpc_trustpoint

************ Generated SelfSigned Certificate **********************
switch# run bash sudo su

bash-4.4# openssl req -x509 -newkey rsa:2048 -keyout grpc_selfsigned2048.key -out grpc_selfsigned2048.pem -days 365 -nodes
Generating a RSA private key
.......................................+++++
....+++++
writing new private key to 'grpc_selfsigned2048.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Texas
Locality Name (eg, city) []:Richardson
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Cisco Lab
Organizational Unit Name (eg, section) []:Lab
Common Name (e.g. server FQDN or YOUR name) []:houlab-core-9k-a
Email Address []:admin@admin.com

bash-4.4# openssl pkcs12 -export -out grpc_selfsign2048.pfx -inkey grpc_selfsigned2048.key -in grpc_selfsigned2048.pem -certfile grpc_selfsigned2048.pem -password pass:C1sco123!
bash-4.4# mv grpc_selfsign2048.pfx /bootflash/grpc_selfsign2048.pfx
************************************************************************