# Authentication Silos

Authentication Silos and Authentication Policies are a new-ish AD feature. By “new-ish”, we mean they became available in Server 2012 R2.

At the most basic level, they are essentially an account/device firewall build into AD that locks specific accounts to specific machines, or vice-versa.

In the MEAM, we have implemented Authentication Silos and Policies in two ways:
- __PAW Silos__: Per-tier "computer" silos that contain PAW machines.
  - Only PAW Users in a specific tier can authenticate.
- __Zone Silos__: Per-zone "user" silos that contain user and service accounts.
  - Users in a zone can only authenticate to:
    - PAW devices
    - Computers within their own zones
    - *DA Silos only*: Domain Controllers

## References
- Authentication Silos build on [Kerberos FAST](https://trustedsec.com/blog/i-wanna-go-fast-really-fast-like-kerberos-fast) and [Kerberos claims, compound authentication and armoring](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831747(v=ws.11)#support-for-claims-compound-authentication-and-kerberos-armoring).
- The documentation for 'Authentication Policies and Authentication Policy Silos' can be found [here](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/authentication-policies-and-authentication-policy-silos).
- For a getting-started guide on configuring Authentication Silos, check out this [step by step guide](https://fitzwindowsblog.blogspot.com/2024/05/step-by-step-guide-to-setting-up.html).
- Another great reference is [Protecting Tier 0 the Modern Way](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/protecting-tier-0-the-modern-way/ba-p/4052851) from the [Microsoft `Core Infrastructure and Security Blog`](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/bg-p/CoreInfrastructureandSecurityBlog)


## Example

The following script is an example of how to create Authentication Policies and Authentication Policy Silos to MEAM design:

- __User Silos__ are used to lock a set of accounts (either "user" or gMSA accounts) to a specific set of computers. When enforced, the Domain Controller will prevent these users from authenticating on machines not specificed in the silo.
- __PAW Silos__ are used to enforce that only users in a specific tier can authenticate to PAW devices in that tier. Like user silos, when enforced, the Domain Controller will prevent unauthorized users authenticating to PAW devices.

```powershell
# Create a per-zone "User" Authentication Policy / Authentication Policy Silo.
function Create-UserSilo {
    param(
        [string]$TierID,
        [string]$ZoneID,
        [string[]]$DestinationComputerSIDs,
        [switch]$Enforced,
        [switch]$DASilo
    )

    $Destinations = ($DestinationComputerSIDs | %{ "SID($_)"}) -join ","

    if(($TierID -eq "0") -and ($DASilo)) {
        $Prefix = "Z$TierID$ZoneID-DA"
        $UserAllowedToAuthenticateFrom = "O:SYG:SYD:(XA;OICI;CR;;;WD;(Member_of_any {$($Destinations)}))"
        $ServiceAllowedToAuthenticateFrom = $Null
    } else {
        $Prefix = "Z$TierID$ZoneID-User"
        $UserAllowedToAuthenticateFrom = "O:SYG:SYD:(XA;OICI;CR;;;WD;(Member_of_any {$($Destinations)}))"
        $ServiceAllowedToAuthenticateFrom = "O:SYG:SYD:(XA;OICI;CR;;;WD;(Member_of_any {$($Destinations)}))"
    }

    $PolicyParameters = @{
        Name = "$($Prefix)Policy"
        Enforced = $Enforced
        ProtectedFromAccidentalDeletion = $True
        RollingNTLMSecret = "Disabled"
        UserAllowedToAuthenticateFrom = $UserAllowedToAuthenticateFrom
        ServiceAllowedToAuthenticateFrom = $ServiceAllowedToAuthenticateFrom
        Server = $DC
    }

    $Policy = New-ADAuthenticationPolicy @PolicyParameters
    $PolicyDN = (Get-ADADAuthenticationPolicy -Identity $PolicyParameters.Name -Server $DC).distinguishedName

    $SiloParameters = @{
        Name = "$($Prefix)Silo"
        Enforced = $Enforced
        ProtectedFromAccidentalDeletion = $True
        OtherAttributes = @{
            "msDS-ComputerAuthNPolicy" = "$($PolicyDN)"
            "msDS-ServiceAuthNPolicy" = "$($PolicyDN)"
            "msDS-UserAuthNPolicy" = "$($PolicyDN)"
        }
        Server = $DC
    }

    New-ADAuthenticationPolicySilo @SiloParameters
}

# Create a per-tier "PAW" Authentication Policy / Authentication Policy Silo
function Create-PAWSilo {
    param(
        [string]$TierID,
        [string]$PawUserGroupSID,
        [switch]$Enforced,
    )

    if($TierID -eq "0") {
        # Allow DAs on Tier 0 PAWs
        $ComputerAllowedToAuthenticateTo = "O:SYG:SYD:(XA;OICI;CR;;;WD;(Member_of_any {SID($PawUserSID),SID(DA)}))"
    } else {
        $ComputerAllowedToAuthenticateTo = "O:SYG:SYD:(XA;OICI;CR;;;WD;(Member_of_any {SID($PawUserSID)}))"
    }

    $Prefix = "T$TierID-PAW"
    $PolicyParameters = @{
        Name = "$($Prefix)Policy"
        Enforced = $Enforced
        ProtectedFromAccidentalDeletion = $True
        RollingNTLMSecret = "Disabled"
        ComputerAllowedToAuthenticateTo = $ComputerAllowedToAuthenticateTo
        Server = $DC
    }

    $Policy = New-ADAuthenticationPolicy @PolicyParameters
    $PolicyDN = (Get-ADADAuthenticationPolicy -Identity $PolicyParameters.Name -Server $DC).distinguishedName

    $SiloParameters = @{
        Name = "$($Prefix)Silo"
        Enforced = $Enforced
        ProtectedFromAccidentalDeletion = $True
        OtherAttributes = @{
            "msDS-ComputerAuthNPolicy" = "$($PolicyDN)"
            "msDS-ServiceAuthNPolicy" = "$($PolicyDN)"
            "msDS-UserAuthNPolicy" = "$($PolicyDN)"
        }
        Server = $DC
    }

    New-ADAuthenticationPolicySilo @SiloParameters
}

# Create PAW silos in each tier
Create-PAWSilo -TierID "0" -PawUserGroupSID (Get-ADGroup "T0-PAWUsers").SID
Create-PAWSilo -TierID "1" -PawUserGroupSID (Get-ADGroup "T1-PAWUsers").SID
Create-PAWSilo -TierID "2" -PawUserGroupSID (Get-ADGroup "T2-PAWUsers").SID

# Create per-zone user silos
Create-UserSilo -TierID "0" -ZoneID "A" -DASilo -DestinationComputerSIDs @(
    (Get-ADGroup "T0-PAWComputers").SID
    (Get-ADGroup "Z0A-ZoneComputers").SID
    (Get-ADGroup "Domain Controllers").SID
)
Create-UserSilo -TierID "0" -ZoneID "A" -DestinationComputerSIDs @(
    (Get-ADGroup "T0-PAWComputers").SID
    (Get-ADGroup "Z0A-ZoneComputers").SID
)

Create-UserSilo -TierID "1" -ZoneID "A" -DestinationComputerSIDs @(
    (Get-ADGroup "T1-PAWComputers").SID
    (Get-ADGroup "Z1A-ZoneComputers").SID
)
Create-UserSilo -TierID "2" -ZoneID "A" -DestinationComputerSIDs @(
    (Get-ADGroup "T2-PAWComputers").SID
    (Get-ADGroup "Z2A-ZoneComputers").SID
)

```
