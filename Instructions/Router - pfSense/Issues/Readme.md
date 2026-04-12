FreeBSD - pfSense - Issues



## Issue - iOS unable to open Web-Server page: This Connection Is Not Private ...

### Symptom

Even though connections to server/services in the local network are possible, the iPad/iPhone is unable to access the Web-Server.

Windows and Linux machines do not have this kind of problem.

On connect with the Web-Server, the following information is presented.

> This Connection Is Not Private
> 
> This website may be impersonating "<own public DDNS name>" to steal your personal or financial information. You should close this page.

Click on "Show Details"

> Safari warns you when a website has a certificate that is not valid. This may happen if an attacker has compromised your connection. For your protection, you can't visit this website. You can check back later to see if this issue has been resolved. You can also contact the website owner to tell them about this error. To learn more, you can view the certificate.

Click on "view the certificate" will show the certificate of the Routers webConfigurator.

Even with "Host Overrides" entries (DNS Resolver) for the public domain name, referring to the IP-Address of the Web-Server, the situation remains unchanged.

Check names resolution on the iOS device, by using for example the "a-Shell" app.

```console
\[\~/Documents\]$ nslookup ddprefixdomaindd.ddbasedomaindd
Server:         <IPv6-Address of Router>
Address:        <IPv6-Address of Router>#53

Name:   ddprefixdomaindd.ddbasedomaindd  
Address: <own public internet address>
```

### Solution / Work-Around

Create a NAT Port Forward for requests on the public Domain Name.

- ⬆ select menu option "**Firewall > NAT**"
     - navigate to tab "**Port Forward**"
     - ⬆ click on "**⤵ Add**"
        - ✏️✓ select **`LAN`** as "**Interface**"
        - keep **`IPv4`** as "**Address Family**"
        - ✏️✓ select **`Any`** as "**Protocol**"
        - ✏️✓ select **`WAN address`** as "**Destination**"
        - ✏️✓ select **`Address or Alias`** as "**Redirect target IP**"
        - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Address**" of "**Redirect target IP**"
        - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⤵ Add**"
        - ✏️✓ select **`LAN`** as "**Interface**"
        - ✏️✓ select **`IPv6`** as "**Address Family**"
        - ✏️✓ select **`Any`** as "**Protocol**"
        - ✏️✓ select **`WAN address`** as "**Destination**"
        - ✏️✓ select **`Address or Alias`** as "**Redirect target IP**"
        - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Redirect target IP**"
        - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"
