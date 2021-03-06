---
title: SBC connectivity issues
ms.author: v-todmc
author: mccoybot
manager: dcscontentpm
ms.date: 1/21/2021
audience: Admin
ms.topic: article
ms.prod: microsoft-teams
localization_priority: Normal
search.appverid:
- SPO160
- MET150
appliesto:
- Microsoft Teams
ms.custom: 
- CI 124780
- CSSTroubleshoot 
ms.reviewer: mikebis
description: Describes how to diagnose SIP options or TLS certificate issues with SBC. 
---

# SBC connectivity issues

## Overview of the SIP options process

1.	SBC sends an SIP options request to the Teams SIP proxy FQDN (for example, sip.pstnhub.microsoft.com).
1.	The TLS connection is validated first. If there are any problems (such as the wrong TLS version or an invalid certificate), the connection is closed, and no SIP options are received by the SIP proxy.
1.	If there are no problems, SIP options are received by the SIP proxy and the SBC FQDN in the Record-Route. If it doesn’t exist, then the Contact header is checked to see whether it belongs to any tenant.
1.	If the SBC FQDN is valid and recognized, the SIP proxy returns the “200 OK” message by using the same TLS connection.
1.	The SIP proxy sends SIP options to SBC and to the SBC FQDN that’s received in the Contact header of the SIP options request.
1.	If the SBC responds to the SIP proxy with the “200 OK” response, it is marked in the Teams Admin portal as Active.

> [!note]
> In a [Hosted Model](https://docs.microsoft.com/microsoftteams/direct-routing-sbc-multiple-tenants), the SIP options are sent only to and from the hosted SBC. The status of derived SBCs will be based on the main SBC.

### SBS setup and communication issues using SIP options

<details>
<summary><b>Not receiving a “200 OK” response from SBC</b></summary>

If you’re not receiving a “200 OK” response from SBC, here are a few things you can try. 
- This situation might occur if you’re using an older version of TLS. To enforce stricter security, [enable TLS 1.2](https://docs.microsoft.com/mem/configmgr/core/plan-design/security/enable-tls-1-2). 
- Make sure that either your SBC certificate is self-signed or a certificate was obtained from a trusted Certificate Authority (CA). 
- If you’re using the minimum required TLS, and no problem is affecting your SBC certificate, your issue most likely occurs because the FQDN is misconfigured in your SIP profile, and it isn’t recognized as belonging to the tenant. To resolve this, narrow down which configuration is causing the issue. The most likely culprits are the following conditions: 
    - The FQDN name that’s provided by SBC in the Record Route or Contact Header differs from what was configured in the Microsoft system. 
    - The FQDN Contact Header contains an IP address instead of the expected FQDN value. 
    - The domain isn’t yet [fully validated](https://docs.microsoft.com/microsoft-365/admin/setup/add-domain). If you add an FQDN that wasn’t previously validated, you must validate it now. 
    - After you register an SBC domain name, you must activate it by [adding at least one E3- or E5-licensed user](https://docs.microsoft.com/microsoftteams/direct-routing-connect-the-sbc#connect-the-sbc-to-the-tenant).

</details>

<details>

<summary><b>“200 OK” response received, but not SIP options</b></summary>

SBC received the “200 OK” response and the FQDN name that’s provided by SBC in the Record Route or Contact Header, but it didn't receive the SIP options you were expecting. If this occurs, make sure that the domain name was entered correctly and that the FQDN DNS record resolves to the correct SBC IP address. 

Another possible cause is that firewall rules aren’t allowing incoming traffic. Make sure that firewall rules are configured to allow incoming connections.

</details>

<details>
<summary><b>SBC status is intermittently inactive</b></summary>

During maintenance or outages, an IP address that the domain points to sometimes changes to a different datacenter while the SBC continues to send SIP options to the inactive or unresponsive datacenter. SBC must be both discoverable and configured to send SIP options to FQDNs. Otherwise, you might experience issues. 
- Check the FQDN status of SBC. If the FQDNs are configured to send SIP options to the specific IP address that the FQDNs resolve to, update it to send SIP options to FQDNs (for example, sip.pstnhub.microsoft.com). 
- Make sure that all devices on the path, such as SBCs and firewalls, are configured to allow communication both to and from all Microsoft-signaling FQDNs. 
- To provide a failover when the connection from an SBC is made to a datacenter that's experiencing an issue, SBC must be configured to use all three SIP proxy FQDNs:
    - sip.pstnhub.microsoft.com
    - sip2.pstnhub.microsoft.com
    - sip3.pstnhub.microsoft.com

    > [!note]
    > Devices that support DNS names can use sip-all.pstnhub.microsoft.com to resolve to all possible IP addresses. 

For more information, see [SIP Signaling: FQDNS](https://docs.microsoft.com/microsoftteams/direct-routing-plan#sip-signaling-fqdns).

</details>

<details>
<summary><b>SIP proxy doesn’t respond to SIP messages from SBC</b></summary>

Multilevel domains can’t be represented by a wildcard character. For example, the wildcard *.contoso.com would match sbc1.contoso.com, but not customer10.sbc1.contoso.com.

If you notice that the FQDN doesn’t match the contents of the Common Name (CN) certificate or Subject Alternate Name (SAN) certificate, request a certificate that matches the selected domains.  

For more information about certificates, see the “Public trusted certificate for the SBC” section of [Plan Direct Routing](https://docs.microsoft.com/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).
</details>

<details>
<summary><b>After successful domain name activation, no valid users are displayed</b></summary>

After you successfully register an SBC domain name and add at least one E3- or E5-licensed user, the name can take up to 24-hours to activate. 

For a list of the licenses that are required for Direct Routing, see the ”Licensing and other requirements” section of [Plan Direct Routing](https://docs.microsoft.com/MicrosoftTeams/direct-routing-plan#licensing-and-other-requirements).

For more information about this process, see the ”Connect the SBC to the tenant” section of [Connect your Session Border Controller (SBC) to Direct Routing](https://docs.microsoft.com/microsoftteams/direct-routing-connect-the-sbc#connect-the-sbc-to-the-tenant).
</details>

### TLS certificate-related issues with SBC setup

After the TLS connection is successfully established and SBC is able to send and receive messages to and from the Teams SIP proxy, there might still be problems that affect the format or content of the SIP messages. The following section provides resolutions that you can try. 
 
<details>
<summary><b>Connection closes and SIP options don’t send, or “200 OK” message is not received</b></summary>

If your connection closes and SIP options are not sent, or the “200 OK” message is not received, the cause might be that the SBC certificate is self-signed or is not assigned by a trusted certificate authority (CA). 

To resolve this problem, check the following:
- This situation might occur if you’re using an older version of TLS. To enforce stricter security, [enable TLS 1.2](https://docs.microsoft.com/mem/configmgr/core/plan-design/security/enable-tls-1-2). 
- Make sure that either your SBC certificate is self-signed or a certificate obtained from a trusted Certificate Authority (CA). The certificate must contain at least one FQDN that belongs to any Microsoft 365 tenant. 

For a list of supported CAs, see the “Public trusted certificate for the SBC” section of [Plan Direct Routing](https://docs.microsoft.com/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).
</details>

<details>
<summary><b>SBC doesn’t trust Teams SIP proxy certificate</b></summary>

Download and install the Baltimore CyberTrust Root Certificate in the SBC Trusted Root Store of the Teams TLS context.

For a list of supported CAs, see the “Public trusted certificate for the SBC” section of [Plan Direct Routing](https://docs.microsoft.com/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).

</details>

<details>
<summary><b>SBC certificate is invalid</b></summary>

You find that the SBC certificate is invalid, expired, or revoked.

Request or renew the certificate from a trusted Certificate Authority (CA). Install it on the SBC according to the vendor’s SBC configuration guide.

For a list of supported CAs, see the “Public trusted certificate for the SBC” section of [Plan Direct Routing](https://docs.microsoft.com/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).

</details>


<details>
<summary><b>SBC or all required intermediary certificates are missing in SBC’s TLS “Hello” message</b></summary>

Check that a valid certificate and all intermediate Certificate Authority certificates are installed correctly, and that the TLS connection settings are correct on SBC.

In some cases, everything might look correct. However, a closer examination of the packet capture might reveal that the TLS certificate of the Intermediate Certificate Authority isn’t provided to the Teams infrastructure.

</details>

<details>
<summary><b>SBC connection is interrupted</b></summary>

The connection is interrupted or does not finish, even though there are no issues caused by certificates or settings on the SBC.

In some cases, the connection may be closed by one of the intermediary devices (for example, the firewall or a router) on the path between the SBC and the Microsoft network. To resolve this issue, verify that there are no connection issues within your managed network.

</details>

## More information

Still need help? Go to [Microsoft Community](https://answers.microsoft.com/).
