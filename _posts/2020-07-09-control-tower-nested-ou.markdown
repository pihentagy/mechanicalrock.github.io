---
layout: post
title: Control Tower and Nested Organizational Units
date: 2020-07-31
tags: account control tower organizations units nested
author: Bret Comstock Waldow
---

## Introduction

AWS provides Control Tower as an approach to organizing and configuring accounts.  Control Tower in turn takes advantage of several other AWS services, including Organizations.  This post is about an approach we wanted to take, what we found, and a solution involving some customisation of an AWS Solution.

## AWS offers several approaches to organizing and configuring resources within Control Tower
# Individual account & Region
  AWS offers and requires users to work in defined accounts and also AWS itself organizes it's resources by Regions - all for AWS's own purpose.  Users must accommodate these approaches and may also take advantage of them for their own purposes - particularly by including related resources into a dedicated account.  But AWS's approach doesn't allow the user to associate several accounts.
# Organizational Unit (OU)
  Control Tower integrates AWS Organizations, which allows gathering accounts into user-defined groups (named Organizational Units - OUs).  Account and Region are Amazon's distinctions in keeping with their architecture, but OUs allow users additional freedom in organizing accounts beyond those structures.
  The user may create and name OUs, and move accounts into and out of them.  The OU functions as an arbitrary association of accounts.

# Nested OUs
  Organizational Units (OUs) must be created within the Organization root, or within another OU.  They may be nested to a depth of 5 levels.

```
example OUs nested 2 levels deep:
  Project1
    Development
    Test
    Production
  Project2
    Development
    Test
    Production
```
  Notice from these examples that some OU names might be duplicated, distinguished only by their parent OU.

  The user may define Service Control Policies (SCP) and apply them to an OU, and any account added to that OU will inherit the SCP - the limits of the SCP will be applied to the account.  Further, the account will inherit any SCPs applied to any parent OUs, up to SCPs applied to the root of the Organization.  

# AWS Control Tower Customization Solution
  AWS provides a central tool to specify and deploy configuration via SCPs definied as JSON files, and resources defined by Cloudformation templates.  The deployment may be targetted by Account, Region, and OU.  This Solution may be found here: [AWS Control Tower Customizations](https://aws.amazon.com/solutions/implementations/customizations-for-aws-control-tower/)<br/>

# A Problem
  At Mechanical Rock we are organizing our accounts into OUs, and nesting them.
  This may lead us to apply a restriction to a child OU that we don't want to apply to the parent OU, and the documentation offers no example of doing this.
  I have confirmed that the AWS Solution does not 'find' OUs by their individual name alone, even if the name is unique within the Organization.  

# Toward a Solution
  AWS pubilshes the source code and I examined that.  I found some mention of nested OUs, but the implementation seemed incomplete, and some exploratory tests confirmed this.
  I forked the existing source and I have opened a [PR with AWS](https://github.com/awslabs/aws-control-tower-customizations/pull/20) with changes.<br/>
  Some existing references in the code suggested ':' as a delimimter character, so I've implemented that.  I provide an example further down.

# Some usage
  Along with other use cases, we are implementing OUs for security and forensic work.  Under the root of the Orgainization, we created an OU named 'Quarantine' and beneath that two OUs - 'Deny-all' and 'Read-only'.
  This is the stanza in the manifest.yaml file which deploys the 'deny-all' SCP - note the nested OU declaration:
  ``` yaml
  organization_policies:
  - name: guardrail-deny-all
    description: To prevent all activity in accounts placed in OU
    policy_file: policies/guardrail-deny-all.json
    # Apply to the following OU(s)
    # Nested OU delimiter: ":"
    apply_to_accounts_in_ou: # :type: list
      - Quarantine:Deny-all
  ```
  Here is the definition of the 'deny-all' SCP:
  ``` json
    {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "denyAll",
              "Effect": "Deny",
              "Action": "*",
              "Resource": "*"
          }
      ]
    }
```

  Deny-all OU - allows us to freeze activity in an account.  This provides a quick way to respond when an account is not behaving as we expect, and doesn't require specific skills beyond using Organizations to move an account.<br/>
  Read-only OU - provides a way to permit inspection of the resources in an account but prevents any new activity.  This is for investigation of problems and unexpected behaviours.
