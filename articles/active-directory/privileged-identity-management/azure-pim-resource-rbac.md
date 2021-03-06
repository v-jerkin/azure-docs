---
title: View activity and audit history for Azure resource roles in PIM - Azure Active Directory | Microsoft Docs
description: View activity and audit history for Azure resource roles in Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''

ms.assetid:
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 01/24/2019
ms.author: rolyon

ms.collection: M365-identity-device-management
---
# View activity and audit history for Azure resource roles in PIM

With Azure Active Directory (Azure AD) Privileged Identity Management (PIM), you can view activity, activations, and audit history for Azure resources roles within your organization. This includes subscriptions, resource groups, and even virtual machines. Any resource within the Azure portal that leverages the Azure role-based access control (RBAC) functionality can take advantage of the security and lifecycle management capabilities in PIM.

## View activity and activations

To see what actions a specific user took in various resources, you can view the Azure resource activity that's associated with a given activation period.

1. Open **Azure AD Privileged Identity Management**.

1. Click **Azure resources**.

1. Click the resource you want to view activity and activations for.

1. Click **Roles** or **Members**.

1. Click a user.

    You see a graphical view of the user's actions in Azure resources by date. It also shows the recent role activations over that same time period.

    ![User details](media/azure-pim-resource-rbac/rbac-user-details.png)

1. Click a specific role activation to see details and corresponding Azure resource activity that occurred while that user was active.

    ![Select role activation](media/azure-pim-resource-rbac/rbac-user-resource-activity.png)

## Export role assignments with children

You may have a compliance requirement where you must provide a complete list of role assignments to auditors. PIM enables you to query role assignments at a specific resource, which includes role assignments for all child resources. Previously, it was difficult for administrators to get a complete list of role assignments for a subscription and they had to export role assignments for each specific resource. Using PIM, you can query for all active and eligible role assignments in a subscription including role assignments for all resource groups and resources.

1. Open **Azure AD Privileged Identity Management**.

1. Click **Azure resources**.

1. Click the resource you want to export role assignments for, such as a subscription.

1. Click **Members**.

1. Click **Export** to open the Export membership pane.

    ![Export membership pane](media/azure-pim-resource-rbac/export-membership.png)

1. Click **Export all members** to export all role assignments in a CSV file.

    ![Export CSV file](media/azure-pim-resource-rbac/export-csv.png)

## View resource audit history

Resource audit gives you a view of all role activity for a resource.

1. Open **Azure AD Privileged Identity Management**.

1. Click **Azure resources**.

1. Click the resource you want to view audit history for.

1. Click **Resource audit**.

1. Filter the history using a predefined date or custom range.

    ![Filter resource audit](media/azure-pim-resource-rbac/rbac-resource-audit.png)

1. For **Audit type**, select **Activate (Assigned + Activated)**.

    ![Activity detail](media/azure-pim-resource-rbac/rbac-audit-activity.png)

1. Under **Action**, click **(activity)** for a user to see that user's activity detail in Azure resources.

    ![User activity detail](media/azure-pim-resource-rbac/rbac-audit-activity-details.png)

## View my audit

My audit enables you to view your personal role activity.

1. Open **Azure AD Privileged Identity Management**.

1. Click **Azure resources**.

1. Click the resource you want to view audit history for.

1. Click **My audit**.

1. Filter the history using a predefined date or custom range.

    ![Personal role activity](media/azure-pim-resource-rbac/my-audit-time.png)

## Next steps

- [Assign Azure resource roles in PIM](pim-resource-roles-assign-roles.md)
- [Approve or deny requests for Azure resource roles in PIM](pim-resource-roles-approval-workflow.md)
- [View audit history for Azure AD roles in PIM](pim-how-to-use-audit-log.md)
