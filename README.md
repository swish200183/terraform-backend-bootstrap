# terraform-backend-bootstrap

A small, standalone Terraform project that provisions the shared infrastructure needed for **remote state management** across other Terraform projects (such as `terraform-aws-infra`).

## What this creates

- **S3 bucket** — stores Terraform state files remotely, with versioning and server-side encryption enabled
- **DynamoDB table** — provides state locking to prevent concurrent `terraform apply` runs from corrupting state

## Why this project exists

Terraform's S3 backend requires the bucket and lock table to already exist before any project can use them as its backend — Terraform can't create the backend it's about to use for itself. This project solves that "chicken and egg" problem by provisioning those resources first, using local state.

This project is run once, and its resources are then reused as the backend for all other Terraform projects.

## Usage

```bash
terraform init
terraform plan
terraform apply
```

## Notes

- This project intentionally does **not** use a remote backend — it manages its own state locally, since it exists to bootstrap remote state for everything else.
- The S3 bucket has `prevent_destroy = true` set as a safety guard against accidental deletion via `terraform destroy`.
- The bucket name must be globally unique across all of AWS — update it if it's already taken.
- Run this project only once per AWS account/region. Other projects should reference these resources in their own `backend.tf`, not recreate them.

## Resources created

| Resource | Purpose |
|---|---|
| `aws_s3_bucket.terraform_state` | Remote state storage |
| `aws_s3_bucket_versioning.terraform_state` | State file version history |
| `aws_s3_bucket_server_side_encryption_configuration.terraform_state` | Encrypts state at rest |
| `aws_dynamodb_table.terraform_lock` | State locking to prevent concurrent writes |
