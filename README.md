This is the documentation of the SEMPER configuration and policy language.

Required Semper Module Version: >= 1.0

# Content <a id="top"></a>
- [Intro](#intro)
- [SEMPER Policy-Repository](#policy_repository)
- [SEMPER Policy-Elements](#policy_elements)
  - [SEMPER Policy Syntax](#policy_syntax)
  - [Section "policyScope"](#policy_scope)
    - [Sub-Section "accountScope"](#account_scope)
    - [Sub-Section "regionScope"](#region_scope)
    - [Samples for "policyScope"](#policy_scope_samples)
- [SEMPER Policy-Types](#policy_types)
  - [Configure-Policies](#policy_type_configure)
    - [AWS Config Rule Policies](#policy_type_configure_config)
    - [AWS EventBridge Rule Policies](#policy_type_configure_eventbridge)
    - [AWS Security Hub Policies](#policy_type_configure_securityhub)
  - [Filter-Policies](#policy_type_filter)
    - [Samples of Filter-Policies](#policy_filter_samples)
  - [Instruction-Policies](#policy_type_instructions)
- [SEMPER Use-Cases](#policy_types)

# Intro <a id="intro"></a> [🔝](#top)
Everything in SEMPER is managed via the policies stored in the SEMPER policy repository in the Core Security account.

The technical format of the policies is [JSON](https://en.wikipedia.org/wiki/JSON).

# SEMPER Policy-Repository <a id="policy_repository"></a> [🔝](#top)
SEMPER distinguishes between different policy types that will be described in later chapters:
- [Configure-Policies](#policy_type_configure)
- [Filter-Policies](#policy_type_filter)
- [Enrichment-Policies](#policy_type_enrichment)

The following folder-structure is required for SEMPER and may not be altered.

There is one JSON file per policy.

In the folders marked with "..." you may place your own policies as JSON files in the respective folder. Please ensure to use the right policy type for the right folder.
If you want to disable policies, just create another subfolder (e.g. /disabled) and move the (given) policies you want to disable there.
```
policy_repository/
│   README.md
│
├───10_configure/
│   │   securityhub.json
│   ├───config_rules/
│   │   │   semper_configure_policy.json
│   │   │   ...
│   │   │
│   │   ├───disabled/
│   │   │   semper_disabled_policy.json
│   │   │   ...
│   │
│   └───event_rules/
│       │   semper_configure_policy.json
│       │   ...
│
├───20_filtering/
│   ├───cloudtrail_api_calls/
│   │   │   semper_filter_policy.json
│   │   │   ...
│   │
│   ├───guardduty_findings/
│   │   │   semper_filter_policy.json
│   │   │   ...
│   │
│   ├───securityhub_findings/
│   │   │   semper_filter_policy.json
│   │   │   ...
│
└───30_enrichment/
    │   semper_enrichment_policy.json
    │   ...
```

# SEMPER Policy-Elements <a id="policy_elements"></a> [🔝](#top)
The SEMPER Policies always have the following sections:
```json {linenos=table,hl_lines=[],linenostart=50}
{
  "metaData": {...},
  "configure" | "filtering" | "enrichment": {
    "policyScope": {...},
    ...
    <typeSpecificSection>
    ...
  },
  "auditing": {...}
}
```
| Key        | Value-Type | Comment |
| :---       | :---  | :---  |
| metaData   | object | (optional but recommended): provide here any attributes helping you to organize your policies. <br> *e.g. versioning, title, description, policy-type, ownership*   |
| configure *or* <br> filtering *or* <br> enrichment | object | determines the [type](#policy_types) of the policy  |
| .policyScope | object | (optional) as described in the following chapter [Section policyScope](#policy_scope). |
| \<typeSpecificSection\> | object | will be described in the chapter [SEMPER Policy-Types](#policy_types). |
| auditing    | object | (optional but recommended) provide here any attributes helping you to audit and reasses your policies. <br> *e.g. lastAttestationDate, contact-details of auditor*  |

## SEMPER Policy Syntax <a id="policy_syntax"></a> [🔝](#top)
The JSON objects *policyScope* and *typeSpecificSection* allow a SEMPER syntax that is in alignment with: [Amazon EventBridge > Create event patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html#eb-create-pattern)
| Comparison | Example | Rule syntax | Matching source example |
| :---   | :---  | :---  | :---  |
| Empty | LastName is empty | "LastName": [""] | "LastName": "" |
| Equals | Name is "Alice" | "Name": [ "Alice" ] | "Name": "Alice" |
| And | Location is "New York" and Day is "Monday" | "Location": [ "New York" ], "Day": ["Monday"] | "Location": "New York", "Day": "Monday" |
| Or | PaymentType is "Credit" | "Debit" | "PaymentType": [ "Credit", "Debit"] | "PaymentType": "Credit" |
| Not (1) | Weather is anything but "Raining" or "Cloudy"| "Weather": [ { "anything-but": [ "Raining", "Cloudy" ] } ] | "Weather": "Sunny" |
| Not (2) | Weather is anything but "Raining" | "Weather": [ { "anything-but": "Raining" } ] | "Weather": "Sunny" or "Cloudy" |
| Begins with | Region is in the US | "Region": [ {"prefix": "us-" } ] | "Region": "us-east-1" |
| Exists | ProductName exists | "ProductName": [ { "exists": true } ] | "ProductName": "SEMPER" |
| Does not exist | ProductName does not exist | "ProductName": [ { "exists": false } ] | n/a |
<br>

*Pending* in SEMPER:
| Comparison | Example | Rule syntax | Matching source example  |
| :---   | :---  | :---  | :---  |
| Null | UserID is null | "UserID": [ null ] |
| Numeric (equals) | Price is 100 | "Price": [ { "numeric": [ "=", 100 ] } ] |
| Numeric (range) | Price is more than 10, and less than or equal to 20 | "Price": [ { "numeric": [ ">", 10, "<=", 20 ] } ] |
<br>

In SEMPER the keys of the findingPattern are **case-insensitive** to the source JSON. Additionally SEMPER supports the following syntax:
| Comparison | Example | Rule syntax | Matching source example |
| :---   | :---  | :---  |:---  |
| Ends with | Dev-System | "serviceName": [ {"suffix": "-dev" } ] | "ServiceName": "employee-database-dev" |
| Contains | ServiceName contains 'database' | "ServiceName": [ {"contains": "database" } ] |  "serviceName": "employee-database-dev" |
| Does not contain | ServiceName does not contain 'database' | "ServiceName": [ {"contains-not": "database" } ] |  "serviceName": "employee-microservice-dev" |



## Section "policyScope" <a id="policy_scope"></a> [🔝](#top)
You can specify on a finegrained level in which member account and in which AWS region a SEMPER policy should be applied.

![aws-organization-account-model](docs/aws-organization-account-model.png)

The *policyScope-Section* allows you to specify
- an **account-scope** given through *AWS account ID*, *OU-ID* and *AWS account-tags* (managed via the Organization Management Account)
- a **region-scope** given through the *region code* of an AWS resource.

```json {linenos=table,hl_lines=[],linenostart=50}
{
    ...
    "policyScope": {
      "accountScope": {
        "exclude": "*" | {...},
        "forceInclude": {...}
      },
      "regionScope": {
        "exclude": "*" | ['string'],
        "forceInclude": ['string']
      }
    }
    ...
}
```
| Key             | Value-Type             | Comment |
| :---            | :---                   | :---  |
| policyScope     | object                 | necessary for each new rule |
| .accountScope   | object                 | (optional) see [accountScope](#account_scope) |
| . .exclude      | "*" or object          | (optional) |
| . .forceInclude | object                 | (optional) |
| .regionScope    | object                 | (optional) see [regionScope](#region_scope) |
| . .exclude      | "*" or array of string | (optional) |
| . .forceInclude | array of string        | (optional) |

### Sub-Section "accountScope" <a id="account_scope"></a> [🔝](#top)
If a member account should be in scope can be defined based on the account-context information:
- AWS account ID
- OU-ID
- AWS account-tags (managed via the Organization Management Account)

The section *accountScope* allows you to **exclude** accounts and in a second step to **forceInclude** accounts based on specific account-context information (similar approach as the tool *rsync* uses to exclude and include files and directories):
```json {linenos=table,hl_lines=[],linenostart=50}
{
      ...
      "accountScope": {
        "exclude": "*" | {
          "accountId": ['string'],
          "ouId": ['string'],
          "accountTags": {
              'tag-key-1-string': ['tag-value-1-string'],
              'tag-key-2-string': ['tag-value-2-string'],
              ...
          }
        },
        "forceInclude": {
          "accountId": ['string'],
          "ouId": ['string'],
          "accountTags": {
              'tag-key-3-string': ['tag-value-3-string'],
              'tag-key-4-string': ['tag-value-4-string'],
              ...
          }
        }
      }
      ...
}
```
| Key               | Value-Type      | Comment |
| :---              | :---            | :---  |
| .accountScope     | object          | (optional) first the optional **exclude**-section is evaluated, then the optional **forceInclude** section. |
| . .exclude        | "*" or object   | (optional) the elements in this section are evaluated using a *logical AND*. |
| . . .accountId    | array of string | (optional) 12-digit AWS account ID. The elements in this array are evaluated using a *logical OR*. |
| . . .ouId         | array of string | (optional) the elements in this array are evaluated using a *logical OR*. |
| . . .accountTags  | dict            | (optional) the elements in this section are evaluated using a *logical AND*. |
| . . . .tag-key-1  | array of string | Value of account-tag 1. The elements in this array are evaluated using a *logical OR*. |
| . . . .tag-key-2  | array of string | Value of account-tag 2. The elements in this array are evaluated using a *logical OR*. |
| . .forceInclude   | object          | (optional) here you can specify account-context information used to include accounts to the scope. <br> Already excluded accounts can be re-added again with this section. <br> The elements in this section are evaluated using a *logical AND*. |
| . . .accountId    | array of string | (optional) 12-digit AWS account ID. The elements in this array are evaluated using a *logical OR*. |
| . . .ouId         | array of string | (optional) the elements in this array are evaluated using a *logical OR*. |
| . . .accountTags  | dict            | (optional) the elements in this section are evaluated using a *logical AND*. |
| . . . .tag-key-3  | array of string | Value of account-tag 3. The elements in this array are evaluated using a *logical OR*. |
| . . . .tag-key-4  | array of string | Value of account-tag 4. The elements in this array are evaluated using a *logical OR*. |

### Sub-Section "regionScope" <a id="region_scope"></a> [🔝](#top)
In the **SEMPER Core Security** module you can specify the target regions where SEMPER will provision *AWS Config Rules*, *AWS EventBus Rules* and customize *AWS Security Hub* in the member accounts.
The section **regionScope** allows you per policy to override this settings using the [AWS region names](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html#Concepts.RegionsAndAvailabilityZones.Regions) (e.g. eu-central-1, us-west-2, more examples see below):

```json {linenos=table,hl_lines=[],linenostart=50}
{
      ...
      "regionScope": {
        "exclude": "*" | ['string'],
        "forceInclude": ['string']
      }
      ...
}
```
| Key            | Value-Type | Comment |
| :---           | :---  | :---  |
| regionScope   | object | (optional) first the optional **exclude**-section is evaluated, then the optional **forceInclude** section. |
| .exclude      | "*" or array of string | (optional) the elements in this section are evaluated using a *logical OR*. |
| .forceInclude | array of string | (optional) the elements in this section are evaluated using a *logical OR*. |

### Samples for "policyScope" <a id="policy_scope_samples"></a> [🠕](#top)

Sample 1: Exclude all regions and include "us-east-1" only.
Via **exclude** all regions will be ignored and via **forceInclude** one specific region will be put into scope again (take care that [SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) might prevent resources in regions):
```json {linenos=table,hl_lines=[],linenostart=50}
Sample:
{
    ...
    "policyScope": {
      "regionScope": {
        "exclude": "*",
        "forceInclude": [
          "us-east-1"
        ]
      }
    }
    ...
}
```

Sample 2: Include all accounts that have account-tag "AccountName" starting with "Core".<br>
With th **exclude** section we ignore all possible account-contexts and with the **forceInclude** we include all accounts that have a tag-value prefixed with "Core" for the tag-key "AccountName".
```json {linenos=table,hl_lines=[],linenostart=50}
{
  ...
  "policyScope": {
    "accountScope": {
      "exclude": "*",
      "forceInclude": {
        "accountTags": {
          "AccountName" : [
            {
              "prefix": "Core"
            }
          ]
        }
      }
    }
  }
  ...
}
```

Sample 3: Include all accounts that have account-tag "Environment" != "Production"
```json {linenos=table,hl_lines=[],linenostart=50}
This
{
  ...
  "policyScope": {
    "accountScope": {
      "exclude": "*",
      "forceInclude": {
        "accountTags": {
          "Environment" : [
            {
              "anything-but": "Production"
            }
          ]
        }
      }
    }
  }
  ...
}
is the same like:
{
  ...
  "policyScope": {
    "accountScope": {
      "exclude": {
        "accountTags": {
          "Environment" : [
            "Production"
          ]
        }
      }
    }
  }
  ...
}
```

# SEMPER Policy-Types <a id="policy_types"></a> [🔝](#top)
SEMPER distinguishes between different policy types:
- [Configure-Policies](#policy_type_configure)
- [Filter-Policies](#policy_type_filter)
- [Enrichment-Policies](#policy_type_enrichment)

## Configure-Policies <a id="policy_type_configure"></a> [🔝](#top)
SEMPER will crawl through all accounts in your AWS Organization and assume the SEMPER Member role in the each account.
Each account-context (AWS account id, OU-ID, AWS account tags) will be determined.

Then SEMPER will iterate through all policies in the folders:
> Folder: /10_configure/config_rules <br>
> Folder: /10_configure/event_rules

With the optional [policyScope](policy_scope) you can specify, if the configure policy will be applied to the member account.
The configure-action specified in the policy will only be applied if the optional policyScope evaluates to `true`.

### AWS Config Rule Policies <a id="policy_type_configure_config"></a> [🔝](#top)
> Folder: /10_configure/config_rules

This policies allow you to specify the provisioning of custom AWS Config Rule to your member accounts.
SEMPER uses boto3 [ConfigService.Client.put_config_rule](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/config.html#ConfigService.Client.put_config_rule) for this feature.

```json {linenos=table,hl_lines=[],linenostart=50}
{
  ...
  "configure": {
    "policyScope": {...},
    "configRuleSettings": {
      "configRuleName": 'string',
      "configRuleDescription": 'string',
      "complianceResourceTypes": ['string'],
      "sourceOwner": "CUSTOM_LAMBDA" | "AWS",
      "sourceIdentifier": 'string',
      "inputParameters ": 'string'
    }
  }
  ...
}
```
| Key                      | Value-Type | Comment |
| :---                     | :---  | :---  |
| .policyScope             | object | (optional) as described in this chapter [Section policyScope](#policy_scope). |
| .configRuleSettings      | object | specifying the attributes used for the boto3 call. |
| . .configRuleName        | string | according to boto3 [ConfigRuleName](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/config.html#ConfigService.Client.put_config_rule)-specification - will be prefixed with ***semper-***.|
| . .configRuleDescription | string | (optional) according to boto3 [Description](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/config.html#ConfigService.Client.put_config_rule)-specification.      |
| . .complianceResourceTypes | array of string |  (optional) according to boto3 [Scope.ComplianceResourceTypes](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/config.html#ConfigService.Client.put_config_rule)-specification |
| . .sourceOwner           | string | Either "CUSTOM_LAMBDA" for a custom Lambda function in the Core Security account or "AWS" for AWS managed Config rules. |
| . .sourceIdentifier      | string | If "sourceOwner" is "CUSTOM_LAMBDA", name of the custom evaluation Lambda hosted in the Core Security that is valid for this Config Rule. <br> If "sourceOwner" is "AWS", identifier of the AWS managed rule. List of [AWS Config Managed Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html). E.g. *IAM_PASSWORD_POLICY* |
| . .inputParameters       | string | (optional) A string, in JSON format, that is passed to the Config rule Lambda function. |

### AWS EventBridge Rule Policies <a id="policy_type_configure_eventbridge"></a> [🔝](#top)
> Folder: /10_configure/event_rules

This policies allow you to provision custom AWS EventBridge Rules to your member accounts.
SEMPER uses boto3 [Events.Client.put_rule](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html#EventBridge.Client.put_rule) for this feature.

```json {linenos=table,hl_lines=[],linenostart=50}
{
  ...
  "configure": {
    "policyScope": {...},
    "eventSettings": {
      "eventName": 'string' according to boto3 Name-specification - will be prefixed with "semper-",
      "eventDescription": 'string' according to boto3 Description-specification,
      "eventPattern": 'string' following the specification described in the link below
    }
  }
  ...
}
```
| Key                 | Value-Type | Comment |
| :---                | :---  | :---  |
| .policyScope        | object | (optional) as described in this chapter [Section policyScope](#policy_scope). |
| .eventSettings      | object | specifying the attributes used for the boto3 call. |
| . .eventName        | string | according to boto3 [Name](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html#EventBridge.Client.put_rule)-specification - will be prefixed with "semper-".|
| . .eventDescription | string | according to boto3 [Description](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html#EventBridge.Client.put_rule)-specification. |
| . .eventPattern     | array of string | The eventPattern has to follow this AWS specification: [Amazon EventBridge event patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html) |

### AWS Security Hub Configuration Policies <a id="policy_type_configure_securityhub"></a> [🔝](#top)
will follow


## Filter-Policies <a id="policy_type_filter"></a> [🔝](#top)
SEMPER will aggregate all the security events from the provisioned SEMPER AWS Config Rules, the AWS EventBridge Rules (API calls via CloudTrail) and also the AWS Security Hub- and Amazon GuardDuty findings. SEMPER forwards all those security findings to a SEMPER Processing Lambda in your Core Security account.

This SEMPER Processing Lambda will determine the account-context (OU-ID, AWS account tags) of the originating account based on the account ID.

Depending on the source (AWS Event, AWS Security Hub or Amazon GuardDuty) SEMPER will iterate through all filtering policies in the following folders:
>Folder: /20_filtering/cloudtrail_api_calls <br>
>Folder: /20_filtering/guardduty_findings <br>
>Folder: /20_filtering/securityhub_findings

Based on the defined filter policies in those folders the processing Lambda will filter the the security findings

The generic structure of a SEMPER Filter-Policy looks like this:
```json {linenos=table,hl_lines=[],linenostart=50}
{
  ...
  "filtering": {
    "policyScope": {...},
    "findingPattern": {...}
  }
  ...
}
```
| Key             | Value-Type | Comment |
| :---            | :---  | :---  |
| filtering       | object | |
| .policyScope    | object | (optional) Each Security Finding has an originating account and an AWS region of the emitting resource. <br>As described in this chapter [Section policyScope](#policy_scope) you can limit the application of a SEMPER policy to a specific account- and region-context. |
| .findingPattern | object | According to the [SEMPER Policy Syntax](#policy_syntax). |


### Samples of Filter-Policies <a id="policy_filter_samples"> [🔝](#top)
```json {linenos=table,hl_lines=[],linenostart=50}
{
  "metaData": {
    "domain": "filter",
    "type": "eventbridge_rule",
    "title": "Ignore KMS events for all environments except 'Production'",
    ...
  },
  "filtering": {
    "policyScope": {
      "accountScope": {
        "exclude": "*",
        "forceInclude": {
          "accountTags": {
            "Environment" : [
              {
                "anything-but": "Production"
              }
            ]
          }
        }
      }
    },
    "findingPattern": {
      "detail": {
        "eventSource": [
          "kms.amazonaws.com"
        ],
        "eventName": [
          "DisableKey",
          "ScheduleKeyDeletion"
        ]
      }
    }
  },
  ...
}
```

```json {linenos=table,hl_lines=[],linenostart=50}
{
  "metaData": {
    "domain": "filter",
    "type": "eventbridge_rule",
    "title": "Drop CIS_AWS_1.4 findings for IAM User foundation_checkpoint_reader",
    ...
  },
  "filtering": {
    "findingPattern": {
      "ProductFields": {
        "StandardsGuideArn": [
          {
            "prefix": "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark"
          }
        ],
        "RuleId": [
          "1.4"
        ]
      },
      "Resources": {
        "Id": [
          {
            "suffix": ":user/checkpoint_api"
          }
        ]
      }
    }
  },
  ...
}
```

## Instruction-Policies <a id="policy_type_instructions"></a> [🔝](#top)
Instruction policies allow you a policy based extension of the finding policy with instructions for post-processing stages.

... will follow
