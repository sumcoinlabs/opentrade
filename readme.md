# OpenTrade is the best opensource cryptocurrency exchange!

Step-by-step install instructions:

1. Register on the VPS hosting like this https://m.do.co/c/b7bfce81f64d
2. Create "Droplet" Ubuntu 20.04 x64 / 2GB / 2vCPU / 25 GB SSD
3. Log in to Droplet over SSH (You will receive a email with IP, username and password)


```
sudo apt-get update
sudo apt-get install build-essential libssl-dev curl -y

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
sudo reboot now

nvm install 12.6.0
nvm use 12.6.0
nvm alias default 12.6.0
echo "12.6.0" > .nvmrc

git clone --recurse-submodules https://github.com/sumcoinlabs/opentrade.git
cd opentrade/accountsserver
git checkout master
cd ..

npm install
npm install -g forever
```

## Here is an example of the file ~/opentrade/server/modules/private_constants.js Edit with your configs.
```
'use strict';

exports.recaptcha_priv_key = 'YOUR_GOOGLE_RECAPTCHA_PRIVATE_KEY';
exports.password_private_suffix = 'LONG_RANDOM_STRING1';

exports.walletspassphrase = {
    'SUM' : 'LONG_RANDOM_STRING2',
    'BTC' : 'LONG_RANDOM_STRING3',
    'DOGE' : 'LONG_RANDOM_STRING4'
};
```
**You will need a domain name and SSL for this to work**
```
sudo apt update
sudo apt install certbot
sudo certbot certonly --standalone -d domain.com -d www.domain.com

Reference that in ~/opentrade/server/modules/private_constants.js: example "/etc/letsencrypt/live/domain.com/privkey.pem"
exports.SSL_KEY = '/etc/letsencrypt/live/domain.com/privkey.pem'; //change to your ssl certificates private key
exports.SSL_CERT = '/etc/letsencrypt/live/domain.com/fullchain.pem'; //change to your ssl certificates fullchain
```
**You MUST change default value exports.password_private_suffix !**

**You have to setup postfix to send/get emails**
```
sudo apt update
sudo apt install -y postfix
```
//During the installation, you will be prompted to select the mail server configuration type.
//Choose "Internet Site" if you want to send and receive emails over the internet.
// Postfix Configuration
// enter the domain name (YOUR-domain.com) after choosing Internet site
// You can upodate the mail configuration later with
```
sudo dpkg-reconfigure postfix
```
// Install mailutils
```
sudo apt install -y mailutils
```
// DNS Record in TXT - SPF record. You will need a TXT record in your domain's DNS like this or it won't deliver
```
v=spf1 mx a:domain.com ip4:YOUR-IPv4-ADDRESS ~all
```
## Setting Up DKIM for YOUR-DOMAIN.com Server

DKIM (DomainKeys Identified Mail) is an email authentication method that helps improve email deliverability and 
reduces the chances of emails being marked as spam. Follow these steps to set up DKIM for your YOUR-DOMAIN.com server.

### Step 1: Generate DKIM Key Pair

First, we need to generate a DKIM key pair using OpenDKIM. If OpenDKIM is not already installed on your server, 
you can install it with the following command:
```
sudo apt install opendkim opendkim-tools
```
Generate the key pair using the opendkim-genkey command, replacing selector with your chosen selector 
(a string that helps identify the key pair):
```
sudo opendkim-genkey -t -s selector -d YOUR-DOMAIN.com
```
This will create two files: selector.private (the private key) and selector.txt (the public key). 
Keep the private key secure.

### Step 2: Publish the Public Key in DNS
Next, we need to publish the public key in DNS. Open the selector.txt file to view the public key, 
which will look something like this:
selector._domainkey IN TXT "v=DKIM1; h=sha256; k=rsa; p=<your-public-key>"

Copy the entire TXT record, including the quotes.

Log in to your DNS provider's control panel or domain registrar's website.

Add a new DNS TXT record with the following information:

Hostname: selector._domainkey (replace selector with your chosen selector)
Value: Paste the entire TXT record you copied from selector.txt
TTL (Time to Live): Set an appropriate TTL (e.g., 600 seconds)
Save the DNS record. It may take some time for DNS propagation.

### Step 3: Configure OpenDKIM
Edit the OpenDKIM configuration file:
```
sudo nano /etc/opendkim.conf
```
Add the following lines to the configuration file, adjusting them as needed:

Domain                  YOUR-DOMAIN.com
KeyFile                 /etc/opendkim/selector.private
Selector                selector

Domain: Your domain (YOUR-DOMAIN.com).
KeyFile: Path to the private key file you generated.
Selector: The selector you chose earlier.
Save the file and exit the text editor.

Edit the OpenDKIM defaults file:
```
sudo nano /etc/default/opendkim
```
Uncomment and modify the following line to specify the socket:
```
SOCKET="inet:12345@localhost" # Replace 12345 with the port you want to use
```
Save the file and exit.

### Step 4: Restart OpenDKIM and Your Email Server
Restart the OpenDKIM service:
```
sudo systemctl restart opendkim
```
Restart your email server (e.g., Postfix or whatever you are using):
```
sudo systemctl restart postfix
```
### Step 5: Test DKIM Signing
Send a test email from your server to an email address you control (e.g., a Gmail account).

Check the email headers in your recipient's mailbox to verify that the DKIM signature is present and valid.

You can use email header analyzer tools online to inspect the DKIM signature.

Once DKIM is set up and functioning correctly, your outgoing emails should be signed with DKIM, 
which can improve their deliverability and reduce the chances of being marked as spam.

## TEST EMAILS 
//To test if Postfix and emails are working, you can send a test email from the command line:
```
echo "This is a test email." | mail -s "Test Email" your-email@example.com
```

**After that succeeds, you can run the exchange with this command.  Note, you do not need && git checkout master after first start**

```
cd ~/opentrade/databaseServer && forever start main.js && cd ~/opentrade/accountsserver && git checkout master && forever start main.js && cd ~/opentrade/server && forever start main.js
```
**Second time starting**
```
cd ~/opentrade/databaseServer && forever start main.js && cd ~/opentrade/accountsserver && forever start main.js && cd ~/opentrade/server && forever start main.js
```
In your browser address bar, type https://127.0.0.1
You will see OpenTrade.
**To Stop servers, run**
```
forever stopall
```
The first registered user will be exchange administrator. 

# Add trade pairs

For each coin you should create ~/.coin/coin.conf file

This is common example for ~/.sumcoin/sumcoin.conf

```
rpcuser=long_username_string
rpcpassword=long_password_string
rpcport=3332
rpcclienttimeout=10
rpcallowip=127.0.0.1 //IP of your core client
server=1
daemon=1
upnp=0
rpcworkqueue=1000
enableaccounts=1
litemode=1
staking=0
addnode=1.2.3.4
addnode=5.6.7.8
port=3333
```

Also, you must encrypt your cryptocurrency wallet with this command.

```
./marycoin-cli encryptwallet random_long_string_SAME_AS_IN_FILE_private_constants.js

```
*If coin have no "coin-cli" file then try something like "coind" instead*

*If coin is not supported by encryption (like ZerroCash and it forks) the coin can not be added to OpenTrade.*


Add your coin details to OpenTrade

1. Register on exchange. The first registered user will be exchange administrator.
2. Go to "Admin Area" -> "Coins" -> "Add coin"
3. Fill up all fields and click "Confirm"
4. Fill "Minimal confirmations count" and "Minimal balance" and uncheck and check "Coin visible" button
5. Click "Save"
6. Check RPC command for the coin. If it worked then coin was added to the exchange!

All visible coins should be appear in the Wallet. You should create default coin pairs now.

File ~/opentrade/server/constants.js have settings that you can change

https://github.com/sumcoinlabs/opentrade/blob/master/server/constants.js

```
exports.NOREPLY_EMAIL = 'no-reply@email.com'; //change no-reply email
exports.SUPPORT_EMAIL = 'support@email.com'; //change to your valid email for support requests
const DOMAIN = 'localhost'; //Change to your domain name

exports.TRADE_MAIN_COIN = "Sumcoin"; //change Sumcoin to your main coin pair
exports.TRADE_DEFAULT_PAIR = "Bitcoin"; //change Bitcoin to your default coin pair
exports.share.TRADE_COMISSION = 0.001; //change trade comission percent
exports.share.DUST_VOLUME = 0.000001; //change minimal order volume

exports.recaptcha_pub_key = "6LeX5SQUAAAAAKTieM68Sz4MECO6kJXsSR7_sGP1"; //change to your recaptcha public key

```

File ~/opentrade/static_pages/chart.html

```
const PORT_SSL = 40443; //change to your ssl port (usualy 443)
const MAIN_COIN = 'Sumcoin'; //change Sumcoin to your main coin pair same as in constants.js
const DEFAULT_PAIR = 'Bitcoin'; //change Litecoin to your default coin pair same as in constants.js
      
const TRADE_COMISSION = 0.001;
```

After that, you coins should appear on the main page.



**Donate**
If you find this script is useful then consider donate please



# License

OpenTrade is released under the terms of the MIT license. See LICENSE for more information or see https://opensource.org/licenses/MIT.



