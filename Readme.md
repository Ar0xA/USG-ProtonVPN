- Download openvpn config files as per https://protonvpn.com/support/linux-vpn-setup/ step 2 and step 3
- In the .ovpn file of the setup you downloaded change "compress" to "comp-lzo"
- SSH into your USG
- Put the .ovpn file in /config/user-data/{vnpfile}
- Create a file for the user credentials in /config/user-data/{username/pw}. Username on first line, password on second
- Setup masquarade rules for the VPN
 [assuming you have no rule 5004 yet!]<br/>
configure<br/>
set service nat rule 5004 description "masq to vpn tun0"<br/>
set service nat rule 5004 destination address 0.0.0.0/0<br/>
set service nat rule 5004 outbound-interface tun0<br/>
set service nat rule 5004 type masquerade<br/>
commit;save;exit<br/>
<br/>
<br/>
- Now try it out with<br/>
openvpn --config /config/user-data/{vpnfile} --auth-user-pass /config/user-data/{username/pw}
<br/>
[now you should be able to browse the web normally and if you go see what your IP is, this should show the ProtonVPN ip, not your own.]<br/>
<br/>
- If all works, you wanna set this up a bit more persistent and run openvpn as a daemon:
openvpn --config /config/user-data/serverlist.ovpn --auth-user-pass /config/user-data/protonvpn.user --daemon
<br/>
If you want the NAT rules to be persistent you can create a config.gateway.json file on your controller. Here's an example of mine:<br/>
<br/>
/usr/lib/unifi/data/sites/default/config.gateway.json<br/>
{<br/>
   "service":{<br/>
      "nat":{<br/>
         "rule":{<br/>
            "5004":{<br/>
               "description":"masq to vpn tun0",<br/>
               "destination":{<br/>
                  "address":"0.0.0.0/0"<br/>
               },<br/>
               "outbound-interface":"tun0",<br/>
               "type":"masquerade"<br/>
            }<br/>
         }<br/>
      }<br/>
   }<br/>
}<br/>
<br/>
Note: The main downside of this method, if the system is rebooted the VPN will not be enabled since I havent figured out how to set custom commands persistent after a firmware upgrade.

