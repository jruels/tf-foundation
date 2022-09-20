# Terraform Lab 4

## Overview 
Terraform supports several variable types including bool, string and numbers.

The following steps continue where the previous lab left off. The below changes should be applied in the `tf-lab3` directory.

## VPN gateway support

Use a `bool` type variable to control whether your VPC is configured with a VPN gateway. Add a declaration for the `enable_vpn_gateway` to `variables.tf`
- variable name: `enable_vpn_gateway`
- description: `Enable a VPN gateway in your VPC.`
- type: `bool`
- default: `false`

Add the new variable in `main.tf` and replace the existing `enable_vpn_gateway = false` with the variable.

Leave the value for `enable_nat_gateway` hard-coded. In any configuration, there may be some values that you want to allow users to configure with variables and others you don't.

When you write Terraform modules you intend to re-use, you will usually want to make as many attributes configurable with variables as possible, to make your module more flexible for use in more situations.

When you write Terraform configuration for a specific project, you may choose to leave some attributes with hard-coded values when it doesn't make sense to allow users to configure them.

## List public and private subnets
So far you have used "simple" variables. These variables all have single values. Now let's use some more complex variables. Terraform supports collection variable types that contain more than one value. Terraform supports several collection variables types. 
- List: A sequence of values of the same type.
- Map: A lookup table, matching keys to values, all of the same type.
- Set: An unordered collection of unique value, all of the same type.

The following lab will use lists and a map, which are the most commonly used of these types. Sets are useful when a unique collection of values is needed, and the order of the items in the collection does not matter.

A likely place to use list variables is when setting the `private_subnets` and `public_subnets` arguments for the VPC. Make this configuration easier to use while still being customizable by using lists along with the `slice()` function.

Add the following variable declarations to `variables.tf`.
- variable: `public_subnet_count`
- description: "Number of public subnets."
- type: number
- default: 2 


- variable: `private_subnet_count`
- description: "Number of private subnets."
- type: number
- default: 2 

- variable: `public_subnet_cidr_blocks`
- description: "Available cidr blocks for public subnets."
- type: list(string)
- default: list all blocks in the range of 10.0.1.0/24 - 10.0.8.0/24

- variable: `private_subnet_cidr_blocks`
- description: "Available cidr blocks for private subnets."
- type: list(string)
- default: list all blocks in the range of 10.0.101.0/24 - 10.0.108.0/24

Notice that the type for the list variables is `list(string)`. Each element in these lists must be a string. List elements must all be the same type, but can be any type, including complex types like `list(list)` and `list(map)`.

Like lists and arrays used in most programming languages, you can refer to individual items in a list by index, starting with 0. Terraform also includes several functions that allow you to manipulate lists and other variable types.

Use the `slice()` function to get a subset of these lists.

The Terraform console command opens an interactive console that you can use to evaluate expressions in the context of your configuration. This can be very useful when working with and troubleshooting variable definitions.

Open a console with the `terraform console` command.

Now inspect the list of private subnet blocks in the Terraform console.

Refer to the variable by name to return the entire list.

Query the var `private_subnet_cidr_blocks`

Now retrieve the second element from the list: 


output: 
```sh
"10.0.102.0/24"
```

Now use the `slice()` function to return the first three elements from the list.

output:
```sh
tolist([
  "10.0.101.0/24",
  "10.0.102.0/24",
  "10.0.103.0/24",
])
```

The `slice()` function takes three arguments: the list to slice, the index to start from, and the number of elements. It returns a new list with the specified elements copied ("sliced") from the original list.

Leave the console by typing `exit` or pressing `Control-D`.

Now that we understand how the `slice()` function works it's time to use it in our configuration. 

In the `main.tf` use the slice function to extract a subnet of the cidr block lists when defining your VPC's public and private subnet configuration.

Remove from the `vpc` module section the hard coded values for: 
- `private_subnets`
- `public_subnets`

This way, users of this configuration can specify the number of public and private subnets they want without worrying about defining CIDR blocks.

## Map resource tags

Each of the resources and modules declared in `main.tf` includes two tags: `project_name` and `environment`. Assign these tags with a map variable type.

Declare a new map variable for resource tags in `variables.tf`.

- name: `resources_tags`
- description: "Tags to set for all resources"
- type: the type is a ``map``, and the the value is `string`
- default: add tags for `project = project-alpha`, and `environment = dev`

Setting the type to `map(string)` tells Terraform to expect strings for the values in the map. Map keys are always strings. Like dictionaries or maps from programming languages, you can retrieve values from a map with the corresponding key. See how this works with the Terraform console.

Retrieve the value for `environment` from the var `resource_tags`

output: 
```sh
"dev"
```

Exit the console 

Now, replace the hard-coded tags in `main.tf` with the new variable. 

The hard-coded tags are used five times in this configuration, be sure to replace them all.

Apply these changes.

The value of `project` tag has changed, so you will be prompted to apply the changes. Respond with `yes` to confirm the changes.

## Assign values when prompted
In the examples, so far, all of the variable have had a default declared. If there is no default Terraform will prompt you at run time for the value. 

Add a new variable with: 
- variable: `ec2_instance_type`
- description: AWS EC2 instance type.
- type is a string

Replace the reference to the EC2 instance type in `main.tf`

Apply this configuration now and provide `t2.micro` as the value for the requested variable.

## Assign values using tfvars file
Whenever you execute a plan, destroy, or apply with any variable unassigned, Terraform will prompt you for a value. Entering variable values manually is time consuming and error prone, so Terraform provides several other ways to assign values to variables.

Create a file named `terraform.tfvars` with: 
- variable: `resource_tags`, defining three tags `project = new project`, `environment = test`, and `owner = me@example.com`
- variable: `ec2_instance_type` with a vaule of `t2.nano`
- variable: `instance_count` with a value of `3`

Terraform automatically loads all files in the current directory with the exact name `terraform.tfvars` or matching `*.auto.tfvars`. You can also use the `-var-file` flag to specify other files by name.

These files use syntax similar to Terraform configuration files (HCL), but they cannot contain configuration such as resource definitions. Like Terraform configuration files, these files can also contain JSON.

Apply the configuration with these new values.

## Interpolate variables in strings

Terraform configuration supports string interpolation — inserting the output of an expression into a string. This allows you to use variables, local values, and the output of functions to create strings in your configuration.

Update the names of the security groups to use the project and environment values from the `resource_tags` map.


## Validate our variables
AWS load balancers have naming restrictions. They only allow a limited set of characters and have a limit of 32 characters. This configuration has a potential problem if someone supplies a name that does not respect these limitations. 

One way to deal with this is to use variable validation to restrict the possible values for the project and environment tags.

Replace the existing `resource tags` variable in `variables.tf` with the below code snippet. 

Add validation blocks to enforce character limits and character sets on both `project` and `environment` values.

Validation blocks must enforce: 
- Tags must be no more than 16 characters
- Tags may only container letters, numbers, and hyphens.

if the above criteria are not met the validation block must return and error_message saying: 
```
The <tag_name> tag must be no more than 16 characters, and only container letters, numbers, and hyphens.``
```

```hcl
variable "resource_tags" {
  description = "Tags to set for all resources"
  type        = map(string)
  default     = {
    project     = "my-project",
    environment = "dev"
  }
```

The `regexall()` function takes a regular expression and a string to test it against, and returns a list of matches found in the string. In this case, the regular expression will match a string that contains anything other than a letter, number, or hyphen.

This way the maximum length of the load balancer name will never exceed 32, and it will not contain invalid characters. Using variable validation can be a good way to catch configuration errors early.

Apply this change to add validation to these two variables. 

Now test the validation rules by specifying an environment tag that is too long. Notice that the command will fail and return the error message specified in the validation block.

```sh
terraform apply -var='resource_tags={project="my-project",environment="development"}'
```

# Cleanup 
Run the following to clean up the resources 
```bash
terraform destroy -auto-approve
```
