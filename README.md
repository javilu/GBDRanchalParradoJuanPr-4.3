# Práctica 4.3: Despliegue de una arquitectura EFS-EC2-MultiAZ.

# Script utilizado en los servidores web.

```
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
yum -y install nfs-utils
```

# Script utilizado en el balanceador.

```
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
```

# Comandos utilizados en los servidores web.

```
cd /var/www/html
sudo mkdir efs-mount
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-03846e04368a76481.efs.us-east-1.amazonaws.com:/ efs-mount
cd efs-mount
sudo wget https://s3.eu-west-1.amazonaws.com/www.profesantos.cloud/Netflix.zip
sudo unzip Netflix.zip
sudo systemctl restart httpd
```

# Comandos utilizados en el balanceador.

```
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_ajp
sudo a2enmod rewrite
sudo a2enmod deflate
sudo a2enmod headers
sudo a2enmod proxy_balancer
sudo a2enmod proxy_connect
sudo a2enmod proxy_html
sudo a2enmod lbmethod_byrequests
```

# Modificación realizada en el archivo /etc/apache2/sites-available/000-default.conf en el balanceador.

```
ProxyPass /balancer-manager !

<Proxy balancer://clusterasir>
        # Server 1
        BalancerMember http://172.31.82.35

        # Server 2
        BalancerMember http://172.31.30.166
</Proxy>

ProxyPass / balancer://clusterasir/
ProxyPassReverse / balancer://clusterasir/

<Location /balancer-manager>
       SetHandler balancer-manager
       Order Deny,Allow
       Allow from all
</Location>
```

# Modificación realizada en el archivo /etc/fstab en el balanceador.

```
fs-03846e04368a76481.efs.us-east-1.amazonaws.com:/	/var/www/html/efs-mount	nfs	defaults	0	0
```
