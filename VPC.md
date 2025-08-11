# Using AWS VPC | Crosswalk | Pulumi Docs


AWS Virtual Private Cloud (VPC)
-------------------------------


![](/docs/iac/clouds/aws/guides/)

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications, and use multiple layers of security, including security groups and network access control lists, to limit network access to and from resources.

Overview
--------

Pulumi Crosswalk for AWS provides simple, out of the box VPC functionality that follows widely accepted best practices. This ensures you can provision and evolve your VPCs across many environments productively and safely, without needing to recreate the same VPC templates for every new project you tackle.

Using these capabilities, you can control the entire virtual network and restrict access to just those network endpoints that require it. These network resources are essential to configuring many of the other Crosswalk AWS components, including ECS and EKS clusters, API gateways, and various network load balancing options.

Each account has a default regional network and VPC to make it easy to get up and running. Most production circumstances call for dedicated VPCs and network isolation. This includes multi-tenanted scenarios where VPCs can be used for strong network isolation between endpoints and resources that are otherwise sharing an AWS account.

Managing VPCs
-------------

The VPC resource class provides full access to the AWS VPC API, and aws.ec2 the entire AWS EC2 API. Using these packages, you can configure all aspects of AWS networks for your applications and infrastructure.

The awsx.ec2.Vpc class encapsulates a complete configuration of an AWS network, including the actual VPC itself, in addition to public and/or private subnets, route tables, and gateways, across multiple availability zones. It is designed to be easier to use, with reasonable defaults, and follows AWS’s own best practices, with configurability for advanced scenarios. The two can be used together.

Below are some of the most common infrastructure as code tasks with VPCs.

Getting the Default VPC
-----------------------

Often resources like clusters, API gateways, lambdas, and more, will request a VPC object or ID. This ensures such resources inside of your VPC so network traffic are isolated from other VPCs in your account.

Each AWS account has a default VPC per region. Using the default VPC is often the easiest path when you’re just getting up and running or don’t yet understand your specific networking requirements. Most resources will use this default VPC automatically if you leave it unspecified. In other cases, you may be required to pass it explicitly, in which case you’ll need to get it programmatically.

The following example will read the default VPC and export some of its properties for easy consumption.

```
"use strict";
const awsx = require("@pulumi/awsx");

// Fetch the default VPC for the current AWS region.
const vpc = new awsx.ec2.DefaultVpc("default-vpc");

// Export a few properties to make them easy to use.
exports.vpcId = vpc.vpcId;
exports.vpcPrivateSubnetIds = vpc.privateSubnetIds;
exports.vpcPublicSubnetIds = vpc.publicSubnetIds;

```


```
import * as awsx from "@pulumi/awsx";

// Fetch the default VPC for the current AWS region.
const vpc = new awsx.ec2.DefaultVpc("default-vpc");

// Export a few properties to make them easy to use.
export const vpcId = vpc.vpcId;
export const vpcPrivateSubnetIds = vpc.privateSubnetIds;
export const vpcPublicSubnetIds = vpc.publicSubnetIds;

```


```
import pulumi
import pulumi_awsx as awsx

# Fetch the default VPC for the current AWS region.
vpc = awsx.ec2.DefaultVpc("default-vpc")

# Export a few properties to make them easy to use.
pulumi.export("vpcId", vpc.vpc_id)
pulumi.export("publicSubnetIds", vpc.public_subnet_ids)
pulumi.export("privateSubnetIds", vpc.private_subnet_ids)

```


```
package main

import (
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		// Fetch the default VPC for the current AWS region.
		vpc, err := ec2.NewDefaultVpc(ctx, "default-vpc", nil)
		if err != nil {
			return err
		}

		// Export a few properties to make them easy to use.
		ctx.Export("vpcId", vpc.VpcId)
		ctx.Export("privateSubnetIds", vpc.PrivateSubnetIds)
		ctx.Export("publicSubnetIds", vpc.PublicSubnetIds)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    // Fetch the default VPC for the current AWS region.
    var vpc = new DefaultVpc("default-vpc");

    // Export a few properties to make them easy to use.
    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
        ["vpcPrivateSubnetIds"] = vpc.PrivateSubnetIds,
        ["vpcPublicSubnetIds"] = vpc.PublicSubnetIds,
    };
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.DefaultVpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            // Fetch the default VPC for the current AWS region.
            var vpc = new DefaultVpc("default-vpc");


            // Export a few properties to make them easy to use.
            ctx.export("vpcId", vpc.vpcId());
            ctx.export("privateSubnetIds", vpc.privateSubnetIds());
            ctx.export("publicSubnetIds", vpc.publicSubnetIds());
        });
    }
}

```


```
name: awsx-vpc-default-yaml
runtime: yaml
description: An example of fetching the default AWS VPC for the current region.

resources:
  # Fetch the default VPC for the current AWS region.
  default-vpc:
    type: awsx:ec2:DefaultVpc

outputs:
  # Export a few properties to make them easy to use.
  vpcId: ${default-vpc.vpcId}
  publicSubnetIds: ${default-vpc.publicSubnetIds}
  privateSubnetIds: ${default-vpc.privateSubnetIds}

```


Once you have defined this function, running `pulumi up` will show:

```
$ pulumi up

Updating (dev)

     Type                    Name          Status
 +   pulumi:pulumi:Stack     awsx-vpc-dev  created (2s)
 +   └─ awsx:ec2:DefaultVpc  default-vpc   created (0.49s)

Outputs:
    privateSubnetIds: [
        [0]: "subnet-0b4f9fb1df1543b07"
    ]
    publicSubnetIds : [
        [0]: "subnet-43f43a1e"
        [1]: "subnet-c7d926bf"
        [2]: "subnet-d7e7fe9c"
    ]
    vpcId           : "vpc-4b82e033"

Resources:
    + 2 created

Duration: 3s

```


In this case, the VPC is not created and managed by Pulumi. Instead `DefaultVpc` reads from your AWS account and returns the VPC metadata. This object can be introspected or passed anywhere a `VpcID` or `SubnetIds` are expected.

Setting Up a New VPC
--------------------

Although using the default VPC is easy, it’s often not suitable for production. By setting up a dedicated VPC, we can isolate workloads from existing ones, and have more control over subnet configuration, routing, and controlling ingress and egress security rules.

To set up a new VPC, allocate a new `awsx.ec2.Vpc` object. This class offers a number of options, ranging from simple defaults that many will want to start with, to complete control over everything VPC has to offer.

The following code creates a new VPC using all default settings:

```
"use strict";
const pulumi = require("@pulumi/pulumi");
const awsx = require("@pulumi/awsx");

// Allocate a new VPC with the default settings.
const vpc = new awsx.ec2.Vpc("vpc");

// Export a few properties to make them easy to use.
exports.vpcId = vpc.vpcId;
exports.privateSubnetIds = vpc.privateSubnetIds;
exports.publicSubnetIds = vpc.publicSubnetIds;

```


```
import * as pulumi from "@pulumi/pulumi";
import * as awsx from "@pulumi/awsx";

// Allocate a new VPC with the default settings.
const vpc = new awsx.ec2.Vpc("vpc");

// Export a few properties to make them easy to use.
export const vpcId = vpc.vpcId;
export const privateSubnetIds = vpc.privateSubnetIds;
export const publicSubnetIds = vpc.publicSubnetIds;

```


```
import pulumi
import pulumi_awsx as awsx

# Allocate a new VPC with the default settings.
vpc = awsx.ec2.Vpc("vpc")

# Export a few properties to make them easy to use.
pulumi.export("vpcId", vpc.vpc_id)
pulumi.export("publicSubnetIds", vpc.public_subnet_ids)
pulumi.export("privateSubnetIds", vpc.private_subnet_ids)

```


```
package main

import (
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		// Allocate a new VPC with the default settings.
		vpc, err := ec2.NewVpc(ctx, "vpc", nil)
		if err != nil {
			return err
		}

		// Export a few properties to make them easy to use.
		ctx.Export("vpcId", vpc.VpcId)
		ctx.Export("privateSubnetIds", vpc.PrivateSubnetIds)
		ctx.Export("publicSubnetIds", vpc.PublicSubnetIds)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    // Allocate a new VPC with the default settings.
    var vpc = new Vpc("vpc");

    // Export a few properties to make them easy to use.
    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
        ["vpcPrivateSubnetIds"] = vpc.PrivateSubnetIds,
        ["vpcPublicSubnetIds"] = vpc.PublicSubnetIds,
    };
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.Vpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            // Allocate a new VPC with the default settings.
            var vpc = new Vpc("vpc");

            // Export a few properties to make them easy to use.
            ctx.export("vpcId", vpc.vpcId());
            ctx.export("privateSubnetIds", vpc.privateSubnetIds());
            ctx.export("publicSubnetIds", vpc.publicSubnetIds());
        });
    }
}

```


```
name: awsx-vpc-yaml
runtime: yaml
description: An example that creates a new VPC using the default settings.
resources:
  # Allocate a new VPC with the default settings.
  vpc:
    type: awsx:ec2:Vpc

outputs:
  # Export a few properties to make them easy to use.
  vpcId: ${vpc.vpcId}
  publicSubnetIds: ${vpc.publicSubnetIds}
  privateSubnetIds: ${vpc.privateSubnetIds}

```


If we run `pulumi up`, the VPC and its supporting resources will be provisioned:

```
$ pulumi up

Updating (dev)

     Type                                          Name           Status
 +   pulumi:pulumi:Stack                           awsx-vpc-dev   created (146s)
 +   └─ awsx:ec2:Vpc                               vpc            created (0.79s)
 +      └─ aws:ec2:Vpc                             vpc            created (1s)
 +         ├─ aws:ec2:Subnet                       vpc-private-3  created (0.95s)
 +         │  └─ aws:ec2:RouteTable                vpc-private-3  created (0.72s)
 +         │     ├─ aws:ec2:RouteTableAssociation  vpc-private-3  created (0.75s)
 +         │     └─ aws:ec2:Route                  vpc-private-3  created (1s)
 +         ├─ aws:ec2:Subnet                       vpc-public-1   created (11s)
 +         │  ├─ aws:ec2:Eip                       vpc-1          created (0.93s)
 +         │  ├─ aws:ec2:RouteTable                vpc-public-1   created (1s)
 +         │  │  ├─ aws:ec2:RouteTableAssociation  vpc-public-1   created (1s)
 +         │  │  └─ aws:ec2:Route                  vpc-public-1   created (1s)
 +         │  └─ aws:ec2:NatGateway                vpc-1          created (95s)
 +         ├─ aws:ec2:Subnet                       vpc-public-3   created (11s)
 +         │  ├─ aws:ec2:RouteTable                vpc-public-3   created (1s)
 +         │  │  ├─ aws:ec2:Route                  vpc-public-3   created (1s)
 +         │  │  └─ aws:ec2:RouteTableAssociation  vpc-public-3   created (1s)
 +         │  ├─ aws:ec2:Eip                       vpc-3          created (1s)
 +         │  └─ aws:ec2:NatGateway                vpc-3          created (125s)
 +         ├─ aws:ec2:Subnet                       vpc-public-2   created (11s)
 +         │  ├─ aws:ec2:Eip                       vpc-2          created (1s)
 +         │  ├─ aws:ec2:RouteTable                vpc-public-2   created (1s)
 +         │  │  ├─ aws:ec2:Route                  vpc-public-2   created (1s)
 +         │  │  └─ aws:ec2:RouteTableAssociation  vpc-public-2   created (1s)
 +         │  └─ aws:ec2:NatGateway                vpc-2          created (95s)
 +         ├─ aws:ec2:Subnet                       vpc-private-1  created (1s)
 +         │  └─ aws:ec2:RouteTable                vpc-private-1  created (1s)
 +         │     ├─ aws:ec2:RouteTableAssociation  vpc-private-1  created (0.51s)
 +         │     └─ aws:ec2:Route                  vpc-private-1  created (1s)
 +         ├─ aws:ec2:Subnet                       vpc-private-2  created (2s)
 +         │  └─ aws:ec2:RouteTable                vpc-private-2  created (1s)
 +         │     ├─ aws:ec2:RouteTableAssociation  vpc-private-2  created (0.72s)
 +         │     └─ aws:ec2:Route                  vpc-private-2  created (1s)
 +         └─ aws:ec2:InternetGateway              vpc            created (2s)

Outputs:
    privateSubnetIds: [
        [0]: "subnet-0b5f7a7f8a4a88d51"
        [1]: "subnet-03dac3c6561fe7140"
        [2]: "subnet-089ba98ecf74614c0"
    ]
    publicSubnetIds : [
        [0]: "subnet-02783beae211c9bab"
        [1]: "subnet-0460f52b1c1786dee"
        [2]: "subnet-0afa4ac06444be3c5"
    ]
    vpcId           : "vpc-0bba07367def55a87"

Resources:
    + 34 created

Duration: 2m27s

```


If unspecified, this VPC will use the following defaults:

*   An IPv4 CIDR block of `10.0.0.0/16`.
*   The first `3` availability zones inside of your region.
*   A public and private subnet per availability zone.
*   Equally partitioned CIDR address spaces per subnet (per availability zone).
*   A NAT Gateway and EIP per public subnet.
*   A single Internet Gateway for all public subnets to use.

The following sections show how to explicitly manage any or all of these settings.

Configuring CIDR Blocks for a VPC
---------------------------------

Although the default CIDR block of `10.0.0.0/16` is reasonable most of the time, it is easy to override.

> Classless Inter-Domain Routing (CIDR) is an Internet standard for specifying ranges of IP addresses. See RFC 4632 for more details.

To set our VPC’s CIDR block, pass a custom `cidrBlock` argument to `awsx.ec2.Vpc`’s constructor:

```
"use strict";
const awsx = require("@pulumi/awsx");

// Allocate a new VPC with a custom CIDR block.
const vpc = new awsx.ec2.Vpc("vpc", {
    cidrBlock: "172.16.8.0/24",
});

exports.vpcId = vpc.vpcId;

```


```
import * as awsx from "@pulumi/awsx";

// Allocate a new VPC with a custom CIDR block.
const vpc = new awsx.ec2.Vpc("vpc", {
    cidrBlock: "172.16.8.0/24",
});

export const vpcId = vpc.vpcId;

```


```
import pulumi
import pulumi_awsx as awsx

# Allocate a new VPC with a custom CIDR block.
vpc = awsx.ec2.Vpc("vpc", awsx.ec2.VpcArgs(
    cidr_block="172.16.8.0/24",
))

pulumi.export("vpcId", vpc.vpc_id)

```


```
package main

import (
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		// Allocate a new VPC with a custom CIDR block.
		vpc, err := ec2.NewVpc(ctx, "vpc", &ec2.VpcArgs{
			CidrBlock: pulumi.StringRef("172.16.8.0/24"),
		})
		if err != nil {
			return err
		}

		ctx.Export("vpcId", vpc.VpcId)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    // Allocate a new VPC with a custom CIDR block.
    var vpc = new Vpc("vpc", new()
    {
        CidrBlock = "172.16.8.0/24",
    });

    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
    };
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.VpcArgs;
import com.pulumi.awsx.ec2.Vpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            // Allocate a new VPC with a custom CIDR block.
            var vpc = new Vpc("vpc", VpcArgs.builder()
                .cidrBlock("172.16.8.0/24")
                .build());

            ctx.export("vpcId", vpc.vpcId());
        });
    }
}

```


```
name: awsx-vpc-cidr-yaml
runtime: yaml
description: An example that creates a new VPC and configuring a custom CIDR block.
resources:
  # Allocate a new VPC with a custom CIDR block.
  vpc:
    type: awsx:ec2:Vpc
    properties:
      cidrBlock: 172.16.8.0/24

outputs:
  vpcId: ${vpc.vpcId}

```


This decreases the number of available IP addresses in our VPC from the default of 65,536 addresses (`/16` netmask) to 256 addresses (`/24` netmask), in addition to changing the IP address prefix from `10.0.0.0` to `172.16.8.0`.

> A VPC can have a minimum of 16 addresses, using the CIDR netmask `/28`, and a maximum of 65,536 addresses, using the netmask `/16`. The addresses are allocated across availability zones which may incur additional constraints.

In addition to configuring the CIDR block for your entire VPC, you can optionally assign a CIDR block to your VPC’s subnets. These must reside entirely within your VPC’s CIDR block. If you do not explicitly specify ranges, traffic will be evenly partitioned between availability zones within the VPC CIDR block range provided.

See IP Addressing in Your VPC for information about the full range of IP address and CIDR configuration available for your VPC.

Configuring Availability Zones for an AWS VPC
---------------------------------------------

A VPC spans all of the availability zones in your region. By default, however, the `awsx.ec2.Vpc` resource will only use 3 of them when allocating subnets and the associated gateways. This provides fault tolerance between three zones at a reasonable cost.

All regions support at least 3 availability zones, but many of them support more. If you’d like to improve the fault tolerance of your configuration, override this with the `numberOfAvailabilityZones` argument:

```
"use strict";
const awsx = require("@pulumi/awsx");

const vpc = new awsx.ec2.Vpc("vpc", {
    numberOfAvailabilityZones: 4,
});

exports.vpcId = vpc.vpcId;

```


```
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc", {
    numberOfAvailabilityZones: 4,
});

export const vpcId = vpc.vpcId;

```


```
import pulumi
import pulumi_awsx as awsx

vpc = awsx.ec2.Vpc("vpc", awsx.ec2.VpcArgs(
    number_of_availability_zones=4,
))

pulumi.export("vpcId", vpc.vpc_id)

```


```
package main

import (
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		vpc, err := ec2.NewVpc(ctx, "vpc", &ec2.VpcArgs{
			NumberOfAvailabilityZones: pulumi.IntRef(4),
		})
		if err != nil {
			return err
		}

		ctx.Export("vpcId", vpc.VpcId)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    var vpc = new Vpc("vpc", new()
    {
        NumberOfAvailabilityZones = 4,
    });

    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
    };
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.VpcArgs;
import com.pulumi.awsx.ec2.Vpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            var vpc = new Vpc("vpc", VpcArgs.builder()
                .numberOfAvailabilityZones(4)
                .build());

            ctx.export("vpcId", vpc.vpcId());
        });
    }
}

```


```
name: awsx-vpc-azs-yaml
runtime: yaml
description: An example that creates a new VPC and configuring availability zones.

resources:
  vpc:
    type: awsx:ec2:Vpc
    properties:
      numberOfAvailabilityZones: 4

outputs:
  vpcId: ${vpc.vpcId}

```


The VPC resource will internally adjust to fully consume 4 availability zones and split traffic accordingly.

If creating a VPC with the availability zone configuration set to 4 or higher, please ensure you are deploying in a region that supports more than 3 availability zones.

For information about regional support for availability zones, refer to AWS’s Global Infrastructures Regions and AZs help page.

Configuring Subnets for a VPC
-----------------------------

A VPC spans all of the availability zones in a region. You can additionally create one or more subnets in each availability zone, to increase your fault tolerance within a region and control routing.

By default, the `awsx.ec2.Vpc` class will allocate a public and a private subnet for each availability zone and evenly partition traffic amongst each of them. In the event that you do not wish to keep this default, you can override the behavior using its constructor’s `subnets` argument.

For example, this program replicates the default behavior but with an explicit specification:

```
"use strict";
const awsx = require("@pulumi/awsx");

const vpc = new awsx.ec2.Vpc("vpc", {
    subnetSpecs: [
        {
            type: awsx.ec2.SubnetType.Public,
            cidrMask: 22,
        },
        {
            type: awsx.ec2.SubnetType.Private,
            cidrMask: 20,
        },
    ],
});

exports.vpcId = vpc.vpcId;

```


```
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc", {
    subnetSpecs: [
        {
            type: awsx.ec2.SubnetType.Public,
            cidrMask: 22,
        },
        {
            type: awsx.ec2.SubnetType.Private,
            cidrMask: 20,
        },
    ],
});

export const vpcId = vpc.vpcId;

```


```
import pulumi
import pulumi_awsx as awsx

vpc = awsx.ec2.Vpc("vpc", awsx.ec2.VpcArgs(
    subnet_specs=[
        awsx.ec2.SubnetSpecArgs(
            type=awsx.ec2.SubnetType.PUBLIC,
            cidr_mask=22,
        ),
        awsx.ec2.SubnetSpecArgs(
            type=awsx.ec2.SubnetType.PRIVATE,
            cidr_mask=20,
        ),
    ],
))

pulumi.export("vpcId", vpc.vpc_id)

```


```
package main

import (
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		vpc, err := ec2.NewVpc(ctx, "vpc", &ec2.VpcArgs{
			SubnetSpecs: []ec2.SubnetSpecArgs{
				{
					Type:     ec2.SubnetTypePublic,
					CidrMask: pulumi.IntRef(22),
				},
				{
					Type:     ec2.SubnetTypePrivate,
					CidrMask: pulumi.IntRef(20),
				},
			},
		})
		if err != nil {
			return err
		}

		ctx.Export("vpcId", vpc.VpcId)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2.Inputs;
using Ec2 = Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    var vpc = new Ec2.Vpc("vpc", new()
    {
        SubnetSpecs =
        {
            new SubnetSpecArgs
            {
                Type = Ec2.SubnetType.Public,
                CidrMask = 22,
            },
            new SubnetSpecArgs
            {
                Type = Ec2.SubnetType.Private,
                CidrMask = 20,
            },
        },
    });

    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
    };
});

```


```
package myproject;

import java.util.Arrays;
import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.VpcArgs;
import com.pulumi.awsx.ec2.enums.SubnetType;
import com.pulumi.awsx.ec2.inputs.SubnetSpecArgs;
import com.pulumi.awsx.ec2.Vpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            var vpc = new Vpc("vpc", VpcArgs.builder()
                .subnetSpecs(new SubnetSpecArgs[]{
                    SubnetSpecArgs.builder()
                        .type(SubnetType.Public)
                        .cidrMask(22)
                        .build(),
                    SubnetSpecArgs.builder()
                        .type(SubnetType.Private)
                        .cidrMask(20)
                        .build()
                    })
                .build());

            ctx.export("vpcId", vpc.vpcId());
        });
    }
}

```


```
name: awsx-vpc-subnets-yaml
runtime: yaml
description: An example that creates a new VPC with a custom subnet specification.

resources:
  vpc:
    type: awsx:ec2:Vpc
    properties:
      subnetSpecs:
        - type: "Public"
          cidrMask: 22
        - type: "Private"
          cidrMask: 20

outputs:
  vpcId: ${vpc.vpcId}

```


The `subnetSpecs` argument takes an array of subnet specifications. Each one can include this information:

*   `type`: A required type of subnet to create. There are three kinds available:
    
    *   A `Public` subnet is one whose traffic is routed to an Internet Gateway (IGW).
    *   A `Private` subnet is one that is configured to use a NAT Gateway (NAT) so that it can reach the internet, but which prevents the internet from initiating connections to it.
    *   An `Isolated` subnet is one that cannot reach the internet either through an IGW or with NAT.
*   `cidrMask`: The number of leading bits in the VPC’s CIDR block to use to define the CIDR for this specific subnet. By providing masking bits, this ensures each subnet has a distinct block.
    
*   `name`: An optional name to use as part of the subnet name. If not provided, the type of the subnet will be used. As a result, this is required when making multiple subnets of the same type.
    

Refer to VPCs and Subnets for complete information about how VPCs and subnets relate in AWS and the configuration options available to you.

Configuring Internet and NAT Gateways for Subnets in a VPC
----------------------------------------------------------

By default, any VPC with public subnets will have a single Internet Gateway created for it. All public subnets will be routable for all IPv4 addresses connections through this gateway.

To allow connections from private subnets to the internet, NAT gateways will also be created. If not specified, one NAT Gateway will be created for each availability zone, to maximize fault tolerance. Because the NAT gateway must be in a public subnet, NAT gateways will only be created if there is at least one public subnet.

Fewer NAT gateways can be requested (e.g., to save on costs) using the `natGateways` property:

```
"use strict";
const awsx = require("@pulumi/awsx");

const vpc = new awsx.ec2.Vpc("vpc", {
    natGateways: {
        strategy: awsx.ec2.NatGatewayStrategy.Single,
    },
});

exports.vpcId = vpc.vpcId;

```


```
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc", {
    natGateways: {
        strategy: awsx.ec2.NatGatewayStrategy.Single,
    },
});

export const vpcId = vpc.vpcId;

```


```
import pulumi
import pulumi_awsx as awsx

vpc = awsx.ec2.Vpc("vpc", awsx.ec2.VpcArgs(
    nat_gateways=awsx.ec2.NatGatewayConfigurationArgs(
        strategy=awsx.ec2.NatGatewayStrategy.SINGLE,
    ),
))

pulumi.export("vpcId", vpc.vpc_id)

```


```
package main

import (
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		vpc, err := ec2.NewVpc(ctx, "vpc", &ec2.VpcArgs{
			NatGateways: &ec2.NatGatewayConfigurationArgs{
				Strategy: ec2.NatGatewayStrategySingle,
			},
		})
		if err != nil {
			return err
		}

		ctx.Export("vpcId", vpc.VpcId)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Pulumi.Awsx.Ec2.Inputs;
using Ec2 = Pulumi.Awsx.Ec2;

return await Deployment.RunAsync(() =>
{
    var vpc = new Ec2.Vpc("vpc", new()
    {
        NatGateways = new NatGatewayConfigurationArgs
        {
            Strategy = Ec2.NatGatewayStrategy.Single,
        },
    });

    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
    };
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.awsx.ec2.VpcArgs;
import com.pulumi.awsx.ec2.enums.NatGatewayStrategy;
import com.pulumi.awsx.ec2.inputs.NatGatewayConfigurationArgs;
import com.pulumi.awsx.ec2.Vpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            var vpc = new Vpc("vpc", VpcArgs.builder()
                .natGateways(
                    NatGatewayConfigurationArgs.builder()
                        .strategy(NatGatewayStrategy.Single)
                        .build()
                )
                .build());

            ctx.export("vpcId", vpc.vpcId());
        });
    }
}

```


```
name: awsx-vpc-nat-gateways-yaml
runtime: yaml
description: An example that creates a new VPC with a custom NAT gateway strategy.

resources:
  vpc:
    type: awsx:ec2:Vpc
    properties:
      natGateways:
        strategy: Single

outputs:
  vpcId: ${vpc.vpcId}

```


In the case where there is one NAT gateway per availability zone, then routing is very simple. Each private subnet will have have connections routed through gateway in that availability zone.

In the case where there are fewer NAT gateways than availability zones, however, routing works differently. If there are _N_ NAT gateways requested, then the first _N_ availability zones will get a NAT gateway. Routing to private subnets in those availability zones works as above. However, all remaining availability zones will have their private subnets routed to in a round-robin fashion from the availability zones with NAT gateways.

> Warning: While reducing the number of NAT gateways will save money, it also introduces risk as failure of one availability zone may impact others.

Configuring Security Groups for a VPC
-------------------------------------

A security group acts as a virtual firewall for your instance (e.g EC2) to control inbound and outbound traffic. Security groups act at the instance level, not the subnet level. Therefore, each instance in a subnet in your VPC can be assigned to a different set of security groups.

For security groups, you add _rules_ that control how traffic is permitted in the form of _ingress rules_ (for inbound traffic) and _egress rules_ (outbound traffic). In addition to specifying what network protocol and ports these rules apply to, you can also specify source and destination locations using CIDR blocks and other notations.

Each VPC has a default security group that disallows all ingress from any external source, and permits all outbound traffic. This will be used by default, however you may allocate and assign resources to different groups explicitly.

Here is a program that allocates a new group with a few rules:

```
"use strict";
const aws = require("@pulumi/aws");
const awsx = require("@pulumi/awsx");

const vpc = new awsx.ec2.Vpc("vpc");

const securityGroup = new aws.ec2.SecurityGroup("group", {
    vpcId: vpc.vpcId,
    ingress: [
        {
            fromPort: 22,
            toPort: 22,
            protocol: "tcp",
            cidrBlocks: ["203.0.113.25/32"],
        },
        {
            fromPort: 443,
            toPort: 443,
            protocol: "tcp",
            cidrBlocks: ["0.0.0.0/0"],
        },
    ],
    egress: [
        {
            fromPort: 0,
            toPort: 0,
            protocol: "-1",
            cidrBlocks: ["0.0.0.0/0"],
        },
    ],
});

exports.vpcId = vpc.vpcId;

```


```
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc");

const securityGroup = new aws.ec2.SecurityGroup("group", {
    vpcId: vpc.vpcId,
    ingress: [
        {
            fromPort: 22,
            toPort: 22,
            protocol: "tcp",
            cidrBlocks: ["203.0.113.25/32"],
        },
        {
            fromPort: 443,
            toPort: 443,
            protocol: "tcp",
            cidrBlocks: ["0.0.0.0/0"],
        },
    ],
    egress: [
        {
            fromPort: 0,
            toPort: 0,
            protocol: "-1",
            cidrBlocks: ["0.0.0.0/0"],
        },
    ],
});

export const vpcId = vpc.vpcId;

```


```
import pulumi
import pulumi_aws as aws
import pulumi_awsx as awsx

vpc = awsx.ec2.Vpc("vpc")

security_group = aws.ec2.SecurityGroup(
    "group",
    vpc_id=vpc.vpc_id,
    ingress=[
        aws.ec2.SecurityGroupIngressArgs(
            from_port=22,
            to_port=22,
            protocol="tcp",
            cidr_blocks=["203.0.113.25/32"],
        ),
        aws.ec2.SecurityGroupIngressArgs(
            from_port=443,
            to_port=443,
            protocol="tcp",
            cidr_blocks=["0.0.0.0/0"],
        ),
    ],
    egress=[
        aws.ec2.SecurityGroupEgressArgs(
            from_port=0,
            to_port=0,
            protocol="-1",
            cidr_blocks=["0.0.0.0/0"],
        )
    ],
)

pulumi.export("vpcId", vpc.vpc_id)

```


```
package main

import (
	awsec2 "github.com/pulumi/pulumi-aws/sdk/v6/go/aws/ec2"
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		vpc, err := ec2.NewVpc(ctx, "vpc", nil)
		if err != nil {
			return err
		}

		_, err = awsec2.NewSecurityGroup(ctx, "group", &awsec2.SecurityGroupArgs{
			VpcId: vpc.VpcId,
			Ingress: awsec2.SecurityGroupIngressArray{
				&awsec2.SecurityGroupIngressArgs{
					FromPort: pulumi.Int(22),
					ToPort:   pulumi.Int(22),
					Protocol: pulumi.String("tcp"),
					CidrBlocks: pulumi.StringArray{
						pulumi.String("203.0.113.25/32"),
					},
				},
				&awsec2.SecurityGroupIngressArgs{
					FromPort: pulumi.Int(443),
					ToPort:   pulumi.Int(443),
					Protocol: pulumi.String("tcp"),
					CidrBlocks: pulumi.StringArray{
						pulumi.String("0.0.0.0/0"),
					},
				},
			},
			Egress: awsec2.SecurityGroupEgressArray{
				&awsec2.SecurityGroupEgressArgs{
					FromPort: pulumi.Int(0),
					ToPort:   pulumi.Int(0),
					Protocol: pulumi.String("-1"),
					CidrBlocks: pulumi.StringArray{
						pulumi.String("0.0.0.0/0"),
					},
				},
			},
		})
		if err != nil {
			return err
		}

		ctx.Export("vpcId", vpc.VpcId)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Aws = Pulumi.Aws;
using Awsx = Pulumi.Awsx;

return await Deployment.RunAsync(() =>
{
    var vpc = new Awsx.Ec2.Vpc("vpc");

    var securityGroup = new Aws.Ec2.SecurityGroup("group", new Aws.Ec2.SecurityGroupArgs
    {
        VpcId = vpc.VpcId,
        Ingress =
        {
            new Aws.Ec2.Inputs.SecurityGroupIngressArgs
            {
                FromPort = 22,
                ToPort = 22,
                Protocol = "tcp",
                CidrBlocks =
                {
                    "203.0.113.25/32"
                },
            },
            new Aws.Ec2.Inputs.SecurityGroupIngressArgs
            {
                FromPort = 443,
                ToPort = 443,
                Protocol = "tcp",
                CidrBlocks =
                {
                    "0.0.0.0/0"
                },
            },
        },
        Egress =
        {
            new Aws.Ec2.Inputs.SecurityGroupEgressArgs
            {
                FromPort = 0,
                ToPort = 0,
                Protocol = "-1",
                CidrBlocks =
                {
                    "0.0.0.0/0",
                },
            },
        },
    });


    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
    };
});

```


```
package myproject;

import com.pulumi.Pulumi;
import com.pulumi.aws.ec2.SecurityGroup;
import com.pulumi.aws.ec2.SecurityGroupArgs;
import com.pulumi.aws.ec2.inputs.SecurityGroupEgressArgs;
import com.pulumi.aws.ec2.inputs.SecurityGroupIngressArgs;
import com.pulumi.awsx.ec2.Vpc;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            var vpc = new Vpc("vpc");

            var securityGroup = new SecurityGroup("group", SecurityGroupArgs.builder()
                .vpcId(vpc.vpcId())
                .ingress(SecurityGroupIngressArgs.builder()
                    .fromPort(22)
                    .toPort(22)
                    .protocol("tcp")
                    .cidrBlocks("203.0.113.25/32")
                    .build())
                .ingress(SecurityGroupIngressArgs.builder()
                    .fromPort(443)
                    .toPort(443)
                    .protocol("tcp")
                    .cidrBlocks("0.0.0.0/0")
                    .build())
                .egress(SecurityGroupEgressArgs.builder()
                    .fromPort(0)
                    .toPort(0)
                    .protocol("-1")
                    .cidrBlocks("0.0.0.0/0")
                    .build())
                .build()
            );

            ctx.export("vpcId", vpc.vpcId());
        });
    }
}

```


```
name: awsx-vpc-security-groups-yaml
runtime: yaml
description: An example that creates a new VPC with a custom security group.

resources:
  vpc:
    type: awsx:ec2:Vpc
    properties:
      natGateways:
        strategy: Single

  securityGroup:
    type: aws:ec2:SecurityGroup
    properties:
      vpcId: ${vpc.vpcId}
      ingress:
        - fromPort: 22
          toPort: 22
          protocol: tcp
          cidrBlocks:
            - "203.0.113.25/32"
        - fromPort: 443
          toPort: 443
          protocol: tcp
          cidrBlocks:
            - "0.0.0.0/0"
      egress:
        - fromPort: 0
          toPort: 0
          protocol: "-1"
          cidrBlocks:
            - 0.0.0.0/0

outputs:
  vpcId: ${vpc.vpcId}

```


For additional details about configuring security group rules, See the Security Groups for Your VPC documentation.

How to use your VPC, Security Group, and EC2 instance
-----------------------------------------------------

This example shows how to deploy an EC2 instance using a VPC and security group provisioned with the Crosswalk for AWS component:

```
"use strict";
const aws = require("@pulumi/aws");
const awsx = require("@pulumi/awsx");

const vpc = new awsx.ec2.Vpc("vpc");

const securityGroup = new aws.ec2.SecurityGroup("group", {
    vpcId: vpc.vpcId,
});

const ami = aws.ec2.getAmiOutput({
    filters: [{ name: "name", values: ["amzn2-ami-hvm-*"] }],
    owners: ["amazon"],
    mostRecent: true,
});

const instance = new aws.ec2.Instance("instance", {
    ami: ami.id,
    instanceType: "t2.micro",
    vpcSecurityGroupIds: [securityGroup.id],
    subnetId: vpc.publicSubnetIds.apply(ids => ids[0]),
});

exports.vpcId = vpc.vpcId;

```


```
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

const vpc = new awsx.ec2.Vpc("vpc");

const securityGroup = new aws.ec2.SecurityGroup("group", {
    vpcId: vpc.vpcId,
});

const ami = aws.ec2.getAmiOutput({
    filters: [{ name: "name", values: ["amzn2-ami-hvm-*"] }],
    owners: ["amazon"],
    mostRecent: true,
});

const instance = new aws.ec2.Instance("instance", {
    ami: ami.id,
    instanceType: "t2.micro",
    vpcSecurityGroupIds: [securityGroup.id],
    subnetId: vpc.publicSubnetIds.apply(ids => ids[0]),
});

export const vpcId = vpc.vpcId;

```


```
import pulumi
import pulumi_aws as aws
import pulumi_awsx as awsx

vpc = awsx.ec2.Vpc("vpc")

security_group = aws.ec2.SecurityGroup(
    "group",
    vpc_id=vpc.vpc_id,
)

ami = aws.ec2.get_ami_output(
    most_recent=True,
    owners=["amazon"],
    filters=[aws.ec2.GetAmiFilterArgs(name="name", values=["amzn2-ami-hvm-*"])],
)

instance = aws.ec2.Instance(
    "instance",
    ami=ami.id,
    instance_type="t2.micro",
    vpc_security_group_ids=[security_group.id],
    subnet_id=vpc.public_subnet_ids.apply(lambda ids: ids[0]),
)

pulumi.export("vpcId", vpc.vpc_id)

```


```
package main

import (
	awsec2 "github.com/pulumi/pulumi-aws/sdk/v6/go/aws/ec2"
	"github.com/pulumi/pulumi-awsx/sdk/v2/go/awsx/ec2"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {

		vpc, err := ec2.NewVpc(ctx, "vpc", nil)
		if err != nil {
			return err
		}

		securityGroup, err := awsec2.NewSecurityGroup(ctx, "group", &awsec2.SecurityGroupArgs{
			VpcId: vpc.VpcId,
		})
		if err != nil {
			return err
		}

		ami, err := awsec2.LookupAmi(ctx, &awsec2.LookupAmiArgs{
			Filters: []awsec2.GetAmiFilter{
				{
					Name:   "name",
					Values: []string{"amzn2-ami-hvm-*"},
				},
			},
			Owners:     []string{"amazon"},
			MostRecent: pulumi.BoolRef(true),
		})
		if err != nil {
			return err
		}

		awsec2.NewInstance(ctx, "instance", &awsec2.InstanceArgs{
			Ami:                 pulumi.String(ami.Id),
			InstanceType:        pulumi.String("t2.micro"),
			VpcSecurityGroupIds: pulumi.StringArray{securityGroup.ID()},
			SubnetId:            vpc.PublicSubnetIds.Index(pulumi.Int(0)),
		})
		if err != nil {
			return err
		}

		ctx.Export("vpcId", vpc.VpcId)
		return nil
	})
}

```


```
﻿using Pulumi;
using System.Collections.Generic;
using Aws = Pulumi.Aws;
using Awsx = Pulumi.Awsx;

return await Deployment.RunAsync(() =>
{
    var vpc = new Awsx.Ec2.Vpc("vpc");

    var securityGroup = new Aws.Ec2.SecurityGroup("group", new Aws.Ec2.SecurityGroupArgs
    {
        VpcId = vpc.VpcId,
    });

    var ami = Aws.Ec2.GetAmi.Invoke(new()
    {
        Filters = new[]
        {
            new Aws.Ec2.Inputs.GetAmiFilterInputArgs
            {
                Name = "name",
                Values = new[] { "amzn2-ami-hvm-*" },
            },
        },
        Owners = new[] { "amazon" },
        MostRecent = true,
    });

    var instance = new Aws.Ec2.Instance("instance", new Aws.Ec2.InstanceArgs
    {
        Ami = ami.Apply(ami => ami.Id),
        InstanceType = "t2.micro",
        VpcSecurityGroupIds =
        {
            securityGroup.Id,
        },
        SubnetId = vpc.PublicSubnetIds.Apply(ids => ids[0]),
    });

    return new Dictionary<string, object?>
    {
        ["vpcId"] = vpc.VpcId,
    };
});

```


```
package myproject;

import java.util.List;

import com.pulumi.Pulumi;
import com.pulumi.aws.ec2.Ec2Functions;
import com.pulumi.aws.ec2.Instance;
import com.pulumi.aws.ec2.InstanceArgs;
import com.pulumi.aws.ec2.SecurityGroup;
import com.pulumi.aws.ec2.SecurityGroupArgs;
import com.pulumi.aws.ec2.inputs.GetAmiArgs;
import com.pulumi.aws.ec2.inputs.GetAmiFilterArgs;
import com.pulumi.aws.ec2.outputs.GetAmiResult;
import com.pulumi.awsx.ec2.Vpc;
import com.pulumi.core.Output;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {

            var vpc = new Vpc("vpc");

            var securityGroup = new SecurityGroup("group", SecurityGroupArgs.builder()
                .vpcId(vpc.vpcId())
                .build()
            );

            var ami = Ec2Functions.getAmi(GetAmiArgs.builder()
                .filters(List.of(GetAmiFilterArgs.builder()
                    .name("name")
                    .values(List.of("amzn2-ami-hvm-*"))
                    .build()))
                .owners(List.of("amazon"))
                .mostRecent(true)
                .build());

            var server = new Instance("web-server-www", InstanceArgs.builder()
                .ami(ami.applyValue(GetAmiResult::id))
                .instanceType("t2.micro")
                .vpcSecurityGroupIds(Output.all(securityGroup.id()))
                .subnetId(vpc.publicSubnetIds().applyValue(ids -> ids.get(0)))
                .build()
            );

            ctx.export("vpcId", vpc.vpcId());
        });
    }
}

```


```
name: awsx-vpc-sg-ec2-yaml
runtime: yaml
description: An example that deploys an EC2 instance using a VPC and security group with Crosswalk for AWS.

variables:
  amiId:
    fn::invoke:
      function: aws:ec2:getAmi
      arguments:
        filters:
          - name: "name"
            values: ["amzn2-ami-hvm-*"]
        owners: ["amazon"]
        mostRecent: true
      return: id

resources:
  vpc:
    type: awsx:ec2:Vpc
    properties:
      natGateways:
        strategy: Single

  securityGroup:
    type: aws:ec2:SecurityGroup
    properties:
      vpcId: ${vpc.vpcId}

  instance:
    type: aws:ec2:Instance
    properties:
      ami: ${amiId}
      instanceType: t2.micro
      vpcSecurityGroupIds:
        - ${securityGroup.id}
      subnetId: ${vpc.publicSubnetIds[0]}

outputs:
  vpcId: ${vpc.vpcId}

```


If we run `pulumi up`, the `aws.ec2.Instance` will be provisioned using the _first_ public subnet from the `awsx.ec2.Vpc` component and the security group provisioned with the `awsx.ec2.SecurityGroup` component:

```
$ pulumi up

Updating (dev)

     Type                                          Name           Status
 +   pulumi:pulumi:Stack                           awsx-vpc-dev   created (166s)
 +   ├─ awsx:ec2:Vpc                               vpc            created (0.66s)
 +   │  └─ aws:ec2:Vpc                             vpc            created (1s)
 +   │     ├─ aws:ec2:Subnet                       vpc-private-2  created (0.97s)
 +   │     │  └─ aws:ec2:RouteTable                vpc-private-2  created (0.81s)
 +   │     │     ├─ aws:ec2:RouteTableAssociation  vpc-private-2  created (0.77s)
 +   │     │     └─ aws:ec2:Route                  vpc-private-2  created (1s)
 +   │     ├─ aws:ec2:InternetGateway              vpc            created (0.94s)
 +   │     ├─ aws:ec2:Subnet                       vpc-public-1   created (11s)
 +   │     │  ├─ aws:ec2:RouteTable                vpc-public-1   created (0.85s)
 +   │     │  │  ├─ aws:ec2:RouteTableAssociation  vpc-public-1   created (1s)
 +   │     │  │  └─ aws:ec2:Route                  vpc-public-1   created (2s)
 +   │     │  ├─ aws:ec2:Eip                       vpc-1          created (1s)
 +   │     │  └─ aws:ec2:NatGateway                vpc-1          created (85s)
 +   │     ├─ aws:ec2:Subnet                       vpc-private-1  created (1s)
 +   │     │  └─ aws:ec2:RouteTable                vpc-private-1  created (0.97s)
 +   │     │     ├─ aws:ec2:RouteTableAssociation  vpc-private-1  created (0.94s)
 +   │     │     └─ aws:ec2:Route                  vpc-private-1  created (1s)
 +   │     ├─ aws:ec2:Subnet                       vpc-public-2   created (11s)
 +   │     │  ├─ aws:ec2:RouteTable                vpc-public-2   created (1s)
 +   │     │  │  ├─ aws:ec2:RouteTableAssociation  vpc-public-2   created (1s)
 +   │     │  │  └─ aws:ec2:Route                  vpc-public-2   created (1s)
 +   │     │  ├─ aws:ec2:Eip                       vpc-2          created (1s)
 +   │     │  └─ aws:ec2:NatGateway                vpc-2          created (146s)
 +   │     ├─ aws:ec2:Subnet                       vpc-public-3   created (12s)
 +   │     │  ├─ aws:ec2:Eip                       vpc-3          created (1s)
 +   │     │  ├─ aws:ec2:RouteTable                vpc-public-3   created (1s)
 +   │     │  │  ├─ aws:ec2:RouteTableAssociation  vpc-public-3   created (1s)
 +   │     │  │  └─ aws:ec2:Route                  vpc-public-3   created (1s)
 +   │     │  └─ aws:ec2:NatGateway                vpc-3          created (95s)
 +   │     └─ aws:ec2:Subnet                       vpc-private-3  created (2s)
 +   │        └─ aws:ec2:RouteTable                vpc-private-3  created (0.70s)
 +   │           ├─ aws:ec2:RouteTableAssociation  vpc-private-3  created (13s)
 +   │           └─ aws:ec2:Route                  vpc-private-3  created (1s)
 +   ├─ aws:ec2:SecurityGroup                      group          created (2s)
 +   └─ aws:ec2:Instance                           instance       created (32s)

Outputs:
    vpcId: "vpc-02d379aaaa281f99d"

Resources:
    + 36 created

```


Setting Up a New VPC the Hard Way
---------------------------------

The `awsx.ec2.Vpc` component encapsulates a lot of details, including subnets, route tables, gateways, in addition to the VPC resource itself. The `aws.ec2` package, on the other hand, out of which `Vpc` is built, provides all of these raw resources so that you can code directly to the underlying AWS resource types, exposing every supported capability.

For information about configuring each of these resources, refer to each type’s API documentation:

*   Vpc
*   Subnet
*   InternetGateway
*   NatGateway
*   SecurityGroup

These resources can be independently allocated, just as with the `Vpc` class shown above. They will need to be connected together manually, however, which can provide greater flexibility but at a greater implementation cost.

Note that the constituent parts, in the form of these raw resources, are available as properties on the resulting `Vpc` class. For instance, `internetGateway` will return the Internet Gateway object for a VPC.

Additional VPC Resources
------------------------

For more information about VPCs, read the following:

*   Amazon Virtual Private Cloud homepage
