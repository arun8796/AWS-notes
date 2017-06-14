## Preface

AWS CloudFormation is a service that gives developers a way to create a collection of AWS resources in an orderly and predictable fashion via a well defined scripting language that can be based on JSON or YAML.

In CloudFrmation developers create templates that are then uploaded into S3 and executed via the console or via the APIs. When a template is started a new Stack is being create and the outcome of this operation can be one one of the following two:
- The stack succeded, all the AWS resources described in the template were properly provisioned in AWS.
- The stack failed, something went wrong during the execution, all the resources provisioned as part ofthe stack are automatically removed (Rollback).

There is no limit on the total number of templates that developers can create however there is a soft limit on the total number of Stacks that can be executed, default value is 200 (this soft limit can be however changed raising a support request with AWS).

When a new stack need to be executed the following actions are required:
- A unique stack name must be provided.
- A template is selected.
- Igf the template declares input parameters then these parmeters must be provided by the user.
- The stck is being created and when finished the template output information are returned to the user.



## Anathomy of a CloudFormation template

```
{
  "AWSTemplateFormatVersion" : "version date",

  "Description" : "JSON string",

  "Metadata" : {
    template metadata
  },

  "Parameters" : {
    set of parameters
  },

  "Mappings" : {
    set of mappings
  },

  "Conditions" : {
    set of conditions
  },

  "Transform" : {
    set of transforms
  },

  "Resources" : {
    set of resources
  },

  "Outputs" : {
    set of outputs
  }
}
```

## Pricing

From a pricing perspective the AWS CloudFormation service is free of charge, user only pays the resources that are provisione as part of a stack creation. 

## TOP APIs to remember