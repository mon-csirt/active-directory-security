# Active Directory Security Guide

## [Monash Enterprise Access Model](./MEAM/README.md)

The "Monash Enterprise Access Model" (MEAM) is a model for tiering Active Directory that builds heavily on the [Microsoft Enterprise Access Model](https://learn.microsoft.com/en-us/security/privileged-access-workstations/privileged-access-access-model).

The MEAM is developed by the Enterprise Engineering team at Monash University, Australia.

The MEAM builds on three core components:
- The __administrative tier__: a "macro" partition of AD based on the level of privilege & control.
- The __zone__: a horizontal split of a *tier*, where *services* are placed into *silos* preventing lateral movement.
- The __service__: a delegation target within a *zone*, containing computers and groups.

__TL;DR__: The MEAM brings microsegmentation to AD, using entirely built-in functionality.

## Active Directory Security Controls

The following is a list of some great *built-in* and *third-party* Active Directory security controls worth exploring:

### [Built-in] 'Protected Users' security group

Protected Users is a global security group for Active Directory (AD) designed to protect against credential theft attacks. The group triggers non-configurable protection on devices and host computers to prevent credentials from being cached when group members sign-in.

Anyone in this group:
- Cannot authenticate with old protocols like NTLM
- Cannot use outdated Kerberos encryption like DES or RC4
- Cannot have their account delegated
 - Has their Kerberos TGTs limited to 4 hour sessions before they need to reauthenticate.
Wherever they login:
- Their credentials are never cached

This is a protection that - for example - is a great security measure for your domain admins and privileged users.

*Note*: Do your reading on this first! This feature (deliberately) breaks NTLMv2 and Kerberos Delegation for users, so be careful to ensure this will not impact users within Protected Users.

[Reference Documentation](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group)

### [Built-In] Smartcard Authentication

Passwords are so last year!

Windows has had passwordless login forever; in the form of smart cards. And - fun fact - an ~$80 Yubikey can act as a PIV smartcard.

You can an ADCS certificate enrollment template let your admins self-enrol.
  - Natively in AD, you can even enforce smart-card login for a user across the domain.

No more stealing the “domain admin password”!

This is also a great time to audit Active Directory Certificate Services! Check out [Locksmith](https://github.com/TrimarcJake/Locksmith) by @TrimarcJake.

__References__:
- [Yubikey PIV + ADCS Setup Guide](https://support.yubico.com/hc/en-us/articles/360015654500-Setting-up-Windows-Server-for-YubiKey-PIV-Authentication)

### [Built-in] Authentication Silos

Authentication Silos and Authentication Policies are a new-ish AD feature, which became available in Server 2012 R2.

At the most basic level, they are an account/device firewall build into AD that locks specific accounts to specific machines, or vice-versa.

__References__:
- Authentication Silos build on [Kerberos FAST](https://trustedsec.com/blog/i-wanna-go-fast-really-fast-like-kerberos-fast) and [Kerberos claims, compound authentication and armoring](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831747(v=ws.11)#support-for-claims-compound-authentication-and-kerberos-armoring).
- The documentation for 'Authentication Policies and Authentication Policy Silos' can be found [here](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/authentication-policies-and-authentication-policy-silos).
- For a getting-started guide on configuring Authentication Silos, check out this [step by step guide](https://fitzwindowsblog.blogspot.com/2024/05/step-by-step-guide-to-setting-up.html).
- Another great reference is [Protecting Tier 0 the Modern Way](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/protecting-tier-0-the-modern-way/ba-p/4052851) from the [Microsoft `Core Infrastructure and Security Blog`](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/bg-p/CoreInfrastructureandSecurityBlog)

### [Built-in] gMSAs

Many organisations have "user"-type service accounts sitting around with passwords that never change; if the password leaks, it’s game over.

Many of these accoutns are ripe for [Kerberoasting](https://www.crowdstrike.com/cybersecurity-101/kerberoasting/) and [PtH](https://www.crowdstrike.com/cybersecurity-101/pass-the-hash/) attacks.

AD has a special type of account called a gMSA (Group Managed Service Account)
- One or more computers can be delegated to read the encrypted password from AD
- AD rotates the password monthly to 256 random characters

These accounts are set & forget, and can be used in AD environments where an account is required for “service” or “background” jobs.

They even be used to login to things like databases!

[Reference Documentation](https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)

### [Third-Party] Lithnet AD Password Protection

Stop your users setting bad passwords!

Proactively protect users by ensuringe users pick strong passwords.

[Lithnet Password Protection for Active Directory (LPP)](https://github.com/lithnet/ad-password-protection/) is a free (MIT) tool that can enforce password quality - both at the time of set & change - in a number of ways:
- Every password change is compared against the a local copy of the HIBP list (nearly a billion known bad passwords!)
- Words can be banned:
  - “P4$$w0rd” gets turned back into “password”
  - You can ban passwords based off a single dictionary word, as well as things like a company name
  - e.g., “M0na$h2023” doesn’t cut the mustard
- Complexity Rules:
  - The longer your password, the less complex it needs to be
  - Encourages users to pick strong passphrases

### [Built-in] LAPS

Windows LAPS is a feature that automatically manages and backs up the password of a local Administrator account on AD or Entra-joined devices.

Local admin passwords are automatically backed-up to the relevant authority (AD/Entra), and are accessible by delegated administrators.

Using LAPS to regularly rotate and manage local administrator account passwords helps protect against pass-the-hash and lateral-movement attacks

While you're here, make sure to enforce [Network Level Authentication](https://admx.help/?Category=Windows_10_2016&Policy=Microsoft.Policies.TerminalServer::TS_USER_AUTHENTICATION_POLICY) to prevent local accounts from being usable over the network.

[Reference Documentation](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)

### [Third-Party] Lithnet Access Manager

> [Lithnet Access Manager](https://docs.lithnet.io/ams/) (AMS) is a tool that allows you to safely delegate sensitive administrative access to computers in your organization in a modern and user-friendly way
>
> It provides a web-based interface that allows users to request local admin/root passwords, BitLocker recovery keys, and grant just-in-time administrative access to their own accounts.
>
> It is fully compatible and works out-of-the-box with Microsoft LAPS, but also comes with its own agent, which expands LAPS coverage to Azure AD joined and registered devices, as well as macOS and Linux devices.

__LAPS__: AMS allows users to retrieve LAPS apsswords over the web, using modern OIDC authentication and MFA.

__JIT__: AMS allows users to request "just-in-time" access to systems, preventing persistence of administrative privileges across the domain.
