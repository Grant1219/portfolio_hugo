+++
title = "Terraform and non-uniform environments"
description = "My thoughts from using Terraform over the past year or so"
tags = ["development", "terraform"]
categories = ["Development"]
# series = []
date = 2021-04-09T21:11:56-07:00
+++

My dealings with Terraform in the past were mainly for my own curiousity, and to understand what all the "infrastructure as code" buzz was about.
Until recently I didn't have the opportunity to use Terraform in a large project professionally, but over the past year or so that changed, and so did my opinion of Terraform.

### Terraform is great, as long as your environments are relatively uniform

Since Terraform is declarative rather than procedural, you need to define the end state you want, and it is meant to figure out how to get there.

As anyone somewhat familiar with Terraform knows, the language has no "if" blocks, the closest you can get is this weird hack using "count":
```
resource "telemetry_alarm" "test_alarm" {
  # only create this resource if alarms are enabled
  count = var.enable_alarms ? 1 : 0

  ...
}
```

And the shorthand conditional assignment you'll find in many C-like languages:
```
locals {
  instance_count = var.is_test_env ? 1 : 3
}
```

This is all fine until the number of special cases in your environment continues to increase, and these conditional lines end up strewn throughout your entire codebase.
Not only is it ugly, but it's error prone because you need to make sure the conditionals are consistent across your modules, and update them all when a special case changes.
What can be done about it?

### Handle all the special cases first, and give your modules a uniform data set

One solution is to create a module that takes all the input about the environments, filter, sort, rearrange, and output it in a consistent format that can then be iterated over
and passed to your business logic modules later on. Below is a short example of what I mean.

The data input to start with, this is usually generic enough that it can be shared between all teams in an organization and easily maintained/updated.
```
locals {
  regions = [{
    name = "test1"
    ad_count = 1
    type = "test"
  }, {
    name = "region1"
    ad_count = 2
    type = "prod"
  }, {
    name = "region2"
    ad_count = 3
    type = "prod"
  }, {
    name = "region4"
    ad_count = 2
    type = "prod"
  }]
}
```

This input data could come from a static file or be passed from some other source, but the goal is to get it into a better format so your modules can use it.
```
variable "all_regions" {}

locals {
  # add the custom parameters needed to each region
  regions_extended = [
    for r in var.all_regions :
      merge(r, {
        # standard options wanted for production regions
        instance_count = r.ad_count * 4
        deploy_branch = "stable"
        domain = "prod.example.com"
        alarms = true
      }, (r.type == "test") ? {
        # these options will overwrite the above for the test regions
        instance_count = r.ad_count * 2
        deploy_branch = "development"
        domain = "dev.example.com"
        alarms = false
      } : {}, (r.name == "region4") ? {
        # is there an oddball region? no problem
        deploy_branch = "old_stable"
        domain = "old.example.com"
      } : {})...
    ]

  # split up the test regions with the production ones (and convert them to maps)
  test_regions = { for r in local.regions_extended : r.name => r if r.type == "test" }
  prod_regions = { for r in local.regions_extended : r.name => r if r.type == "prod" }

  # do more stuff here as needed...
}

output "test_regions" {
  value = local.test_regions
}

output "prod_regions" {
  value = local.prod_regions
}
```

Now that you have your module that outputs all your deployment regions with the extra special case data you need, just pass them along to your business modules.
Note that `for_each` on `module` blocks only works in Terraform 0.13 and beyond.
```
module "generator" {
  source "./modules/generator"

  all_regions = local.regions
}

locals {
  test_regions = module.generator.test_regions
  prod_regions = module.generator.prod_regions
}

module "test_region" {
  source "./modules/test_region"
  for_each = local.test_regions

  name = each.key
  settings = each.value
}

module "prod_region" {
  source "./modules/prod_region"
  for_each = local.prod_regions

  name = each.key
  settings = each.value
}
```

This has greatly improved the cleanliness of our downstream Terraform modules because you can just pass the exact generated values to each of the resources.
Although I tried to keep this example very short, in reality the code that processes the environment data can get messy. Due to the limited nature of some of the
Terraform DSL (domain specific language), I found myself fighting with the language when I wanted to do more advanced processing or restructing of the data.
Maybe I'm just bad at Terraform and need to improve, but this leads to my final point.

### Just use a real language

As I continue to work with Terraform, and sometimes struggle with keeping my mind in a "Terraform" mode, I can't help but think all this would be easier if we
could just use Python or some other language that already exists, and simply write configuration modules similar to how providers are written for Terraform.

I understand that keeping Terraform limited helps from users shooting themselves in the foot with embedding complex logic into their infrastructure code
(I've seen some crazy Ruby embedded in Chef recipes), but personally I prefer to be trusted about when and where to use the advanced capability, instead of
being unable to do something at all. This is similar to arguments I've heard about the complexity of C++ being both its strength and its weakness, but I still
use C++ because it does give you that freedom when necessary.

I was surprised to find out something like this for infrastructure already does exist called [Pulumi](https://www.pulumi.com/). I have not tried it yet,
but it does look pretty cool and seems to support several languages already. I'm definitely planning to give it a try.

To be clear, I'm not hating on Terraform here. I've enjoyed learning it, and it really has helped clean up the infrastructure deployments. Sometimes I just
question why another DSL needs to be invented when we already have so many great languages to work with.
