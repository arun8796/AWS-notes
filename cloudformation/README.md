## Preface

AWS CloudFormation is a service that gives developers a way to create a collection of AWS resources in an orderly and predictable fashion via a well defined scripting language that can be based on JSON or YAML.

In CloudFormation developers create templates that are then uploaded into S3 and executed via the console or via the APIs. When a template is started a new Stack is being create and the outcome of this operation can be one one of the following two:
- The stack succeded, all the AWS resources described in the template were properly provisioned in AWS.
- The stack failed, something went wrong during the execution, all the resources provisioned as part of the stack execution are automatically removed (Rollback).

There is no limit on the total number of templates that developers can create however there is a soft limit on the total number of Stacks that can be executed, default value is 200 (this soft limit can be however changed raising a support request with AWS).

When a new stack need to be executed the following actions are required:

- A unique stack name must be provided.
- A template is selected.
- If the template declares input parameters then these parameters must be provided by the user.
- The stack is being created and when finished the template output information are returned to the user.

For every stack a users can also decide to:

- Delete the stack entirely, in this case all the resources provisioned during the creation are disposed.
- Update an existing stack, in this case CloudFrmation computes a **diff** between the new template and the resources already allocated in the Stack and only applies the difference.

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

- Format Version (optional). Specifies the AWS CloudFormation template version that the template conforms to. The template format version is not the same as the API or WSDL version. The template format version can change independently of the API and WSDL versions.
- Description (optional). A text string that describes the template. This section must always follow the template format version section.
- Metadata (optional). Objects that provide additional information about the template.
- Parameters (optional). Specifies values that you can pass in to your template at runtime (when you create or update a stack). You can refer to parameters in the Resources and Outputs sections of the template.
- Mappings (optional). A mapping of keys and associated values that you can use to specify conditional parameter values, similar to a lookup table. You can match a key to a corresponding value by using the Fn::FindInMap intrinsic function in the Resources and Outputs section.
- Conditions (optional). Defines conditions that control whether certain resources are created or whether certain resource properties are assigned a value during stack creation or update. For example, you could conditionally create a resource that depends on whether the stack is for a production or test environment.
- Transform (optional). For serverless applications (also referred to as Lambda-based applications), specifies the version of the AWS Serverless Application Model (AWS SAM) to use. When you specify a transform, you can use AWS SAM syntax to declare resources in your template. The model defines the syntax that you can use and how it is processed. You can also use the AWS::Include transform to work with template snippets that are stored separately from the main AWS CloudFormation template. You store your snippet files in an Amazon S3 bucket and then reuse the functions across multiple templates.
- Resources (required). Specifies the stack resources and their properties, such as an Amazon Elastic Compute Cloud instance or an Amazon Simple Storage Service bucket. You can refer to resources in the Resources and Outputs sections of the template.
- Outputs (optional). Describes the values that are returned whenever you view your stack's properties. For example, you can declare an output for an S3 bucket name and then call the aws cloudformation describe-stacks AWS CLI command to view the name.

##  Intrinsic Functions

Intrinsic functions in CloudFormation help developers to assign values to template properties that are not available until runtime.

```
**Example**
A classic example is the registration of a LoadBalancer DNS with a Route53 domain.

As at the time the developer creates the template the LodBalancer **DnsName** is not yet assigned an intrinsic function, Fn:GetAtt in this case, can be used as placeholder to reference the value of the **DnsName** output property that is obtained once the AWS resource is created.
```

Top Intrinsic functions to remember:

Function | Description | Syntax
--- | --- | ---
Fn::FindInMap | Returns the value corresponding to keys in a two-level map that is declared in the Mappings section. | ``` { "Fn::FindInMap" : [ "MapName", "TopLevelKey", "SecondLevelKey"] } ```
Fn::GetAtt | Returns the value of an attribute from a resource in the template. | ``` { "Fn::GetAtt" : [ "logicalNameOfResource", "attributeName" ] } ```
Fn::GetAZs | Returns an array that lists Availability Zones for a specified region. | ``` { "Fn::GetAZs" : "region" } ```
Fn::Join | Appends a set of values into a single value, separated by the specified delimiter. If a delimiter is the empty string, the set of values are concatenated with no delimiter. | ``` { "Fn::Join" : [ "delimiter", [ comma-delimited list of values ] ] } ```

## Pricing

From a pricing perspective the AWS CloudFormation service is free of charge, user only pays the resources that are provisione as part of a stack creation. 

## TOP APIs to remember