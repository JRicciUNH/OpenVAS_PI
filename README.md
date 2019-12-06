# OpenVAS_PI
How to guide on how to set up OpenVAS on a Raspberry Pi 3 B+ with email notifications

# Requirements

* Email account to send OpenVAS notifications from
* Raspberry Pi (I have only tested this on the Pi 3 B+, im not sure how well this will work on earlier versions)
* SD card with Raspbian Buster Lite

# Setup Raspberry Pi
There are plenty of guides on how to flash your SD card, just look up how to do that. 

I used Balena Etcher to flash and format the SD card.

Create a file in the /boot/ directory called "ssh" - This will automatically start SSH server so you can connect to the Pi headless (without GUI)

Pop in the SD card and power up your Pi

Give it a couple of minutes to boot up. Then SSH to your raspberry Pi. 

# Setup OpenVas
Once you have shelled into your Pi run the following:

sudo apt install openvas
sudo openvas-setup

This usually takes a while to setup so give it time.

Once that is installed you should have user credentials. If you did not happen to see creds appear on your screen then create an admin user.

openvasmd --create-user="username"

The password will display on your terminal. 

Next we will setup the mail server.

# Setup Mail Server

sudo apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules

Modify the following file:
nano /etc/postfix/main.cf

and paste this block at the end of the file:
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/postfix/cacert.pem
smtp_use_tls = yes

Create the file that will hold your gmail credentials:
vim /etc/postfix/sasl_passwd
Modify the following line and add it to the file
[smtp.gmail.com]:587    USERNAME@gmail.com:PASSWORD

sudo chmod 400 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd

Check if your certificate works:
cat /etc/ssl/certs/Thawte_Premium_Server_CA.pem | sudo tee -a /etc/postfix/cacert.pem

If that did not work try the next command

cat /etc/ssl/certs/thawte_Primary_Root_CA.pem | sudo tee -a /etc/postfix/cacert.pem

sudo /etc/init.d/postfix reload


Send yourself a test email to make sure it worked
echo "Test mail from postfix" | mail -s "Test Postfix" <username>@<domain>.com
  
  
