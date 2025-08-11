

AWS Identity & Access Management (IAM)
--------------------------------------



[AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) enables you to manage access to AWS services and resources securely. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

Overview
--------

Pulumi Crosswalk for AWS adds strongly typed IAM resource classes, to ensure that you can create, update, and otherwise manage AWS users, groups, and roles securely and robustly.

Creating IAM Policy Documents Safely and Easily
-----------------------------------------------

You manage access in AWS by creating policies and attaching them to IAM identities or AWS resources. A policy is an object in AWS that, when associated with an entity or resource, defines their permissions. AWS evaluates these policies when a principal, such as a user, makes a request. Permissions in the policies determine whether the request is allowed or denied.

### Policy Documents

IAM policies define permissions for an action regardless of the method that you use to perform the operation. For example, if a policy allows the GetUser action, then a user with that policy can get user information from the AWS Management Console, the AWS CLI, or the AWS API. When you create an IAM user, you can set up the user to allow console or programmatic access. The IAM user can sign in to the console using a user name and password. Or they can use access keys to work with the CLI or API.

Most policies are stored in AWS as JSON documents. Identity-based policies, policies used to set boundaries, or AWS STS boundary policies are JSON policy documents that you attach to a user or role. Resource-based policies are JSON policy documents that you attach to a resource. SCPs are JSON policy documents with restricted syntax that you attach to an AWS Organizations organizational unit (OU). ACLs are also attached to a resource, but you use a different syntax.

A JSON policy document includes these elements:

*   Optional policywide information at the top of the document
*   One or more individual statements

Each statement includes information about a single permission. If a policy includes multiple statements, AWS applies a logical OR across the statements when evaluating them. If multiple policies apply to a request, AWS applies a logical OR across all of those policies when evaluating them.

For more extensive details about IAM policies and their contents, refer to the [AWS access policies documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html).

#### Strongly Typed Policy Documents (TypeScript-only)

Pulumi Crosswalk for AWS in TypeScript defines [the `aws.iam.PolicyDocument` interface](https://www.pulumi.com/registry/packages/aws/api-docs/iam) to add strong type checking to your policy documents. By using this type, you will know at compile time whether you’ve mistyped an attribute:

```
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const policy: aws.iam.PolicyDocument = {
    Version: "2012-10-17",
    Statement: [
        {
            Action: "sts:AssumeRole",
            Principal: {
                Service: "ec2.amazonaws.com",
            },
            Effect: "Allow",
        },
    ],
};

```


This policy object can then be used to configure a variety of IAM objects, as you will see below. For example, you can use the above policy to configure an IAM role that permits an assume role action for a given principal:

```
const role = new aws.iam.Role("instance-role", {
    assumeRolePolicy: policy,
    path: "/",
});

const profile = new aws.iam.InstanceProfile("instance-profile", { role });

```


### Pre-Defined IAM Managed Policies

An AWS managed policy is a standalone policy that is created and administered by AWS. Standalone policy means that the policy has its own Amazon Resource Name (ARN) that includes the policy name. For example, `arn:aws:iam::aws:policy/IAMReadOnlyAccess` is an AWS managed policy.

In places that accept a policy ARN, such as the `RolePolicyAttachment` resource, you can pass the ARN as a string, but that requires that you either memorize or look up the ARN each time. Instead, you can use the strongly typed `ManagedPolicy` enum, which exports a collection of constants for all available managed policies. For example, instead of typing out the ARN by hand, you can just reference `ManagedPolicy`’s `IAMReadOnlyAccess` enum value:

```
"use strict";
const aws = require("@pulumi/aws");

const role = new aws.iam.Role("my-role", {
    assumeRolePolicy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: "sts:AssumeRole",
                Principal: {
                    Service: "ec2.amazonaws.com",
                },
                Effect: "Allow",
            },
        ],
    },
});

const rolePolicyAttachment = new aws.iam.RolePolicyAttachment("my-rpa", {
    role: role,
    policyArn: aws.iam.ManagedPolicy.IAMReadOnlyAccess,
});

```


```
import * as aws from "@pulumi/aws";

const role = new aws.iam.Role("my-role", {
    assumeRolePolicy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: "sts:AssumeRole",
                Principal: {
                    Service: "ec2.amazonaws.com",
                },
                Effect: "Allow",
            },
        ],
    },
});

const rolePolicyAttachment = new aws.iam.RolePolicyAttachment("my-rpa", {
    role: role,
    policyArn: aws.iam.ManagedPolicy.IAMReadOnlyAccess,
});

```


```
import json
import pulumi_aws as aws

role = aws.iam.Role('my-role',
    assume_role_policy=json.dumps({
        'Version': '2012-10-17',
        'Statement': [{
            'Action': 'sts:AssumeRole',
            'Principal': {
                'Service': 'ec2.amazonaws.com'
            },
            'Effect': 'Allow',
        }],
    }),
)

role_policy_attachment = aws.iam.RolePolicyAttachment('my-rpa',
    role=role.id,
    policy_arn=aws.iam.ManagedPolicy.IAM_READ_ONLY_ACCESS,
)

```


```
package main

import (
	"github.com/pulumi/pulumi-aws/sdk/v6/go/aws/iam"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		assumeRole, err := iam.GetPolicyDocument(ctx, &iam.GetPolicyDocumentArgs{
			Statements: []iam.GetPolicyDocumentStatement{
				{
					Effect: pulumi.StringRef("Allow"),
					Principals: []iam.GetPolicyDocumentStatementPrincipal{
						{
							Type: "Service",
							Identifiers: []string{
								"ec2.amazonaws.com",
							},
						},
					},
					Actions: []string{
						"sts:AssumeRole",
					},
				},
			},
		}, nil)
		if err != nil {
			return err
		}

		role, err := iam.NewRole(ctx, "my-role", &iam.RoleArgs{
			AssumeRolePolicy: pulumi.String(assumeRole.Json),
		})
		if err != nil {
			return err
		}

		_, err = iam.NewRolePolicyAttachment(ctx, "my-rpa", &iam.RolePolicyAttachmentArgs{
			Role:      role.Name,
			PolicyArn: iam.ManagedPolicyReadOnlyAccess,
		})
		if err != nil {
			return err
		}
		return nil
	})
}

```


```
using Pulumi;
using Pulumi.Aws.Iam;
using System.Collections.Generic;
using System.Text.Json;

return await Deployment.RunAsync(() =>
{
    var role = new Role("my-role", new RoleArgs
    {
        AssumeRolePolicy = JsonSerializer.Serialize(new Dictionary<string, object?>
        {
            ["Version"] = "2012-10-17",
            ["Statement"] = new[]
            {
                new Dictionary<string, object?>
                {
                    ["Action"] = "sts:AssumeRole",
                    ["Effect"] = "Allow",
                    ["Sid"] = "",
                    ["Principal"] = new Dictionary<string, object?>
                    {
                        ["Service"] = "ec2.amazonaws.com",
                    },
                },
            },
        }),
    });
    
    var rolePolicyAttachment = new RolePolicyAttachment("my-rpa", new()
    {
        Role = role.Id,
        PolicyArn = ManagedPolicy.ReadOnlyAccess.ToString(),
    });
});

```


```
package myproject;

import com.pulumi.Context;
import com.pulumi.Pulumi;
import com.pulumi.aws.iam.Role;
import com.pulumi.aws.iam.RoleArgs;
import com.pulumi.aws.iam.RolePolicyAttachment;
import com.pulumi.aws.iam.RolePolicyAttachmentArgs;
import static com.pulumi.codegen.internal.Serialization.*;
import com.pulumi.aws.iam.enums.ManagedPolicy;
import com.pulumi.core.Output;

public class App {
    public static void main(String[] args) {
        Pulumi.run(App::stack);
    }

    public static void stack(Context ctx) {
        var role = new Role("my-role", RoleArgs.builder()        
            .assumeRolePolicy(serializeJson(
                jsonObject(
                    jsonProperty("Version", "2012-10-17"),
                    jsonProperty("Statement", jsonArray(jsonObject(
                        jsonProperty("Action", "sts:AssumeRole"),
                        jsonProperty("Effect", "Allow"),
                        jsonProperty("Sid", ""),
                        jsonProperty("Principal", jsonObject(
                            jsonProperty("Service", "ec2.amazonaws.com")
                        ))
                    )))
                )))
            .build());

        new RolePolicyAttachment("my-rpa", RolePolicyAttachmentArgs.builder()        
            .role(role.name())
            .policyArn(Output.format(ManagedPolicy.ReadOnlyAccess.getValue()))
            .build());
    }
}
```


```
name: aws-iam-role-policyattachment-managedpolicy-yaml
runtime: yaml
description: An example that deploys an IAM role and a policy attachment with an AWS managed policy on AWS.

resources:
  role:
    type: aws:iam:Role
    name: my-role
    properties:
      assumeRolePolicy: ${assumeRole.json}

  myRpa:
    type: aws:iam:RolePolicyAttachment
    properties:
      role: ${role.name}
      policyArn: "arn:aws:iam::aws:policy/ReadOnlyAccess"

## The enum functionality is not supported in YAML. To use an AWS Managed policy, reference the policy ARN directly.

variables:
  assumeRole:
    fn::invoke:
      Function: aws:iam:getPolicyDocument
      Arguments:
        statements:
          - effect: Allow
            principals:
              - type: Service
                identifiers:
                  - ec2.amazonaws.com
            actions:
              - sts:AssumeRole

```


Creating IAM Users, Groups, and Roles
-------------------------------------

The primary IAM object types are users, groups, and roles. This section demonstrates how they relate and how to create and configure them.

### IAM Users

An AWS Identity and Access Management (IAM) user is an entity that you create in AWS to represent the person or application that uses it to interact with AWS. A user in AWS consists of a name and credentials.

Use the [`User` resource](https://www.pulumi.com/registry/packages/aws/api-docs/iam/user) to create new IAM users. This example creates an IAM user and attaches a policy:

```
"use strict";
const pulumi = require("@pulumi/pulumi");
const aws = require("@pulumi/aws");

const user = new aws.iam.User("webmaster", {
    path: "/system/",
    tags: { Name: "webmaster" },
});

const userAccessKey = new aws.iam.AccessKey("webmasterKey", { user: user.name });

const userPolicy = new aws.iam.UserPolicy("webmasterPolicy", {
    user: user.name,
    policy: JSON.stringify({
        Version: "2012-10-17",
        Statement: [
            {
                Action: ["ec2:Describe*"],
                Effect: "Allow",
                Resource: "*",
            },
        ],
    }),
});

```


```
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const user = new aws.iam.User("webmaster", {
    path: "/system/",
    tags: { Name: "webmaster" },
});

const userAccessKey = new aws.iam.AccessKey("webmasterKey", { user: user.name });

const userPolicy = new aws.iam.UserPolicy("webmasterPolicy", {
    user: user.name,
    policy: JSON.stringify({
        Version: "2012-10-17",
        Statement: [
            {
                Action: ["ec2:Describe*"],
                Effect: "Allow",
                Resource: "*",
            },
        ],
    }),
});

```


```
import json
import pulumi_aws as aws

user = aws.iam.User('webmaster',
    path='/system/',
    tags={ 'Name': 'webmaster' },
)
user_access_key = aws.iam.AccessKey('webmasterKey',
    user=user.name,
)
user_policy = aws.iam.UserPolicy('webmasterPolicy',
    user=user.id,
    policy=json.dumps({
        'Version': '2012-10-17',
        'Statement': [{
            'Action': [ 'ec2:Describe*' ],
            'Effect': 'Allow',
            'Resource': '*'
        }],
    }),
)
```


```
package main

import (
	"encoding/json"
	"github.com/pulumi/pulumi-aws/sdk/v6/go/aws/iam"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		user, err := iam.NewUser(ctx, "webmaster", &iam.UserArgs{
			Path: pulumi.String("/system/"),
			Tags: pulumi.StringMap{
				"Name": pulumi.String("webmaster"),
			},
		})
		if err != nil {
			return err
		}

		_, err = iam.NewAccessKey(ctx, "webmasterKey", &iam.AccessKeyArgs{
			User: user.Name,
		})
		if err != nil {
			return err
		}

		policy, err := json.Marshal(map[string]interface{}{
			"Version": "2012-10-17",
			"Statement": []map[string]interface{}{
				map[string]interface{}{
					"Action": []string{
						"ec2:Describe*",
					},
					"Effect":   "Allow",
					"Resource": "*",
				},
			},
		})
		if err != nil {
			return err
		}

		_, err = iam.NewUserPolicy(ctx, "webmasterPolicy", &iam.UserPolicyArgs{
			User:   user.Name,
			Policy: pulumi.String(string(policy)),
		})
		if err != nil {
			return err
		}

		return nil
	})
}

```


```
using Pulumi;
using Pulumi.Aws.Iam;
using System.Collections.Generic;
using System.Text.Json;

return await Deployment.RunAsync(() =>
{
    var user = new User("webmaster", new UserArgs
    {
        Path = "/system/",
        Tags = new InputMap<string>
        {
            { "Name", "webmaster" }
        },
    });
    
    var userAccessKey = new AccessKey("webmasterKey", new AccessKeyArgs
    {
        User = user.Name,
    });
    
    var userPolicy = new UserPolicy("webmasterPolicy", new UserPolicyArgs
    {
        User = user.Id,
        Policy = JsonSerializer.Serialize(new Dictionary<string, object>
        {
            ["Version"] = "2012-10-17",
            ["Statement"] = new[]
            {
                new Dictionary<string, object?>
                {
                    ["Action"] = new[]
                    {
                        "ec2:Describe*",
                    },
                    ["Effect"] = "Allow",
                    ["Resource"] = "*",
                },
            },
        }),
    });
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.aws.iam.User;
import com.pulumi.aws.iam.UserArgs;
import com.pulumi.aws.iam.UserPolicy;
import com.pulumi.aws.iam.UserPolicyArgs;
import com.pulumi.aws.iam.AccessKey;
import com.pulumi.aws.iam.AccessKeyArgs;
import static com.pulumi.codegen.internal.Serialization.*;
import java.util.Map;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var user = new User("webmaster", UserArgs.builder()
                .path("/system/")
                .tags(Map.of("Name", "webmaster"))
                .build());

            new AccessKey("webmasterKey", AccessKeyArgs.builder()
                .user(user.name())
                .build());

            new UserPolicy("webmasterPolicy", UserPolicyArgs.builder()
                .user(user.id())
                .policy(serializeJson(
                    jsonObject(
                        jsonProperty("Version", "2012-10-17"),
                        jsonProperty("Statement", jsonArray(jsonObject(
                            jsonProperty("Action", jsonArray("ec2:Describe*")),
                            jsonProperty("Effect", "Allow"),
                            jsonProperty("Resource", "*")
                        )))
                    )))
                .build());
        });
    }
}

```


```
name: aws-iam-user-userpolicy-yaml
runtime: yaml
description: An example that deploys an IAM user and user policy on AWS.

resources:
  webmasterUser:
    type: aws:iam/user:User
    properties:
      path: /system/
      name: webmaster

  webmasterAccessKey:
    type: aws:iam/accessKey:AccessKey
    name: webmaster
    properties:
      user: ${webmasterUser.name}

  webmasterPolicy:
    type: aws:iam/userPolicy:UserPolicy
    properties:
      user: ${webmasterUser.name}
      policy:
        fn::toJSON:
          Version: "2012-10-17"
          Statement:
            - Effect: Allow
              Action:
                - "ec2:Describe*"
              Resource: "*"

```



### IAM Groups

An IAM group is a collection of IAM users, making it easier for you specify permissions for multiple users at once. For example, you could have a group called Admins and give that group the types of permissions that administrators need.

Any user in that group automatically has the permissions assigned to the group. If a new user joins your organization and needs administrator privileges, you can assign the appropriate permissions by adding the user to that group. Similarly, if a person changes jobs in your organization, instead of editing that user’s permissions, you can remove them from the old groups and add them to the appropriate new groups.

Use the [`Group` resource](https://www.pulumi.com/registry/packages/aws/api-docs/iam/group) to manage IAM groups. For example, this code creates a new group for an organization’s developers, specifies a policy for that group, and adds a couple users into it, thereby granting them permissions from the developer group all at once:

```
"use strict";
const pulumi = require("@pulumi/pulumi");
const aws = require("@pulumi/aws");

// Create our users.
const jane = new aws.iam.User("jane");
const mary = new aws.iam.User("mary");

// Define a group and assign a policy for it.
const devs = new aws.iam.Group("devs", {
    path: "/users/",
});

const myDeveloperPolicy = new aws.iam.GroupPolicy("my_developer_policy", {
    group: devs.id,
    policy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: ["ec2:Describe*"],
                Effect: "Allow",
                Resource: "*",
            },
        ],
    },
});

// Finally add the users as members to this group.
const devTeam = new aws.iam.GroupMembership("dev-team", {
    group: devs.id,
    users: [jane.id, mary.id],
});

```


```
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// Create our users.
const jane = new aws.iam.User("jane");
const mary = new aws.iam.User("mary");

// Define a group and assign a policy for it.
const devs = new aws.iam.Group("devs", {
    path: "/users/",
});

const myDeveloperPolicy = new aws.iam.GroupPolicy("my_developer_policy", {
    group: devs.id,
    policy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: ["ec2:Describe*"],
                Effect: "Allow",
                Resource: "*",
            },
        ],
    },
});

// Finally add the users as members to this group.
const devTeam = new aws.iam.GroupMembership("dev-team", {
    group: devs.id,
    users: [jane.id, mary.id],
});

```


```
import json
import pulumi_aws as aws

# Create our users.
jane = aws.iam.User('jane')
mary = aws.iam.User('mary')

# Define a group and assign a policy for it.
devs = aws.iam.Group('devs',
    path='/users/',
)
my_developer_policy = aws.iam.GroupPolicy('my_developer_policy',
    group=devs.id,
    policy=json.dumps({
        'Version': '2012-10-17',
        'Statement': [{
            'Action': [ 'ec2:Describe*' ],
            'Effect': 'Allow',
            'Resource': '*',
        }],
    }),
)

# Finally add the users as members to this group.
dev_team = aws.iam.GroupMembership('dev-team',
    group=devs.id,
    users=[ jane.id, mary.id ],
)

```


```
package main

import (
	"github.com/pulumi/pulumi-aws/sdk/v6/go/aws/iam"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		// Create our users.
		jane, err := iam.NewUser(ctx, "jane", &iam.UserArgs{
			Name: pulumi.String("jane"),
		})
		if err != nil {
			return err
		}
		mary, err := iam.NewUser(ctx, "mary", &iam.UserArgs{
			Name: pulumi.String("mary"),
		})
		if err != nil {
			return err
		}

		// Define a group and assign a policy for it.
		devs, err := iam.NewGroup(ctx, "devs", &iam.GroupArgs{
			Path: pulumi.String("/users/"),
		})
		if err != nil {
			return err
		}
		_, err = iam.NewGroupPolicy(
			ctx, "my_developer_policy", &iam.GroupPolicyArgs{
				Group: devs.ID().ToStringOutput(),
				Policy: pulumi.String(`{
					"Version": "2012-10-17",
					"Statement": [{
						"Action": [ "ec2:Describe*" ],
						"Effect": "Allow",
						"Resource": "*"
					}]
				}
				`),
			},
		)
		if err != nil {
			return err
		}

		// Finally add the users as members to this group.
		_, err = iam.NewGroupMembership(
			ctx, "dev-team", &iam.GroupMembershipArgs{
				Group: devs.ID().ToStringOutput(),
				Users: pulumi.StringArray{
					jane.ID().ToStringOutput(),
					mary.ID().ToStringOutput(),
				},
			},
		)
		if err != nil {
			return err
		}

		return nil
	})
}

```


```
using Pulumi;
using Pulumi.Aws.Iam;
using System.Collections.Generic;
using System.Text.Json;

return await Deployment.RunAsync(() =>
{
    // Create our users.
    var jane = new User("jane", new UserArgs());
    var mary = new User("mary", new UserArgs());

    // Define a group and assign a policy for it.
    var devs = new Group("devs", new GroupArgs
    {
        Path = "/users/",
    });

    var myDeveloperPolicy = new GroupPolicy("my_developer_policy", new GroupPolicyArgs
    {
        Group = devs.Id,
        Policy = JsonSerializer.Serialize(new
        {
            Version = "2012-10-17",
            Statement = new[]
            {
                new
                {
                    Action = new[] { "ec2:Describe*" },
                    Effect = "Allow",
                    Resource = "*"
                }
            }
        }),
    });

    // Finally, add the users as members to this group.
    var devTeam = new GroupMembership("dev-team", new GroupMembershipArgs
    {
        Group = devs.Id,
        Users = { jane.Id, mary.Id }
    });
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.aws.iam.User;
import com.pulumi.aws.iam.UserArgs;
import com.pulumi.aws.iam.Group;
import com.pulumi.aws.iam.GroupArgs;
import com.pulumi.aws.iam.GroupPolicy;
import com.pulumi.aws.iam.GroupPolicyArgs;
import com.pulumi.aws.iam.GroupMembership;
import com.pulumi.aws.iam.GroupMembershipArgs;
import static com.pulumi.codegen.internal.Serialization.*;
import com.pulumi.core.Output;
import java.util.List;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            
            var jane = new User("jane", UserArgs.builder().build());
            var mary = new User("mary", UserArgs.builder().build());

            // Create a single output from the two other outputs.
            var userIds = Output.all(jane.id(), mary.id()).applyValue(ids -> List.of(ids.get(0), ids.get(1)));
            
            // Define a group and assign a policy for it.
            var devs = new Group("devs", GroupArgs.builder()
                .path("/users/")
                .build());
            
            new GroupPolicy("my_developer_policy", GroupPolicyArgs.builder()
                .group(devs.name())
                .policy(serializeJson(
                    jsonObject(
                        jsonProperty("Version", "2012-10-17"),
                        jsonProperty("Statement", jsonArray(jsonObject(
                            jsonProperty("Action", jsonArray("ec2:Describe*")),
                            jsonProperty("Effect", "Allow"),
                            jsonProperty("Resource", "*")
                        )))
                    )))
                .build());
    
            new GroupMembership("dev-team", GroupMembershipArgs.builder()
                .group(devs.id())
                .users(userIds)
                .build());
        });
    }
}
```


```
name: aws-iam-user-group-grouppolicy-yaml
runtime: yaml
description: An example that deploys an IAM user, group, and group policy on AWS.

resources:
  jane:
    type: aws:iam:User
    properties:
      name: jane

  mary:
    type: aws:iam:User
    properties:
      name: mary

  devs:
    type: aws:iam:Group
    properties:
      name: devs
      path: /users/

  myDeveloperPolicy:
    type: aws:iam:GroupPolicy
    name: my_developer_policy
    properties:
      name: my_developer_policy
      group: ${devs.name}
      policy:
        fn::toJSON:
          Version: 2012-10-17
          Statement:
            - Action:
                - ec2:Describe*
              Effect: Allow
              Resource: "*"

  devTeam:
    type: aws:iam:GroupMembership
    properties:
      name: dev-team
      users:
        - ${jane.id}
        - ${mary.id}
      group: ${devs.name}

```



### IAM Roles

An IAM role is an IAM identity that you can create in your account that has specific permissions. This is similar to an IAM user, in that it is an AWS identity with permission policies that determine what the identity can and cannot do in AWS. Instead of being uniquely associated with one person, however, a role is assumable by anyone who needs it. Also, a role does not have standard long-term credentials such as a password or access keys associated with it. Instead, when you assume a role, it provides you with temporary security credentials for your role session.

To manage IAM roles, use the [`Role` resource](https://www.pulumi.com/registry/packages/aws/api-docs/iam/role). The following example creates a new role with a custom policy document, and also attaches a managed policy afterwards:

```
"use strict";
const aws = require("@pulumi/aws");

const role = new aws.iam.Role("my-role", {
    assumeRolePolicy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: "sts:AssumeRole",
                Principal: {
                    Service: "ec2.amazonaws.com",
                },
                Effect: "Allow",
            },
        ],
    },
});

const rolePolicyAttachment = new aws.iam.RolePolicyAttachment("my-rpa", {
    role: role,
    policyArn: aws.iam.ManagedPolicy.IAMReadOnlyAccess,
});

```


```
import * as aws from "@pulumi/aws";

const role = new aws.iam.Role("my-role", {
    assumeRolePolicy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: "sts:AssumeRole",
                Principal: {
                    Service: "ec2.amazonaws.com",
                },
                Effect: "Allow",
            },
        ],
    },
});

const rolePolicyAttachment = new aws.iam.RolePolicyAttachment("my-rpa", {
    role: role,
    policyArn: aws.iam.ManagedPolicy.IAMReadOnlyAccess,
});

```


```
import json
import pulumi_aws as aws

role = aws.iam.Role('my-role',
    assume_role_policy=json.dumps({
        'Version': '2012-10-17',
        'Statement': [{
            'Action': 'sts:AssumeRole',
            'Principal': {
                'Service': 'ec2.amazonaws.com'
            },
            'Effect': 'Allow',
        }],
    }),
)

role_policy_attachment = aws.iam.RolePolicyAttachment('my-rpa',
    role=role.id,
    policy_arn=aws.iam.ManagedPolicy.IAM_READ_ONLY_ACCESS,
)

```


```
package main

import (
	"github.com/pulumi/pulumi-aws/sdk/v6/go/aws/iam"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		assumeRole, err := iam.GetPolicyDocument(ctx, &iam.GetPolicyDocumentArgs{
			Statements: []iam.GetPolicyDocumentStatement{
				{
					Effect: pulumi.StringRef("Allow"),
					Principals: []iam.GetPolicyDocumentStatementPrincipal{
						{
							Type: "Service",
							Identifiers: []string{
								"ec2.amazonaws.com",
							},
						},
					},
					Actions: []string{
						"sts:AssumeRole",
					},
				},
			},
		}, nil)
		if err != nil {
			return err
		}

		role, err := iam.NewRole(ctx, "my-role", &iam.RoleArgs{
			AssumeRolePolicy: pulumi.String(assumeRole.Json),
		})
		if err != nil {
			return err
		}

		_, err = iam.NewRolePolicyAttachment(ctx, "my-rpa", &iam.RolePolicyAttachmentArgs{
			Role:      role.Name,
			PolicyArn: iam.ManagedPolicyReadOnlyAccess,
		})
		if err != nil {
			return err
		}
		return nil
	})
}

```


```
using Pulumi;
using Pulumi.Aws.Iam;
using System.Collections.Generic;
using System.Text.Json;

return await Deployment.RunAsync(() =>
{
    var role = new Role("my-role", new RoleArgs
    {
        AssumeRolePolicy = JsonSerializer.Serialize(new Dictionary<string, object?>
        {
            ["Version"] = "2012-10-17",
            ["Statement"] = new[]
            {
                new Dictionary<string, object?>
                {
                    ["Action"] = "sts:AssumeRole",
                    ["Effect"] = "Allow",
                    ["Sid"] = "",
                    ["Principal"] = new Dictionary<string, object?>
                    {
                        ["Service"] = "ec2.amazonaws.com",
                    },
                },
            },
        }),
    });
    
    var rolePolicyAttachment = new RolePolicyAttachment("my-rpa", new()
    {
        Role = role.Id,
        PolicyArn = ManagedPolicy.ReadOnlyAccess.ToString(),
    });
});

```


```
package myproject;

import com.pulumi.Context;
import com.pulumi.Pulumi;
import com.pulumi.aws.iam.Role;
import com.pulumi.aws.iam.RoleArgs;
import com.pulumi.aws.iam.RolePolicyAttachment;
import com.pulumi.aws.iam.RolePolicyAttachmentArgs;
import static com.pulumi.codegen.internal.Serialization.*;
import com.pulumi.aws.iam.enums.ManagedPolicy;
import com.pulumi.core.Output;

public class App {
    public static void main(String[] args) {
        Pulumi.run(App::stack);
    }

    public static void stack(Context ctx) {
        var role = new Role("my-role", RoleArgs.builder()        
            .assumeRolePolicy(serializeJson(
                jsonObject(
                    jsonProperty("Version", "2012-10-17"),
                    jsonProperty("Statement", jsonArray(jsonObject(
                        jsonProperty("Action", "sts:AssumeRole"),
                        jsonProperty("Effect", "Allow"),
                        jsonProperty("Sid", ""),
                        jsonProperty("Principal", jsonObject(
                            jsonProperty("Service", "ec2.amazonaws.com")
                        ))
                    )))
                )))
            .build());

        new RolePolicyAttachment("my-rpa", RolePolicyAttachmentArgs.builder()        
            .role(role.name())
            .policyArn(Output.format(ManagedPolicy.ReadOnlyAccess.getValue()))
            .build());
    }
}
```


```
name: aws-iam-role-policyattachment-managedpolicy-yaml
runtime: yaml
description: An example that deploys an IAM role and a policy attachment with an AWS managed policy on AWS.

resources:
  role:
    type: aws:iam:Role
    name: my-role
    properties:
      assumeRolePolicy: ${assumeRole.json}

  myRpa:
    type: aws:iam:RolePolicyAttachment
    properties:
      role: ${role.name}
      policyArn: "arn:aws:iam::aws:policy/ReadOnlyAccess"

## The enum functionality is not supported in YAML. To use an AWS Managed policy, reference the policy ARN directly.

variables:
  assumeRole:
    fn::invoke:
      Function: aws:iam:getPolicyDocument
      Arguments:
        statements:
          - effect: Allow
            principals:
              - type: Service
                identifiers:
                  - ec2.amazonaws.com
            actions:
              - sts:AssumeRole

```


Roles are often useful for creating [instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html), which controls the IAM role assumed by compute running inside of your AWS account, whether that be in EC2, ECS, EKS, or Lambda, for example. To create one, use the [`InstanceProfile` resource](https://www.pulumi.com/registry/packages/aws/api-docs/iam/instanceprofile) and pass in your role:

```
"use strict";
const aws = require("@pulumi/aws");

const role = new aws.iam.Role("my-role", {
    assumeRolePolicy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: "sts:AssumeRole",
                Principal: {
                    Service: "ec2.amazonaws.com",
                },
                Effect: "Allow",
            },
        ],
    },
});

const profile = new aws.iam.InstanceProfile("instance-profile", { role });

```


```
import * as aws from "@pulumi/aws";

const role = new aws.iam.Role("my-role", {
    assumeRolePolicy: {
        Version: "2012-10-17",
        Statement: [
            {
                Action: "sts:AssumeRole",
                Principal: {
                    Service: "ec2.amazonaws.com",
                },
                Effect: "Allow",
            },
        ],
    },
});

const profile = new aws.iam.InstanceProfile("instance-profile", { role });

```


```
import json
import pulumi_aws as aws

role = aws.iam.Role('my-role',
    assume_role_policy=json.dumps({
        'Version': '2012-10-17',
        'Statement': [{
            'Action': 'sts:AssumeRole',
            'Principal': {
                'Service': 'ec2.amazonaws.com'
            },
            'Effect': 'Allow',
        }],
    }),
)

profile = aws.iam.InstanceProfile('instance-profile', role=role.id)
```


```
package main

import (
	"github.com/pulumi/pulumi-aws/sdk/v6/go/aws/iam"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		assumeRole, err := iam.GetPolicyDocument(ctx, &iam.GetPolicyDocumentArgs{
			Statements: []iam.GetPolicyDocumentStatement{
				{
					Effect: pulumi.StringRef("Allow"),
					Principals: []iam.GetPolicyDocumentStatementPrincipal{
						{
							Type: "Service",
							Identifiers: []string{
								"ec2.amazonaws.com",
							},
						},
					},
					Actions: []string{
						"sts:AssumeRole",
					},
				},
			},
		}, nil)
		if err != nil {
			return err
		}

		role, err := iam.NewRole(ctx, "my-role", &iam.RoleArgs{
			AssumeRolePolicy: pulumi.String(assumeRole.Json),
		})
		if err != nil {
			return err
		}

		_, err = iam.NewInstanceProfile(ctx, "instance-profile", &iam.InstanceProfileArgs{
			Name: pulumi.String("instance-profile"),
			Role: role.Name,
		})
		if err != nil {
			return err
		}
		return nil
	})
}

```


```
using Pulumi;
using Pulumi.Aws.Iam;
using System.Collections.Generic;
using System.Text.Json;

return await Deployment.RunAsync(() =>
{
    var role = new Role("my-role", new RoleArgs
    {
        AssumeRolePolicy = JsonSerializer.Serialize(new Dictionary<string, object?>
        {
            ["Version"] = "2012-10-17",
            ["Statement"] = new[]
            {
                new Dictionary<string, object?>
                {
                    ["Action"] = "sts:AssumeRole",
                    ["Effect"] = "Allow",
                    ["Sid"] = "",
                    ["Principal"] = new Dictionary<string, object?>
                    {
                        ["Service"] = "ec2.amazonaws.com",
                    },
                },
            },
        }),
    });
    
    var profile = new InstanceProfile("instance-profile", new()
    { 
        Role = role.Name 
        
    });
});

```


```
package myproject;

import com.pulumi.Context;
import com.pulumi.Pulumi;
import com.pulumi.aws.iam.Role;
import com.pulumi.aws.iam.RoleArgs;
import com.pulumi.aws.iam.InstanceProfile;
import com.pulumi.aws.iam.InstanceProfileArgs;
import static com.pulumi.codegen.internal.Serialization.*;

public class App {
    public static void main(String[] args) {
        Pulumi.run(App::stack);
    }

    public static void stack(Context ctx) {
        var role = new Role("my-role", RoleArgs.builder()        
            .assumeRolePolicy(serializeJson(
                jsonObject(
                    jsonProperty("Version", "2012-10-17"),
                    jsonProperty("Statement", jsonArray(jsonObject(
                        jsonProperty("Action", "sts:AssumeRole"),
                        jsonProperty("Effect", "Allow"),
                        jsonProperty("Sid", ""),
                        jsonProperty("Principal", jsonObject(
                            jsonProperty("Service", "ec2.amazonaws.com")
                        ))
                    )))
                )))
            .build());

        new InstanceProfile("instanceProfile", InstanceProfileArgs.builder()        
            .name("instance-profile")
            .role(role.name())
            .build());
    }
}
```


```
name: aws-iam-role-instanceprofile-yaml
runtime: yaml
description: An example that deploys an IAM role and instance profile on AWS.

resources:
  role:
    type: aws:iam:Role
    name: my-role
    properties:
      assumeRolePolicy: ${assumeRole.json}

  instanceProfile:
    type: aws:iam:InstanceProfile
    name: instance-profile
    properties:
      name: instance-profile
      role: ${role.name}

variables:
  assumeRole:
    fn::invoke:
      Function: aws:iam:getPolicyDocument
      Arguments:
        statements:
          - effect: Allow
            principals:
              - type: Service
                identifiers:
                  - ec2.amazonaws.com
            actions:
              - sts:AssumeRole

```

