**httpd-website-on-ec2**



sudo su -
yum update -y
yum install -y httpd
systemctl status httpd
mkdir temp
cd temp
wget https://www.free-css.com/assets/files/free-css-templates/download/page269/complex.zip

unzip complex.zip
cd complex
ls -lrt
mv * /var/www/html
systemctl enable httpd
systemctl start httpd

