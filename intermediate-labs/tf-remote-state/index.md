# Terraform Lab 5

## Overview 
In this lab you will create an S3 bucket and migrate the Terraform state to a remote backend. 

## Create an S3 bucket 
AWS requires every S3 bucket to have a unique name. For this reason add your initials to the end of the bucket. The example below uses `jrs` as the initials.

For the following steps replace all references to 'remote-state-jrs' with your bucket name.

In the `tf-lab3/learn-terraform-variables` directory create a new file `s3.tf` declaring:
- aws_s3_bucket resource
- acl: private
- tags: `Name = remote state backend`


So that we can easily retrieve the name of the bucket in the future, add an output that displays the s3 bucket id

Using Terraform apply the changes. 

## Migrate the state
Now that we've created an S3 bucket, we need to migrate the state to the remote backend. 

When creating the backend configuration remember to replace the `bucket` with the name of the bucket you created. 

Create `backend.tf` declaring:
- `s3` backend
- region = `us-west-2`
- bucket = `remote-state-jrs`
- key = `state.tfstate`


## Reinitialize Terraform 
Now that you have created the S3 bucket and configured the `backend.tf` you must run `terraform init` to migrate the state to the new remote backend. 

If prompted to migrate the existing state type `yes`

If everything is successful you should see a message telling you the backend was migrated. 

# Congrats!

