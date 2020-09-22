---
layout: default
---

# [TUTORIAL] Setting up a VPN for Personal Needs/Remote Work

* * *

<style type="text/css">

</style>

## What is a VPN <i>(oversimplified)</i>  ?

<strong>VPN</strong> is technology that connects 2 devices, like they are on the same private network, but through the publicly routable Internet, using <strong>encapsulation</strong><i>(providing special header where the data should be routed)</i> and <strong>encryption</strong><i>(for secure connection, since private data is going out in the wild).</i> Effectively creating the so called <strong>"tunnel"</strong>, <i>which is used for the data exchange.</i>

## Benefits from VPN:

| Personal Use                        | Business/Corporate World                                                    |
|-------------------------------------|-----------------------------------------------------------------------------|
| 1) Anonymity                        | 1) Enchanted Security/Data Integrity                                        |
| 2) Bypassing restrictions           | 2) Connecting to Corporate Network - <strong>Vital for remote work</strong> |
| 3) Remote Access to Home Network    | 3) <- Everything else from the left section                                 |
| 4) Data Encryption on public WiFi   |                                                                             |
| 5) Preventing network-level threads |                                                                             |


## VPN protocols:

| Protocol | Underlying Security Mechanism | Network Ports | Practical Use |
|-|-|-|-|
| <strong>PPTP</strong>-<i> Point-to-Point Tunneling Protocol </i> | <strong>Basic Encryption</strong> - MS-CHAP-v1,MS-CHAP-v2 | 1723/TCP | Fundamentally Insecure/Broken |
| <strong>L2TP</strong> - <i> Layer 2 Tunneling Protocol </i> | Used along with IPsec - <strong> Strong Encryption </strong> | 500/UDP | Good Enough |
| <strong> SSTP </strong> - <i> Secure Socket Tunneling Protocol </i> | SSL 3.0 - <b>Very Good</b>, BUT proprietary by Microsoft <i> Uses 443/TCP - blends in with the other SSL traffic, hard to block </i> | 443/TCP | Very Good |
| <b>IKEv2</b> - <i> Internet Key Exchange ver. 2</i> | Most of the times, IPPsec in Tunnel Mode. | 500/UPD | Very Good, auto re-establishes the connection, after the Internet has dropped (VPN Reconnect feature) |
| <b>OpenVPN</b> | Open Source. OpenSSL Library, AES encryption | Can use 443/TCP to bypass FW, best runs on 1194/UDP | <b>The Gold Standard!</b> |

#### Bellow I will take a look at configuring the most popular VPN solutions for Personal and Enterprise use.

> * <strong>Life Hack 1 :</strong> If you need to port forward just 1 service, you can use SSH Tunneling to achieve this. <br>
> <u>Example:</u> Using VNC to access a remote station - SSH tunnel to "forward" the port through an encrypted connection.
> Or we are using the more secure protocol <strong>(SSH)</strong> to encapsulate the least secure one <strong>(VNC)</strong>. <br>
> `ssh -L <port on the LOCAL machine>:<Remote Address>:<Remote Port> username@server`<br>
> You can specify `localhost` on place of `<Remote Address>`, this way if the service listens on `lo`,
> which in many cases does(although it depends on what you want to do, and the service itself... ), <br>
> you connect directly without knowing the internal DNS Name/LAN IP. <br>
> Also you may need to play with your `sshd_config`, you can Google it for more information ...


## Setting up OpenVPN - Access Server - <a href="https://www.youtube.com/watch?v=ZEEL4I1S4TQ" target="_blank"> Video</a>

<br>
<div class="video-container">
	<iframe width="1280" height="720" src="https://www.youtube.com/embed/ZEEL4I1S4TQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>
<br><br>
<a href="/pics/vpn/topology-1.png" target="_blank"><img src="/pics/vpn/topology-1.png"></a>
<center><i>Network topology of the OpenVPN Labs</i></center>
<br>
This is the System Administrator's Guide for OpenVPN Access Server <a href="https://openvpn.net/images/pdf/OpenVPN_Access_Server_Sysadmin_Guide_Rev.pdf" target="_blank">[PDF]</a> .<br>
Comparison between OpenVPN Access Server and Community Edition<a href="https://openvpn.net/open-source-vs-openvpn-access-server/" target="_blank"> [1]</a> .<br>
<br>

## Setting up OpenVPN - Community Edition - <a href="https://www.youtube.com/watch?v=MdN22Oy_Tsg" target="_blank"> Video</a>

<br>
<div class="video-container">
	<iframe width="1280" height="720" src="https://www.youtube.com/embed/MdN22Oy_Tsg" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>
<br>

## Setting up Remote Access Role on Windows Server 2019 - VPN only - SSTP - <a href="https://www.youtube.com/watch?v=t0Ib_XmGmWI" target="_blank"> Video</a>

<br>
<div class="video-container">
	<iframe width="1280" height="720" src="https://www.youtube.com/embed/t0Ib_XmGmWI" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

<br><br>
<a href="/pics/vpn/topology-2.png" target="_blank"><img src="/pics/vpn/topology-2.png"></a>
<center><i>Network topology of the Windows Server Lab</i></center>
<br>

<i>Note:</i> If you get an error that Routing and Remote Access Service couldn't start <strong>(Error 8007042a)</strong>, make sure: <br>
1) Your server is patched <br>
2) Services may be starting in wrong order. If you stop NPS service, and then try to start the RRAS service, it should start and automatically trigger NPS to start as well. <br>