---
title: Protected Users Security Group
description: "Windows Server Security"
ms.custom: na
ms.prod: windows-server-threshold
ms.reviewer: na
ms.suite: na
ms.technology: security-credential-protection
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 1b0b5180-f65a-43ac-8ef3-66014116f296
author: coreyp-at-msft
ms.author: coreyp
manager: dongill
ms.date: 10/12/2016
---
# Protected Users Security Group

>Applies To: Windows Server 2016, Windows Server 2012 R2, Windows Server 2012

This topic for the IT professional describes the Active Directory security group Protected Users, and explains how it works. This group was introduced in  Windows Server 2012 R2 .

## <a name="BKMK_ProtectedUsers"></a>Members of this group are afforded additional protections against the compromise of credentials during authentication processes.

This security group is designed as part of a strategy to effectively protect and manage credentials within the enterprise. Members of this group automatically have non-configurable protections applied to their accounts. Membership in the Protected Users group is meant to be restrictive and proactively secure by default. The only method to modify these protections for an account is to remove the account from the security group.

> [!WARNING]
> Accounts for services and computers should not be members of the Protected Users group. This group provides no local protection because the password or certificate is always available on the host for the service or host. Authentication will fail with the error "the user name or password is incorrect" for any service or computer that is added to the Protected Users group.

This domain-related, global group triggers non-configurable protection on devices and host computers running  Windows Server 2012 R2  and Windows 8.1, and on domain controllers in domains with a primary domain controller running  Windows Server 2012 R2. This greatly reduces the memory footprint of credentials when users sign in to computers on the network from a non-compromised computer.

Depending on the account's domain functional level, members of the Protected Users group are further protected due to behavior changes in the authentication methods that are supported in Windows.

-   The member of the Protected Users group cannot authenticate by using NTLM, Digest Authentication, or CredSSP. On a device running Windows 8.1, passwords are not cached, so the device that uses any one of these Security Support Providers (SSPs) will fail to authenticate to a domain when the account is a member of the Protected User group.

-   The Kerberos protocol will not use the weaker DES or RC4 encryption types in the pre-authentication process. This means that the domain must be configured to support at least the AES cipher suite.

-   The user's account cannot be delegated with Kerberos constrained or unconstrained delegation. This means that former connections to other systems may fail if the user is a member of the Protected Users group.

-   The default Kerberos Ticket Granting Tickets (TGTs) lifetime setting of four hours is configurable by using Authentication Policies and Silos, which can be accessed through the Active Directory Administrative Center (ADAC). This means that when four hours has passed, the user must authenticate again.

For more information, see [How it works](#BKMK_HowItWorks) in this topic.

The following table specifies the properties of the Protected Users group.

|Attribute|Value|
|-------|-----|
|Well-known SID/RID|S-1-5-21-<domain>-525|
|Type|Domain Global|
|Default container|CN=Users, DC=<domain>, DC=|
|Default members|None|
|Default member of|None|
|Protected by ADMINSDHOLDER?|No|
|Safe to move out of default container?|Yes|
|Safe to delegate management of this group to non-service admins?|No|
|Default user rights|No default user rights|

### <a name="BKMK_HowItWorks"></a>How the Protected Users group works
This section explains how the Protected Users group works when:

-   A user signs in a Windows devices which supports Protected User group client-side protections.

-   The user belongs to a Windows Server 2012 R2 domain functional level domain.

**Protected User group client-side protections**

When a user signs in a Windows device which supports Protected User group client-side protections and they are a member for the Protected users group, client-side protections are automatically applied. Members of the Protected Users group can no longer authenticate by using:

-   Default credential delegation (CredSSP). Plain text credentials are not cached even when the **Allow delegating default credentials** Group Policy setting is enabled.

-   Windows Digest. Plain text credentials are not cached even when Windows Digest is enabled.

-   NTLM. The result of the NT one-way function, NTOWF, is not cached.

-   Kerberos DES or RC4 keys. Only AES Kerberos long term keys are created from a password sign in. The Kerberos initial TGT is cached to provide SSO, however the Kerberos long term keys are not cached after the initial TGT is recieved. So each request for an intial TGT requires the user to provide credentials.

-   Sign-in offline. A cached verifier is not created at sign-in.

After the user account is added to the Protected Users group, protection is already in place when the user signs in to the domain.

**Enabling Protected Users security group with domain controllers earlier than  Windows Server 2012 R2**

The Protected Users group can be applied to domain controllers that run an operating system earlier than  Windows Server 2012 R2 . This allows the added security that is achieved by using the Protected Users group to be applied to Windows devices that support the client-side protections even without upgrading the DFL. The Protected Users group can be created by  HYPERLINK "http://technet.microsoft.com/library/cc816944(v=ws.10).aspx" transferring the primary domain controller (PDC) emulator role to a domain controller that runs Windows Server 2012 R2. After that group object is replicated to other domain controllers, the PDC emulator role can be hosted on a domain controller that runs an earlier version of Windows Server.

For more information, see [How to Configure Protected Accounts](how-to-configure-protected-accounts.md).

**Protected Users group Domain Controller protections**

Accounts that are members of the Protected Users group that authenticate to a Windows Server 2012 R2 domain are unable to:

-   Authenticate with NTLM authentication.

-   Use DES or RC4 encryption types in Kerberos pre-authentication.

-   Be delegated with unconstrained or constrained delegation.

-   Renew the Kerberos TGTs beyond the initial four-hour lifetime.

Non-configurable settings to the TGTs expiration are established for every account in the Protected Users group. Normally, the domain controller sets the TGTs lifetime and renewal, based on the domain policies, **Maximum lifetime for user ticket** and **Maximum lifetime for user ticket renewal**. For the Protected Users group, 600 minutes is set for these domain policies.

> [!WARNING]
> Accounts for services and computers should not be members of the Protected Users group. This group provides no local protection because the password or certificate is always available on the host.

### Event log information
Two operational administrative logs are available to help troubleshoot events that are related to Protected Users. These new logs are located in Event Viewer and are disabled by default, and are located under **Applications and Services Logs\Microsoft\Windows\Microsoft\Authentication**.

|Event ID and Log|Description|
|----------|--------|
|104<br /><br />**ProtectedUser-Client**|Reason: The security package on the client does not contain the credentials.<br /><br />The error is logged in the client computer when the account is a member of the Protected Users security group. This event indicates that the security package does not cache the credentials that are needed to authenticate to the server.<br /><br />Displays the package name, user name, domain name, and server name.|
|304<br /><br />**ProtectedUser-Client**|Reason: The security package does not store the Protected User's credentials.<br /><br />An informational event is logged in the client to indicate that the security package does not cache the user's sign-in credentials. It is expected that Digest (WDigest), Credential Delegation (CredSSP), and NTLM fail to have sign-on credentials for Protected Users. Applications can still succeed if they prompt for credentials.<br /><br />Displays the package name, user name, and domain name.|
|100<br /><br />**ProtectedUserFailures-DomainController**|Reason: An NTLM sign-in failure occurs for an account that is in the Protected Users security group.<br /><br />An error is logged in the domain controller to indicate that NTLM authentication failed because the account was a member of the Protected Users security group.<br /><br />Displays the account name and device name.|
|104<br /><br />**ProtectedUserFailures-DomainController**|Reason: DES or RC4 encryption types are used for Kerberos authentication and a sign-in failure occurs for a user in the Protected User security group.<br /><br />Kerberos preauthentication failed because DES and RC4 encryption types cannot be used when the account is a member of the Protected Users security group.<br /><br />(AES is acceptable.)|
|303<br /><br />**ProtectedUserSuccesses-DomainController**|Reason: A Kerberos ticket-granting-ticket (TGT) was successfully issued for a member of the Protected User group.|

### Deployment requirements
Requirements to provide client-side protection for members of the Protected Users group include:

-   The Protected Users global security group is replicated to all domain controllers in the account domain.

-   Devices and hosts are running Windows 8.1 or  Windows Server 2012 R2 .

Requirements to provide domain controller protection for members of the Protected Users group include:

-   The domain functional level in the account domains is set to  Windows Server 2012 R2 .

To enable  Windows Server 2012 R2  and Windows 8.1 protection for clients on domains with pre- Windows Server 2012 R2  domain functional levels, after the Protected Users group has replicated throughout the domain, a user signs in with an account that is a member of a Protected Users group.

## Additional resources

-   [Credentials Protection and Management](credentials-protection-and-management.md)

-   [Authentication Policies and Authentication Policy Silos](authentication-policies-and-authentication-policy-silos.md)

-   [How to Configure Protected Accounts](how-to-configure-protected-accounts.md)


