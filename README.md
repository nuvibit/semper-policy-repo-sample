#Policy-Elements
SEMPER Policies always have the following sections

## Section "policyScope"
Mandatory: no
All elements of the policyScope-Section are potional
```json {linenos=table,hl_lines=[],linenostart=50}
{
    ...
    "policyScope": {
      "accountScope" *optional*: {
        "exclude" *optional*: {
            ...
        },
        "forceInclude" *optional*: {
            ...
        }
      },
      "regionScope *optional*": {
        "exclude" *optional*: ["string"],
        "forceInclude" *optional*: ["string"]
      }
    }
    ...
}
```

### Section "accountScope"
The section "accountScope" allows you to exclude accounts based on specific context-information, which is:
```json {linenos=table,hl_lines=[],linenostart=50}
Excluding based on specific account context-information
{
    ...
          "accountId" *optional*: ["string"],
          "ouId" *optional*: ["string"],
          "accountTags" *optional*: {
              "key1-string": "value1-string",
              "key2-string": "value2-string",
              ...
          }
    ...
}
```

The optional "exclude" node can have two flavors:
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

or exclude everything endependent of account context information
{
    ...
        "exclude" : ["*"]
    ...
}

```




#Policy-Types
##Configure-Policies

##Filter-Policies
Filtering-Policies are applied to all Security-Findings. 
A Filtering-Policy can be equipped with a policyScope section.
This section will limit the application of the Filtering-Policy to findings where the "origin" corresponds to the policyScope.

```json {linenos=table,hl_lines=[],linenostart=50}
{
    ...
    // Optional Section
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
        "forceInclude": [
          "us-east-1"
        ]
      }
    }
    ...
}
```