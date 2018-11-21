- Download openvpn config files as per https://protonvpn.com/support/linux-vpn-setup/ step 2 and step 3
- In the .ovpn file of the setup you downloaded change "compress" to "comp-lzo"
- SSH into your USG
- Put the .ovpn file in /config/user-data/{vnpfile}
- Create a file for the user credentials in /config/user-data/{username/pw}
- Setup masquarade rules for the VPN
 [assuming you have no rule 5004 yet!]
configure
set service nat rule 5004 description "masq to vpn tun0"
set service nat rule 5004 destination address 0.0.0.0/0
set service nat rule 5004 outbound-interface tun0
set service nat rule 5004 type masquerade
commit;save;exit


- Now try it out with
openvpn --config /config/user-data/serverlist.ovpn --auth-user-pass /config/user-data/protonvpn.user

[now you should be able to browse the web normally and if you go see what your IP is, this should show the ProtonVPN ip, not your own.]

- If all works, you wanna set this up a bit more persistent and run openvpn as a daemon:
openvpn --config /config/user-data/serverlist.ovpn --auth-user-pass /config/user-data/protonvpn.user --daemon


If you want the NAT rules to be persistent you can create a config.gateway.json file on your controller. Here's an example of mine:

/usr/lib/unifi/data/sites/default/config.gateway.json
{
   "service":{
      "nat":{
         "rule":{
            "5004":{
               "description":"masq to vpn tun0",
               "destination":{
                  "address":"0.0.0.0/0"
               },
               "outbound-interface":"tun0",
               "type":"masquerade"
            }
         }
      }
   }
}

Note: The main downside of this method, if the system is rebooted the VPN will not be enabled since I havent figured out how to set custom commands persistent after a firmware upgrade.

