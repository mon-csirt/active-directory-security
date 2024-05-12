# Delegations

The following table describes the different delegations applied for the Delegation groups described in the MEAM OU structure.

The __DACL__ and __Inheritance__ columns below describe arguments to `dsacls`, for granting a *group* certain rights to an *OU*.

---

<table>
<thead>
    <th scope="col">Class</th>
    <th scope="col">Name</th>
    <th scope="col">Description</th>
    <th scope="col">DACL</th>
    <th scope="col">Inheritance</th>
    <th scope="col">DACL Description</th>
</thead>

<tbody>
<!-- Computers -->

<!-- ComputerDomainJoin -->
<tr>
    <th scope="row" rowspan="2">Computers</th>
    <td rowspan="2">ComputerDomainJoin</td>
    <td rowspan="2">
    Join a fresh computer to the domain
    </td>
    <td><pre>CCDC;computer;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Create and delete child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;computer</pre></td>
    <td><pre>[None]</pre></td>
    <td>Create and delete child computer objects in the current OU</td>
</tr>

<!-- ComputerDomainReJoin -->
<tr>
    <th scope="row" rowspan="2">Computers</th>
    <td rowspan="2">ComputerDomainReJoin</td>
    <td rowspan="2">
    Rejoin a computer to the domain (should be paired with <i>ComputerDomainJoin</i>)
    </td>
    <td><pre>CA;Reset Password;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Reset machine account password on child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>WP;account restrictions;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write UAC on child computer objects in descendant OUs</td>
</tr>

<!-- ComputerRW -->
<tr>
    <th scope="row" rowspan="7">Computers</th>
    <td rowspan="7">ComputerRW</td>
    <td rowspan="7">
    Manage Computer object
    </td>
    <td><pre>GRSDDT;;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Generic read, delete, delete subtree</td>
</tr>
<tr>
    <td><pre>WP;name;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'name' on child computer objects in descendant OUs (allow move)</td>
</tr>
<tr>
    <td><pre>WP;cn;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'cn' on child computer objects in descendant OUs (allow move)</td>
</tr>
<tr>
    <td><pre>WP;samAccountName;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'samAccountName' on child computer objects in descendant OUs (allow rename)</td>
</tr>
<tr>
    <td><pre>WP;description;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'description' on child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;computer;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Create and delete child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;computer</pre></td>
    <td><pre></pre></td>
    <td>Create and delete child computer objects in the current OU</td>
</tr>

<!-- ComputerMove -->
<tr>
    <th scope="row" rowspan="5">Computers</th>
    <td rowspan="5">ComputerMove</td>
    <td rowspan="5">
    Move Computer objects
    </td>
    <td><pre>CCDC;computer;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Create and delete child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;computer</pre></td>
    <td><pre></pre></td>
    <td>Create and delete child computer objects in the current OU</td>
</tr>
<tr>
    <td><pre>GRSDDT;;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Generic read, delete, delete subtree</td>
</tr>
<tr>
    <td><pre>WP;name;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'name' on child computer objects in descendant OUs (required for moving objects)</td>
</tr>
<tr>
    <td><pre>WP;cn;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'cn' on child computer objects in descendant OUs (required for moving objects)</td>
</tr>

<!-- ComputerUacWrite -->
<tr>
    <th scope="row" rowspan="1">Computers</th>
    <td rowspan="1">ComputerUacWrite</td>
    <td rowspan="1">
    Manage Computer <i>userAccountControl</i> attribute (enabled/disabled, etc.)
    </td>
    <td><pre>WP;userAccountControl;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'userAccountControl' on child computer objects in descendant OUs</td>
</tr>

<!-- ComputerResetPwd -->
<tr>
    <th scope="row" rowspan="1">Computers</th>
    <td rowspan="1">ComputerResetPwd</td>
    <td rowspan="1">
    Reset Computer Password
    </td>
    <td><pre>CA;Reset Password;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Reset password child computer objects in descendant OUs</td>
</tr>

<!-- LAPSRead -->
<tr>
    <th scope="row" rowspan="2">Computers</th>
    <td rowspan="2">LAPSRead</td>
    <td rowspan="2">
    Read Computer LAPS password and expiry time
    </td>
    <td><pre>RPCA;ms-Mcs-AdmPwd;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read 'ms-Mcs-AdmPwd' on child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>RP;ms-Mcs-AdmPwdExpirationTime;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read 'ms-Mcs-AdmPwdExpirationTime' on child computer objects in descendant OUs</td>
</tr>

<!-- LAPSReset -->
<tr>
    <th scope="row" rowspan="1">Computers</th>
    <td rowspan="1">LAPSReset</td>
    <td rowspan="1">
    Write Computer LAPS expiry time (e.g., force LAPS password reset)
    </td>
    <td><pre>RPWP;ms-Mcs-AdmPwdExpirationTime;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Write 'ms-Mcs-AdmPwdExpirationTime' on child computer objects in descendant OUs</td>
</tr>

<!-- BitlockerRecovery -->
<tr>
    <th scope="row" rowspan="2">Computers</th>
    <td rowspan="2">BitLockerRecovery</td>
    <td rowspan="2">
    Read Computer BitLocker recovery keys
    </td>
    <td><pre>CAGR;;msFVE-RecoveryInformation</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read 'msFVE-RecoveryInformation' on child computer objects in descendant OUs (FVE recovery password)</td>
</tr>
<tr>
    <td><pre>CARP;msTPM-OwnerInformation;computer</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read 'msTPM-OwnerInformation' on child computer objects in descendant OUs (TPM owner information)</td>
</tr>

<!-- GPOModelling -->
<tr>
    <th scope="row" rowspan="2">Computers</th>
    <td rowspan="2">GPOModelling</td>
    <td rowspan="2">
    Execute GPO Modelling against a Computer OU
    </td>
    <td><pre>CA;Generate resultant set of policy (planning)</pre></td>
    <td><pre>/I:T</pre></td>
    <td>Execute GPO Modelling (planning) on this object (OU) and below</td>
</tr>
<tr>
    <td><pre>CA;Generate resultant set of policy (logging)</pre></td>
    <td><pre>/I:T</pre></td>
    <td>Execute GPO Modelling (logging) on this object (OU) and below</td>
</tr>

<!-- GPOLink -->
<tr>
    <th scope="row" rowspan="1">Computers</th>
    <td rowspan="1">GPOLink</td>
    <td rowspan="1">
    Link GPO to a Computer OU
    </td>
    <td><pre>WPRP;gpLink</pre></td>
    <td><pre>/I:T</pre></td>
    <td>Read/write to the 'gpLink' attribute on this (and descendant) OUs</td>
</tr>

<!-- OrgUnitRW -->
<tr>
    <th scope="row" rowspan="3">Computers</th>
    <td rowspan="3">OrgUnitRW</td>
    <td rowspan="3">
    Create 
    </td>
    <td><pre>CCDC;organizationalUnit;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Create/delete child 'organizationalUnit' objects (one level deep)</td>
</tr>
<tr>
    <td><pre>GRGW;;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read/write on descendant 'organizationalUnit' objects</td>
</tr>
<tr>
    <td><pre>CCDC;organizationalUnit</pre></td>
    <td><pre></pre></td>
    <td>Create/delete child 'organizationalUnit' objects</td>
</tr>

<!-- MemberRW -->
<tr>
    <th scope="row" rowspan="1">Groups</th>
    <td rowspan="1">MemberRW</td>
    <td rowspan="1">
    Add/remove members of Groups
    </td>
    <td><pre>WPRP;member;group</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read/write 'member' attribute of child group objects in descendant OUs</td>
</tr>


<!-- GroupRW -->
<tr>
    <th scope="row" rowspan="3">Groups</th>
    <td rowspan="3">GroupRW</td>
    <td rowspan="3">
    Add and remove groups
    </td>
    <td><pre>GA;;group</pre></td>
    <td><pre>/I:S</pre></td>
    <td>GenericAll rights to child group objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;group;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Create/delete child 'group' objects (one level deep)</td>
</tr>
<tr>
    <td><pre>CCDC;group;</pre></td>
    <td><pre></pre></td>
    <td>Create/delete child 'group' objects </td>
</tr>

<!-- Users -->

<!-- UserRW -->
<tr>
    <th scope="row" rowspan="3">Accounts</th>
    <td rowspan="3">UserRW</td>
    <td rowspan="3">
    Manage User Accounts
    </td>
    <td><pre>GA;;user</pre></td>
    <td><pre>/I:S</pre></td>
    <td>GenericAll on 'user' objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;user;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Create/delete 'user' objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>CCDC;user;</pre></td>
    <td><pre></pre></td>
    <td>Create/delete 'user' objects</td>
</tr>


<!-- ResetPwd -->
<tr>
    <th scope="row" rowspan="3">Accounts</th>
    <td rowspan="3">ResetPwd</td>
    <td rowspan="3">
    Reset Passwords for Accounts
    </td>
    <td><pre>CA;Reset Password;user</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Reset user account password on child computer objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>WPRP;pwdLastSet;user</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read/write 'pwdLastSet' attribute on user objects in descendant OUs</td>
</tr>
<tr>
    <td><pre>WPRP;lockoutTime;user</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read/write 'lockoutTime' attribute on user objects in descendant OUs</td>
</tr>


<!-- GPOLink -->
<tr>
    <th scope="row" rowspan="2">Accounts</th>
    <td rowspan="2">GPOLink</td>
    <td rowspan="2">
    Link GPOs to Account OUs
    </td>
    <td><pre>WPRP;gpLink;</pre></td>
    <td><pre>/I:P</pre></td>
    <td>Read/write to the 'gpLink' attribute on this OUs (this object and one level below only)</td>
</tr>
<tr>
    <td><pre>WPRP;gpLink;organizationalUnit</pre></td>
    <td><pre>/I:S</pre></td>
    <td>Read/write to the 'gpLink' attribute on descendant OUs</td>
</tr>

<!-- GPOModelling -->
<tr>
    <th scope="row" rowspan="2">Accounts</th>
    <td rowspan="2">GPOModelling</td>
    <td rowspan="2">
    GPO Modelling Rights
    </td>
    <td><pre>CA;Generate resultant set of policy (planning)</pre></td>
    <td><pre>/I:T</pre></td>
    <td>Execute GPO Modelling (planning) on this object (OU) and below</td>
</tr>
<tr>
    <td><pre>CA;Generate resultant set of policy (logging)</pre></td>
    <td><pre>/I:T</pre></td>
    <td>Execute GPO Modelling (logging) on this object (OU) and below</td>
</tr>

</tbody>
</table>