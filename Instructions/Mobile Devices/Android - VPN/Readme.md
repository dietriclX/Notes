Android - VPN

# IPsec

In the "**⚙ Settings**" of Android follow the [instructions](https://github.com/dietriclX/Notes/tree/main/Instructions).

- ⬆ tap on "**Network and Internet**"
	- ⬆ tap on "**VPN**"
	    - ⬆ tap on "**➕**" (top right corner)
		    - 🖉✓ type `Home` in "**Name**"
		    - keep selected **`IKEv2/IPsec RSA`**¹ as "**Type**"
		    - 🖉✓ type **`<server name>`**² in "**Server address**"
		    - 🖉✓ type **`<server name>`**² in "**IPSec identifier**"
		    - 🖉✓ select own client certificate³ as "**IPSec user certificate**"
	- 🖉 ~~disable~~ "**Show search suggestions**"
	- 🖉 ~~disable~~ "**Search synchronized tabs**"
	- 🖉 ~~disable~~ "**Show clipboard suggestions**"
	- 🖉 ~~disable~~ "**Show voice search**"
	- 🖉 ~~disable~~ "**Autocomplete URLs**"
	- 🔙 navigate back

¹ Matches IPsec configuration **`EAP-TLS`** as "**Authentication Method**".

² Matches IPsec configuration **`<server name>`** in "**My identifier**" ("**ASN.1 distinguished Name**").

² Signed by the CA defined in IPsec configuration as "**Peer Certificate Authority**".