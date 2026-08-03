# CST8919 Lab 3 - Azure Policy Lab

**Student Name**: Anoop Sidhu
**Student ID**: 040984994
**Course**: CST8919 DevOps - Security and Compliance
**Semester**: Summer 2026

## Demo Video

🎥 [Watch Demo Video](https://youtu.be/Hq-BgjvUcZQ)

## Summary

MapleTech Solutions needed guardrails to stop developers from deploying resources outside approved regions, skipping resource tagging, and exposing public IP addresses. This lab implements those guardrails using native **Azure Policy**, grouped into a single **Policy Initiative** and enforced at the resource group scope.

## Policies

### 1. `Only-CanadaCentral`
- **Effect**: Deny
- **What it does**: Restricts resource deployment to the `Canada Central` region only. Any resource targeting a different region (e.g. East US) is blocked at the validation step before creation.
- **Mechanism**: Compares the resource's `location` field against an allowed-locations parameter (array), denying anything not in the list.

### 2. `Require-ProjectName-Tag`
- **Effect**: Deny
- **What it does**: Requires every taggable resource to carry a `ProjectName` tag. Resources deployed without this tag are denied.
- **Mechanism**: Checks whether the `tags['ProjectName']` field exists on the resource; denies if it does not.
- **Note**: This policy evaluates *every* taggable resource type in the scope, not just the "headline" resource being deployed — see Challenges below.

### 3. `Deny-Public-IP`
- **Effect**: Deny
- **What it does**: Prevents creation of any Public IP address resource in the resource group, whether created directly or as a dependency of another resource (e.g. a VM).
- **Mechanism**: Matches on resource `type` equal to `Microsoft.Network/publicIPAddresses` and denies unconditionally.


## Challenges and Lessons Learned

- **Policy propagation delay**: After assigning the initiative, enforcement did not take effect immediately. Azure Policy assignments can take up to 15–30 minutes to fully propagate before deny rules reliably apply, which initially made it look like the policies weren't working.

- **Tagging cascades to dependent resources**: The mandatory tagging policy doesn't just apply to the resource you're explicitly creating — it applies to every taggable resource in scope. When deploying a VM through the portal, Azure auto-creates dependent resources (VNet, NIC, NSG, disk) alongside it. Tags set on the VM in the creation wizard aren't automatically propagated to these dependent resources, so the VNet or NIC could get denied for missing the `ProjectName` tag even when the VM itself was correctly tagged. The fix was to pre-create and tag the VNet/NIC separately, then reference them during VM creation, rather than letting Azure auto-generate untagged networking resources.