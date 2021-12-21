# Content
[Intro](#intro)  
[SEMPER Policy-Repository](#policy_repository)  
[SEMPER Policy-Elements](#policy_elements)  
[SEMPER Policy-Scope](#policy_scope)  
[SEMPER Policy-Scope - accountScope](#account_scope)  
[SEMPER Policy-Scope - regionScope](#region_scope)  



# Intro {#intro}
Everything in SEMPER is managed via the policies stored in the SEMPER policy repository in the Core Security account.
The format of the policies is JSON.

#SEMPER Policy-Repository {#policy_repository}
The following folder-structure is required for SEMPER and may not be altered.
In the folders with the "..." you may place your policy-json files.
In case you like to disable policies, just create a further sub-folder (e.g. /disabled) and move the policies you like to disable to there.
```
policy_repository
│   README.md
│
├───10_configure
│   │   securityhub.json
│   ├───config_rules
│   │   │   semper_policy.json
│   │   │   ...
│   │   ├───disabled
│   │   │   disabled_policy.json
│   │   │   ...
│   │   
│   └───event_rules
│       │   semper_policy.json
│       │   ...
│   
├───20_filtering
│   ├───event_rules
│   │   │   semper_policy.json
│   │   │   ...
│   │
│   ├───guardduty_findings
│   │   │   semper_policy.json
│   │   │   ...
│   │
│   ├───securityhub_findings
│   │   │   semper_policy.json
│   │   │   ...
│   
└───30_enrichment
    │   semper_policy.json
    │   ...
```

#SEMPER Policy-Elements {#policy_elements}
The SEMPER Policies always have the following sections
```json {linenos=table,hl_lines=[],linenostart=50}
{
  "metaData" *optional but recommended*: {
    // here you can provide any attributes helping you to organize your policies.
    // e.g. versioning, title, description, policy-type, ownership 
  },
  // policy-type specific section 
  "configure" or "filtering" or "enrichment": {
    "policyScope": *optional* {
    }
    ...
  },
  "metaData" *optional but recommended*: {
    // here you can provide any attributes helping you to organize your policies.
    // e.g. versioning, title, description, policy-type, ownership 
    // 
  }
}
```
## Section "policyScope" {#policy_scope}
You can specify on a finegrained level in which member account and in which AWS region a SEMPER policy should be applied. 
If a member account is in scope you can determine based on account-context like:
- AWS account ID
- OU-ID
- AWS account-tags (managed via the Organization Management Account)

![aws-organization-account-model](docs/aws-organization-account-model.png)

Mandatory: no
All elements of the policyScope-Section are optional.
```json {linenos=table,hl_lines=[],linenostart=50}
{
    ...
    "policyScope" *optional*: {
      "accountScope" *optional*: {
        "exclude" *optional*: {
            ...
        },
        "forceInclude" *optional*: {
            ...
        }
      },
      "regionScope" *optional*: {
        "exclude" *optional*: ['string'],
        "forceInclude" *optional*: ['string']
      }
    }
    ...
}
```

### Sub-Section "accountScope" {#account_scope}
The section "accountScope" allows you to exclude accounts based on specific context-information, which is:
```json {linenos=table,hl_lines=[],linenostart=50}
Excluding based on specific account context-information:
{
    ...
          "accountId" *optional*: ['string'],
          "ouId" *optional*: ['string'],
          "accountTags" *optional*: {
              "tag-key1-string": 'string' tag-value1,
              "tag-key2-string": 'string' tag-value2,
              ...
          }
    ...
}
```

The optional "exclude"-node can have two flavors:
```json {linenos=table,hl_lines=[],linenostart=50}
Excluding based on specific account context-information
{
    ...
      "accountScope" *optional*: {
        "exclude" : {
          "accountId" *optional*: [],
          "ouId" *optional*: [],
          "accountTags" *optional*: {}
        }
    ...
}

or exclude everything independent of account context information
{
    ...
        "exclude" : ["*"]
    ...
}
```

The optional "forceInclude"-node will be applied after the exclude operation. 
So you are able to add specific accounts again:
```json {linenos=table,hl_lines=[],linenostart=50}
Excluding based on specific account context-information
{
    ...
      "accountScope" *optional*: {
        "forceInclude" : {
          "accountId" *optional*: [],
          "ouId" *optional*: [],
          "accountTags" *optional*: {}
        }
      }
    ...
}
```

For example you can "exclude" all accounts and "forceInclude" the Organization Management Account based on the assigned account-tag:
```json {linenos=table,hl_lines=[],linenostart=50}
Sample:
{
    ...
    "policyScope": {
      "accountScope": {
        "exclude" :  {
          "accountId": ["*"]
        },
        "forceInclude" : {
          "accountTags" : {
            "account_name" : [
              "Core Organization Management"
            ]
          }
        }
      }
    }
    ...
}
```
### Sub-Section "regionScope" {#region_scope} 
In the **SEMPER Core Security** module you can specify the target regions to configure AWS Config Rules, AWS Event Rules and Security Hub customizations. 
The section "regionScope" allows you per policy to override this settings using the [AWS region names](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html#Concepts.RegionsAndAvailabilityZones.Regions):


The optional "exclude"-node can have two flavors:

Excluding based on specific account context-information
```json {linenos=table,hl_lines=[],linenostart=50}
{
    ...
    "policyScope" *optional*: {
      ...
      "regionScope" *optional*: {
        "exclude" *optional*: ['string'],
        "forceInclude" *optional*: ['string']
      }
    }
    ...
}
```

For example you can "exclude" all regions and "forceInclude" one specific region (take care that no SCP ios preventing the region):
```json {linenos=table,hl_lines=[],linenostart=50}
Sample:
{
    ...
    "policyScope": {
      "regionScope": {
        "exclude" : ["*"],
        "forceInclude": [
          "us-east-1"
        ]
      }
    }
    ...
}
```


```json {linenos=table,hl_lines=[],linenostart=50}
{
    "policyScope": {
      "accountScope": {
        "exclude": {
          "accountId": [],
          "ouId": [],
          "accountTags": {}
        },
        "forceInclude": {
          "accountId": [],
          "ouId": [],
          "accountTags": {}
        }
      },
      "regionScope": {
        "exclude": [],
        "forceInclude": []
      }
    }
}
```


#SEMPER Policy-Types <a name="policy_types"></a>
SEMPER distinguishes between different policy types.
- Configure-Policies
- Filter-Policies
- Enrichment-Policies

##Configure-Policies
SEMPER will crawl through all accounts in your AWS Organization and assume the SEMPER Member role in the each account.
Each account-context (account-id, OU-ID, account-tags) will be determined.
Then it will iterate through all policies in the folders: 
/10_configure/config_rules
/10_configure/event_rules

With the "policyScope" in Configure-Policies you can specify if the configure-policy will be applied to the current member account.
The configure-action specified in the policy will only be applied if the optinal policyScope evaluated to "True".
### AWS Config Rule Policies
Folder: /10_configure/config_rules
This policies allow you to provision custom AWS Config Rules to your spoke accounts.
SEMPER uses boto3 [ConfigService.Client.put_config_rule](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/config.html#ConfigService.Client.put_config_rule) for this feature. 

```json {linenos=table,hl_lines=[],linenostart=50}
{
  ...
  "configure": {
    "policyScope": *optional - described in separate chapter* {
    },    
    "configRuleSettings": {
      "configRuleName": 'string' according to boto3 ConfigRuleName-specification - will be prefixed with "semper-",
      "configRuleDescription": 'string' according to boto3 Description-specification,
      "complianceResourceTypes": [
        'string' according to boto3 Socpe.ComplianceResourceTypes-specification
      ],
      "coreSecurityEvalLambdaName": 'string' name of the custom evaluation lambda you provide in the SEMPER Core Security account
    }
  }
  ...
}
```

### AWS EventBridge Rule Policies
Folder: /10_configure/event_rules
This policies allow you to provision custom AWS EventBridge Rules to your spoke accounts.
SEMPER uses boto3 [Events.Client.put_rule](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html#EventBridge.Client.put_rule) for this feature. 

```json {linenos=table,hl_lines=[],linenostart=50}
{
  ...
  "configure": {
    "policyScope" *optional - described in separate chapter*: {
    },    
    "eventSettings": {
      "eventName": 'string' according to boto3 Name-specification - will be prefixed with "semper-",
      "eventDescription": 'string' according to boto3 Description-specification,
      "eventPattern": 'string' following the specification described in the link below
    }  
  }
  ...
}
```
The eventPattern has to follow this AWS specification: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html

##Filter-Policies
Filtering-Policies are applied to all Security-Findings. 
A Filtering-Policy can be equipped with a policyScope section.
