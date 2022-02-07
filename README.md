# BTCPayServer Payment Forward
At Bitcoin Beach Brazil, we are running a [MyNodeBTC](https://mynodebtc.com) with [BTCPayServer](https://btcpayserver.org) as PoS for small businesses and *Virtual BTC Hats* for our local street artists. This script is useful when you need receive funds in your node and forward to other wallets automaticly. 

# How it works
When a invoice generated by BtcPayServer is settled and the Lightning payment is received in our local node, the BtcPayServer Webhook will call a local PHP file, to forward the payment to the end-user wallet and they receive the funds instantly on BlueWallet, connected to LNDHub and LNBits.

# Requirements
 - MyNodeBTC
 - BTCPayServer
 - LndHub
 - LNBits
 - BlueWallet Mobile
 - PHP-FQM, BC, JQ  

# Server Instalation Instrucions
1) Download the files and run the installation script! 
```
# git clone https://github.com/BitcoinBeachBR/BtcPayServer-Payment-Forward
# chmod 755 install.sh
# ./install.sh
```

2) Running the installation script for the first time? Restart Nginx
```
# /etc/init.d/nginx restart
```

3) Run the command below to test if the files are being created at ```/mnt/hdd/BTCPAYSERVER/invoices```
```
# curl --insecure -d "{}" -X POST http://127.0.0.1:44331/log.php?token=123456789
# ls -la /mnt/hdd/BTCPAYSERVER/invoices
```

4) Some routers doe not redirect internal external ports to internal networks. To mitigate that, put an address resolving to loopback address in ```/etc/hosts```
```
127.0.1.1       myNode  your.somedomain.io 
```

5) Set the ONG to split fee, and make a donation. Insert the ```LNDHUB INVOICE URL``` of the ONG Wallet, and the rate. In the example bellow, the ONG will receive 50% of the fee. 
```
# /mnt/hdd/BTCPAYSERVER/bin/psql_query "insert into charity(id,rate,url) values ('ong_name', 0.5, 'lndhub://invoice:XXXXXXXXXXXXXXXXXX@https://lnbits.jericoacoara.io:5001/lndhub/ext/')" insert
```


6) Set *guard_btcpayserver* to start automaticly after reboot. Add this line to ```/etc/rc.local```, before script ```"exit 0"```
```
( sleep 300; /mnt/hdd/BTCPAYSERVER/bin/guard_btcpayserver ) &
```

# LNBits Payer Wallet Configuration
 - Create a wallet on **LNbits**. Demo server [htpps://lnbits.jericoacoara.io:5001](htpps://lnbits.jericoacoara.io:5001) and name it as Payer Wallet
 - Backup the link generated for your LNBits wallet in a safe location. There are no way to recover.
 - Go to **Extensions** and activate **LndHub**
 - In the menu, click on **LndHub Extension**
 - Copy ```LNDHUB ADMIN URL```
 - Include your ```LNDHUB ADMIN URL``` in /mnt/hdd/BTCPAYSERVER/conf/payer.conf.sample
 - After edit the file and save, run the command to in the shell to rename it
```
# mv /mnt/hdd/BTCPAYSERVER/conf/payer.conf.sample /mnt/hdd/BTCPAYSERVER/conf/payer.conf
```

# Run the Script

1) Running the script first time
```
# ./mnt/hdd/BTCPAYSERVER/bin/guard_btcpayserver
```

# User Interface Configuration Instrucions
1) LNBits Configuration
- Create a wallet on **LNbits**. Demo server [htpps://lnbits.jericoacoara.io:5001](htpps://lnbits.jericoacoara.io:5001)\n
- Backup the link generated for your LNBits wallet in a safe location. There are no way to recover.
- Go to **Extensions** and activate **LndHub**
- In the menu, click on **LndHub Extension**
- Copy ```LNDHUB INVOICE URL```
- Leave the screen open for the next 2 steps
  
2) BlueWaleet Configuration
- Click on Create Wallet
- Import Wallet
- Scan the QRCode ```LNDHUB ADMIN URL``` and import
- Remember to backup your wallet link, and don't share it with anyone. 
  
3) BTCPayServer 
- Create an account on BTCPayServer. Demo server [htpps://btcpay.jericoacoara.io](htpps://btcpay.jericoacoara.io)
- Create a store
- In *Settings*, *Payment Methods*, setup use the Lightning Wallet from the Local Node 
- In *Checkout Appearance* at *Invoice Settings*  *BTC (Off-Chain)* as default payment method on checkout
- In *Webhooks* click on *+ Create New Webhook* 
- In *Payload URL* insert *http://127.0.0.1:44331/log.php?token=**YOUR LNDHUB INVOICE URL or if using LNPAY, use as lnpay://nomedousuario@somehost.com ENCODED IN BASE64*** 
- To encode your ```LNDHUB INVOICE URL or LNPAY URL,``` use [this tool](https://www.base64encode.org/).
- The final Payload URL will look like this
```http://127.0.0.1:44331/log.php?token=bG5kaHViOi8vaW52b2ljZTowZmNiMTY5MmVjNGM0NjUxODc1ZGExMmVlNjgwNjJhOUBodHRwczovL2xuYml0cy5qZXJpY29hY29hcmEuaW86NTAwMS9sbmRodWIvZXh0Lw==```
- Activate *Automatic redelivery(
- In *Events* select Send *specific events* and check *A payment has been settled*
- Add Webhook

# Other Features under development
1) Zero Fee Transactions
Self-payments are not allowed, then, to not need a second node for the users wallets and maintain the funds locally, we created an private channel with 0 fee, connected to the node of our DEV. Basicly when the script is called, we will force to use this 0 fee channel to transfer the funds direcly to the end-user wallet back to our local node, but now controled by the user. 

2) The payments with more than 100.000 sats are paid using the lndcli command line



# Payment Forward Fee Policy
- We are charging 2% flat fee for all forwards. 
- 50% of the charged fee is donated to [Xavi Surf Kids](https://www.xavisurfschool.com/pt/o-team/xavi-surf-kids/), the most important social project of Jericoacoara.
- You can change the fee in your self hosted script, by changing the ```RATE``` in ```payer.conf```

# No Warranty
Version Alpha. 
Use at your own risk. 



