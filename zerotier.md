### How to Configure ZeroTier


```
curl -s https://install.zerotier.com | sudo bash

sudo chown $USER:group .zeroTierOneAuthToken
sudo cat /var/lib/zerotier-one/authtoken.secret >>.zeroTierOneAuthToken
chmod 0600 .zeroTierOneAuthToken

zerotier-cli join <network-id>
 ```
