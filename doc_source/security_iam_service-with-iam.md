# How Amazon Aurora works with IAM<a name="security_iam_service-with-iam"></a>

Before you use IAM to manage access to Aurora, you should understand what IAM features are available to use with Aurora\. To get a high\-level view of how Aurora and other AWS services work with IAM, see [AWS services that work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

**Topics**
+ [Aurora identity\-based policies](#security_iam_service-with-iam-id-based-policies)
+ [Aurora resource\-based policies](#security_iam_service-with-iam-resource-based-policies)
+ [Authorization based on Aurora tags](#security_iam_service-with-iam-tags)
+ [Aurora IAM roles](#security_iam_service-with-iam-roles)

## Aurora identity\-based policies<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources as well as the conditions under which actions are allowed or denied\. Aurora supports specific actions, resources, and condition keys\. To learn about all of the elements that you use in a JSON policy, see [IAM JSON policy elements reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

The `Action` element of an IAM identity\-based policy describes the specific action or actions that will be allowed or denied by the policy\. Policy actions usually have the same name as the associated AWS API operation\. The action is used in a policy to grant permissions to perform the associated operation\. 

Policy actions in Aurora use the following prefix before the action: `rds:`\. For example, to grant someone permission to describe DB instances with the Amazon RDS `DescribeDBInstances` API operation, you include the `rds:DescribeDBInstances` action in their policy\. Policy statements must include either an `Action` or `NotAction` element\. Aurora defines its own set of actions that describe tasks that you can perform with this service\.

To specify multiple actions in a single statement, separate them with commas as follows:

```
"Action": [
      "rds:action1",
      "rds:action2"
```

You can specify multiple actions using wildcards \(\*\)\. For example, to specify all actions that begin with the word `Describe`, include the following action:

```
"Action": "rds:Describe*"
```



To see a list of Aurora actions, see [Actions Defined by Amazon RDS](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonrds.html#amazonrds-actions-as-permissions) in the *Service Authorization Reference*\.

### Resources<a name="security_iam_service-with-iam-id-based-policies-resources"></a>

The `Resource` element specifies the object or objects to which the action applies\. Statements must include either a `Resource` or a `NotResource` element\. You specify a resource using an ARN or using the wildcard \(\*\) to indicate that the statement applies to all resources\.



The DB instance resource has the following ARN:

```
arn:${Partition}:rds:${Region}:${Account}:{ResourceType}/${Resource}
```

For more information about the format of ARNs, see [Amazon Resource Names \(ARNs\) and AWS service namespaces](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\.

For example, to specify the `dbtest` DB instance in your statement, use the following ARN:

```
"Resource": "arn:aws:rds:us-west-2:123456789012:db:dbtest"
```

To specify all DB instances that belong to a specific account, use the wildcard \(\*\):

```
"Resource": "arn:aws:rds:us-east-1:123456789012:db:*"
```

Some RDS API operations, such as those for creating resources, cannot be performed on a specific resource\. In those cases, you must use the wildcard \(\*\)\.

```
"Resource": "*"
```

Many Amazon RDS API operations involve multiple resources\. For example, `CreateDBInstance` creates a DB instance\. You can specify that an IAM user must use a specific security group and parameter group when creating a DB instance\. To specify multiple resources in a single statement, separate the ARNs with commas\. 

```
"Resource": [
      "resource1",
      "resource2"
```

To see a list of Aurora resource types and their ARNs, see [Resources Defined by Amazon RDS](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonrds.html#amazonrds-resources-for-iam-policies) in the *Service Authorization Reference*\. To learn with which actions you can specify the ARN of each resource, see [Actions Defined by Amazon RDS](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonrds.html#amazonrds-actions-as-permissions)\.

### Condition keys<a name="UsingWithRDS.IAM.Conditions"></a>

The `Condition` element \(or `Condition` *block*\) lets you specify conditions in which a statement is in effect\. The `Condition` element is optional\. You can build conditional expressions that use [condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html), such as equals or less than, to match the condition in the policy with values in the request\. 

If you specify multiple `Condition` elements in a statement, or multiple keys in a single `Condition` element, AWS evaluates them using a logical `AND` operation\. If you specify multiple values for a single condition key, AWS evaluates the condition using a logical `OR` operation\. All of the conditions must be met before the statement's permissions are granted\.

 You can also use placeholder variables when you specify conditions\. For example, you can grant an IAM user permission to access a resource only if it is tagged with their IAM user name\. For more information, see [IAM policy elements: Variables and tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html) in the *IAM User Guide*\. 

Aurora defines its own set of condition keys and also supports using some global condition keys\. To see all AWS global condition keys, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.



 All RDS API operations support the `aws:RequestedRegion` condition key\. 

To see a list of Aurora condition keys, see [Condition Keys for Amazon RDS](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonrds.html#amazonrds-policy-keys) in the *Service Authorization Reference*\. To learn with which actions and resources you can use a condition key, see [Actions Defined by Amazon RDS](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonrds.html#amazonrds-actions-as-permissions)\.

### Examples<a name="security_iam_service-with-iam-id-based-policies-examples"></a>



To view examples of Aurora identity\-based policies, see [Amazon Aurora identity\-based policy examples](security_iam_id-based-policy-examples.md)\.

## Aurora resource\-based policies<a name="security_iam_service-with-iam-resource-based-policies"></a>

Aurora does not support resource\-based policies\.

## Authorization based on Aurora tags<a name="security_iam_service-with-iam-tags"></a>

You can attach tags to Aurora resources or pass tags in a request to Aurora\. To control access based on tags, you provide tag information in the [condition element](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) of a policy using the `aws:ResourceTag/key-name`, `aws:RequestTag/key-name`, or `aws:TagKeys` condition keys\. For more information, see [Controlling access to AWS resources using tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html) in the *IAM User Guide*\. For more information about tagging Aurora resources, see [Specifying conditions: Using custom tags](security_iam_id-based-policy-examples.md#UsingWithRDS.IAM.SpecifyingCustomTags)\.

To view an example identity\-based policy for limiting access to a resource based on the tags on that resource, see [Grant permission for actions on a resource with a specific tag with two different values](security_iam_id-based-policy-examples.md#security_iam_id-based-policy-examples-grant-permissions-tags)\.

## Aurora IAM roles<a name="security_iam_service-with-iam-roles"></a>

An [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is an entity within your AWS account that has specific permissions\.

### Using temporary credentials with Aurora<a name="security_iam_service-with-iam-roles-tempcreds"></a>

You can use temporary credentials to sign in with federation, assume an IAM role, or to assume a cross\-account role\. You obtain temporary security credentials by calling AWS STS API operations such as [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) or [GetFederationToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html)\. 

Aurora supports using temporary credentials\. 

### Service\-linked roles<a name="security_iam_service-with-iam-roles-service-linked"></a>

[Service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role) allow AWS services to access resources in other services to complete an action on your behalf\. Service\-linked roles appear in the Roles list in the IAM Management Console and are owned by the service\. An IAM administrator can view but not edit the permissions for service\-linked roles\.

Aurora supports service\-linked roles\. For details about creating or managing Aurora service\-linked roles, see [Using service\-linked roles for Amazon Aurora](UsingWithRDS.IAM.ServiceLinkedRoles.md)\.

### Service roles<a name="security_iam_service-with-iam-roles-service"></a>

This feature allows a service to assume a [service role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-role) on your behalf\. This role allows the service to access resources in other services to complete an action on your behalf\. Service roles appear in the Roles list in the IAM Management Console and are owned by your account\. This means that an IAM administrator can change the permissions for this role\. However, doing so might break the functionality of the service\.

Aurora supports service roles\.