# Microsoft Entra ID – Identity Lifecycle & Azure RBAC Lab

## Overview
Hands-on Microsoft Entra ID lab demonstrating identity lifecycle management, attribute-based group automation, least-privilege Azure RBAC, and secure offboarding across Finance, HR, and IT.

The project was designed to build practical SC-300 and entry-level IAM skills by connecting identity attributes to group membership and group membership to authorization.

## Business Scenario
A fictional organization needs to manage employee access across Finance, HR, and IT without manually maintaining every user's permissions. The IAM design must:

- Automatically place users into department groups based on identity attributes.
- Apply least-privilege access to Azure resources through group-based RBAC.
- Support employee transfers without manual group cleanup.
- Support secure offboarding by disabling authentication, revoking sessions, removing direct privilege, and deprovisioning department access.

## Technologies
- Microsoft Entra ID
- Microsoft Entra admin center
- Azure portal
- Dynamic security groups
- Azure Role-Based Access Control (RBAC)

## Architecture

```text
                 Microsoft Entra ID
                        |
                  User identities
                        |
              Department attributes
                        |
          +-------------+-------------+
          |             |             |
       Finance          HR            IT
          |             |             |
          v             v             v
 GRP-Finance-Users  GRP-HR-Users  GRP-IT-Users
     (Dynamic)       (Dynamic)      (Dynamic)
          |
          | Azure RBAC
          v
      Reader Role
          |
          v
 RG-Contoso-Finance
```

## Phase 1 – Identity Attributes
Cloud users were organized into departments by configuring identity attributes such as Department, Company, and Job Title. The `department` attribute became the source used to automate department group membership.

## Phase 2 – Dynamic Security Groups
Three dynamic security groups were configured:

- `GRP-Finance-Users`
- `GRP-HR-Users`
- `GRP-IT-Users`

Example rules:

```text
(user.department -eq "Finance")
(user.department -eq "HR")
(user.department -eq "IT")
```

Microsoft Entra ID reevaluates membership when the underlying user attribute changes.

### Evidence – Dynamic Finance Group
The Finance security group is configured with **Dynamic** membership, allowing membership to follow identity attributes rather than manual assignments.

![GRP-Finance-Users dynamic security group](screenshots/GRP%20Finance%20%202026-08-23%204.35.01%20PM.png)

### Lab Finding
The IT group initially contained more users than expected because older lab accounts already had `Department = IT`. This demonstrated that dynamic membership is attribute-driven and highlighted the importance of identity-data quality.

## Phase 3 – Least-Privilege Azure RBAC
A Finance Azure resource group was used as the authorization target:

```text
RG-Contoso-Finance
```

The Finance dynamic group received the built-in `Reader` role at the resource-group scope:

```text
WHO:  GRP-Finance-Users
WHAT: Reader
WHERE: RG-Contoso-Finance
```

This allows Finance users to view resources without giving them permission to modify resources or delegate access.

### Evidence – Group-Based Reader Assignment
The screenshot below shows `GRP-Finance-Users` selected as the member for the Azure **Reader** role assignment.

![Finance group selected for Azure Reader RBAC](screenshots/create%20and%20review%20RBAC%202026-08-23%204.30.13%20PM.png)

## Phase 4 – Mover Lifecycle Test
An employee transfer was simulated by changing Chris Green's department from Finance to IT.

Before:

```text
Chris Green
Department = Finance
        |
        v
GRP-Finance-Users
```

### Evidence – Before Transfer
Before the mover event, Chris Green and John Miller were members of the Finance group.

![Finance members before mover](screenshots/GRP%20members%202026-08-23%204.43.45%20PM.png)

After the attribute change:

```text
Chris Green
Department = IT
      |
      +--> Automatically removed from GRP-Finance-Users
      |
      +--> Automatically added to GRP-IT-Users
```

### Evidence – After Transfer
After Chris Green's department was changed to IT, he appeared in the IT dynamic group:

![IT members after mover](screenshots/mover%20%202026-08-24%2012.49.22%20AM.png)

The Finance group then showed only John Miller, validating that Chris was automatically removed from Finance membership:

![Finance members after mover](screenshots/Finance%20mover%202026-08-24%2012.51.07%20AM.png)

No group membership was manually changed. Because Finance authorization was group-based, the user's Finance access followed the membership change.

## Phase 5 – Leaver / Offboarding Test
A secure offboarding workflow was tested using a lab user with IT group membership and a directly assigned Microsoft Entra role.

The workflow included:

1. Disabled the account to block new sign-ins.
2. Revoked existing sessions to force reauthentication.
3. Removed the directly assigned privileged role.
4. Cleared the IT department attribute.
5. Verified automatic removal from `GRP-IT-Users` through dynamic membership reevaluation.

### Evidence – Account Disabled
The offboarded lab account shows **Account status: Disabled**, demonstrating the authentication-blocking step of the leaver workflow.

![Offboarded user account disabled](screenshots/Sergeant%20%20account%20status%20%202026-08-24%201.00.29%20AM.png)

### Evidence – Direct Privilege Removed
After privilege cleanup, the user's **Active assignments** view returned **No results**.

![Offboarded user has no active assigned roles](screenshots/sergeant%20role%20assignment%20%202026-08-24%201.09.59%20AM.png)

This demonstrated the difference between blocking authentication and cleaning up authorization.

## Troubleshooting – Entra Roles vs Azure RBAC
During the project, an Entra administrative account could manage directory identities but could not create an Azure resource group.

Investigation showed that the identity had no Azure RBAC assignment on the Azure subscription, while another identity held `Owner` access.

**Key lesson:** Microsoft Entra directory roles and Azure RBAC roles are separate authorization systems. Directory administrative privileges do not automatically grant Azure subscription permissions.

## SC-300 Concepts Practiced
- Microsoft Entra users and identity attributes
- Security groups
- Dynamic user membership
- Group-based access management
- Azure RBAC
- Reader vs Contributor vs Owner
- Least privilege
- Identity lifecycle management
- Joiner / Mover / Leaver concepts
- Session revocation
- Direct privileged-role removal
- Microsoft Entra roles vs Azure resource roles

## Skills Demonstrated
- Microsoft Entra ID administration
- Attribute-based access automation
- Dynamic security groups
- Azure RBAC
- Least-privilege access design
- Identity lifecycle management
- Secure user offboarding
- Privileged-access cleanup
- IAM troubleshooting

## Interview Summary
> Built a Microsoft Entra ID identity lifecycle lab that automated department-based access using dynamic security groups. Implemented group-based Azure RBAC with least-privilege Reader access, validated a Finance-to-IT employee transfer where group membership changed automatically based on the department attribute, and executed a secure offboarding workflow by disabling the account, revoking sessions, removing direct privileged access, and deprovisioning dynamic group membership. Also troubleshot an authorization issue that demonstrated the separation between Microsoft Entra directory roles and Azure subscription RBAC.

## Resume Bullets
- Built an identity lifecycle lab using attribute-based dynamic security groups to automate Finance, HR, and IT access provisioning.
- Implemented group-based Azure RBAC with least-privilege Reader access and validated automated access changes during a Finance-to-IT employee transfer.
- Executed secure offboarding by disabling the user, revoking sessions, removing direct privileged access, and automatically deprovisioning dynamic group membership.
- Troubleshot Azure authorization by identifying the separation between Microsoft Entra directory roles and Azure subscription RBAC permissions.

## Next Projects
Additional Microsoft Entra / SC-300 labs will expand into Conditional Access, MFA, Zero Trust access controls, privileged identity management, and external identity scenarios.
