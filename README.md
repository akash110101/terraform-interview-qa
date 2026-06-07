# Terraform Zero to Hero — Interview Q&A Notes

A complete question-and-answer study guide built from the **terraform-zero-to-hero** course (Day 1 → Day 8). Use it for quick revision before interviews. Questions move from fundamentals to scenario-based and "gotcha" questions interviewers actually ask.

---

## DAY 1 — Fundamentals, Installation, AWS Setup, First Code & State Basics

**Q1. What is Infrastructure as Code (IaC)?**
IaC is the practice of defining, provisioning, and managing infrastructure through machine-readable code instead of manual processes. Infrastructure (servers, networks, databases) is described in configuration files that can be version-controlled, reused, and automated. Tools include Terraform, AWS CloudFormation, and Azure Resource Manager templates.

**Q2. What problems existed before IaC?**
Manual server configuration (leading to inconsistencies/errors), no version control, heavy reliance on documentation that quickly went stale, very limited automation (basic scripting only), and slow provisioning of new resources/environments that delayed delivery.

**Q3. Why choose Terraform over other IaC tools? (Most common opener)**
- **Multi-cloud support** — cloud-agnostic; same workflow across AWS, Azure, GCP, on-prem.
- **Large ecosystem** — thousands of providers and reusable modules from HashiCorp and the community.
- **Declarative syntax** — you describe the *desired end-state*, not the steps.
- **State management** — tracks the real state so it can compute precise diffs.
- **Plan & apply workflow** — preview changes before applying them.
- **Strong community** and **integrations** (Docker, Kubernetes, Ansible, Jenkins).
- **HCL** — human-readable, purpose-built configuration language.

**Q4. Declarative vs imperative — which is Terraform and why does it matter?**
Terraform is **declarative**: you state *what* the final infrastructure should look like and Terraform figures out *how* to get there. Imperative tools require you to specify each step in order. Declarative is easier to read, maintain, and reason about, and it's idempotent.

**Q5. What is HCL?**
HashiCorp Configuration Language — the language used to write Terraform configs. It's designed specifically for describing infrastructure and is human-readable and expressive for both developers and operators.

**Q6. Define the core Terraform terminology.**
- **Provider** — plugin that lets Terraform talk to a platform's API (aws, azurerm, google, kubernetes…).
- **Resource** — an infrastructure component you create/manage (EC2 instance, S3 bucket, subnet).
- **Module** — a reusable, encapsulated package of Terraform code.
- **Configuration file** — `.tf` files describing desired state (usually `main.tf`).
- **Variable** — a placeholder/input that makes configs flexible and reusable.
- **Output** — a value Terraform exposes after creating/updating infrastructure.
- **State file** — `terraform.tfstate`; tracks current real-world state.
- **Plan** — a preview of the changes Terraform will make.
- **Apply** — executes the planned changes.
- **Workspace** — manages multiple environments with isolated state.
- **Remote backend** — non-local storage for state (S3, Azure Blob, Terraform Cloud).

**Q7. How do you install Terraform?**
Download from the HashiCorp downloads page for Windows/Linux/macOS. On Windows you can also use **GitHub Codespaces** (free hours/month) which gives an Ubuntu + VS Code VM, then follow the Linux install steps.

**Q8. How do you set up Terraform to work with AWS?**
1. Install the **AWS CLI**.
2. Create an **IAM user** with programmatic access and appropriate policies (e.g. `AmazonEC2FullAccess`); save the **Access Key ID** and **Secret Access Key**.
3. Run `aws configure` and enter the keys, default region, and output format.

**Q9. What does the most basic Terraform config look like?**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```
The `provider` block configures AWS; the `resource` block declares an EC2 instance with an AMI and instance type.

**Q10. Explain the Terraform lifecycle / core workflow.**
- `terraform init` — initializes the working directory and downloads the required provider plugins.
- `terraform plan` — analyzes config + current state and shows what it *will* do (no changes made).
- `terraform apply` — creates/updates/destroys resources to match the config (prompts for `yes`).
- `terraform destroy` — tears down everything defined in the config.

**Q11. What's the difference between desired state and current state?**
**Desired state** is what your `.tf` code says infrastructure should be. **Current state** is what actually exists (tracked in the state file). Terraform compares the two on `plan` and reconciles them on `apply`.

**Q12. Why is the state file so important?**
It records what Terraform has created, with attributes and dependencies, so Terraform knows exactly what changes are needed on the next run. Without it, Terraform couldn't map your config to real resources.

**Q13. (Scenario) You ran `terraform apply` and it created resources. You run `apply` again with no code changes — what happens?**
Nothing changes. Terraform compares desired vs current state, finds no diff, and reports "no changes." This is idempotency.

---

## DAY 2 — Providers, Variables, Outputs, Conditionals, Functions

**Q14. What is a provider, technically?**
A plugin that enables Terraform to interact with an API — cloud providers (aws, azurerm, google), SaaS, Kubernetes, etc. It exposes the resources Terraform can manage. During `init`, Terraform downloads the provider; during `apply` it uses the provider to make API calls.

**Q15. What are the three ways to configure providers?**
- **Root module** — most common; provider block in the root config, available to all resources.
- **Child module** — pass providers into a module via the `providers` argument (good for reuse).
- **`required_providers` block** — pins provider **source and version** for reproducibility.

**Q16. How do you use multiple providers in one project (e.g. AWS + Azure)?**
Define both provider blocks (often in `providers.tf`), then create resources for each:
```hcl
provider "aws"     { region = "us-east-1" }
provider "azurerm" { /* subscription_id, client_id, ... */ }

resource "aws_instance" "example"           { ... }
resource "azurerm_virtual_machine" "example"{ ... }
```

**Q17. How do you deploy to multiple regions of the same provider?**
Use the **`alias`** argument to create multiple configurations of the same provider, then point each resource at the right alias:
```hcl
provider "aws" { alias = "use1"; region = "us-east-1" }
provider "aws" { alias = "usw2"; region = "us-west-2" }

resource "aws_instance" "a" { provider = aws.use1; ... }
resource "aws_instance" "b" { provider = aws.usw2; ... }
```

**Q18. What is the `required_providers` block for?**
It declares each provider's **source** (e.g. `hashicorp/aws`) and **version constraints** so the same provider versions are used everywhere — preventing "works on my machine" drift.
```hcl
terraform {
  required_providers {
    aws     = { source = "hashicorp/aws", version = "~> 3.0" }
    azurerm = { source = "hashicorp/azurerm", version = ">= 2.0, < 3.0" }
  }
}
```

**Q19. Explain version constraint operators (`~>`, `>=`, etc.).**
- `~> 3.0` — allows `3.x` but not `4.0` (pessimistic/"allow rightmost increment").
- `>= 2.0, < 3.0` — a range: at least 2.0 but below 3.0.
- `=` pins an exact version; omitting it takes the latest acceptable.

**Q20. What are input variables and how are they defined/used?**
Inputs parameterize configs so they're reusable. Defined with a `variable` block (`description`, `type`, optional `default`) and referenced with `var.<name>`:
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
# usage
instance_type = var.instance_type
```

**Q21. What are output variables and why use them?**
Outputs expose values after creation (e.g. an instance's public IP, an ALB DNS name) for display or to pass between modules:
```hcl
output "public_ip" {
  value = aws_instance.example_instance.public_ip
}
```
From a module: `module.<module_name>.<output_name>`.

**Q22. What variable types does Terraform support?**
`string`, `number`, `bool`, plus collections like `list`, `map`, `set`, `tuple`, and `object`.

**Q23. What are `.tfvars` files and why use them?**
Files (`*.tfvars`) that **assign values** to declared variables, separating configuration from code. Benefits: separation of config from code, a place for environment-specific values, can hold sensitive values kept out of version control, and reusability across environments/teams. Use with:
```bash
terraform apply -var-file=dev.tfvars
```

**Q24. How is variable precedence handled (which value wins)?**
Terraform loads variables in increasing precedence: defaults → environment variables (`TF_VAR_*`) → `terraform.tfvars` / `*.auto.tfvars` → `-var-file` → `-var` on the CLI. Later sources override earlier ones.

**Q25. What is a conditional expression and its syntax?**
A ternary that returns one of two values based on a condition:
```hcl
condition ? true_val : false_val
```

**Q26. Give a real use of conditionals — conditionally creating a resource.**
```hcl
resource "aws_instance" "example" {
  count         = var.create_instance ? 1 : 0
  ami           = "ami-xxxx"
  instance_type = "t2.micro"
}
```
If `create_instance` is true → 1 instance; if false → 0 (skipped). This is the classic "feature flag" pattern using `count`.

**Q27. Give a conditional value-assignment example.**
Pick a CIDR based on environment:
```hcl
cidr_blocks = var.environment == "production" ? [var.production_subnet_cidr] : [var.development_subnet_cidr]
```
Or toggle SSH access: `cidr_blocks = var.enable_ssh ? ["0.0.0.0/0"] : []`.

**Q28. Name some built-in functions and what they do.**
- `concat(list1, list2)` — merge lists.
- `element(list, index)` — element at an index.
- `length(list)` — number of elements.
- `lookup(map, key)` — value for a key in a map (the course also uses a default in workspaces).
- `join(separator, list)` — list → single string.
- (`map(...)` builds a map — note: in modern Terraform you use `{}`/`tomap()` instead.)

**Q29. How do you debug and format Terraform?**
- **Formatting:** `terraform fmt` rewrites files to canonical style/indentation.
- **Validation:** `terraform validate` checks syntax/internal consistency.
- **Debugging:** set the `TF_LOG` environment variable (e.g. `TRACE`, `DEBUG`) to get detailed logs.

**Q30. (Project) Walk through the Day-2 VPC + EC2 + ALB project at a high level.**
It builds a full network stack: a **VPC** (`var.cidr`), two **public subnets** in different AZs, an **Internet Gateway**, a **route table** with a `0.0.0.0/0` route plus **associations**, a **security group** (allow HTTP 80 + SSH 22 in, all out), an **S3 bucket**, **two EC2 web servers** (each with `user_data`), and an **Application Load Balancer** with a **target group**, **target group attachments**, and an **HTTP listener** — outputting the **ALB DNS name**. It demonstrates wiring resources together via references like `aws_vpc.myvpc.id`.

---

## DAY 3 — Modules, Local Values, Data Sources, Registry

**Q31. What is a Terraform module?**
A reusable, self-contained package of Terraform code that groups related resources. Every Terraform configuration is technically a module (the **root module**); you call others as **child modules** via `source`.

**Q32. What are the benefits of modules?**
Modularity, reusability, simplified team collaboration, independent versioning/maintenance, abstraction of complexity, easier testing/validation, self-documentation (via inputs/outputs), scalability, and built-in security/compliance patterns.

**Q33. How do you call a module and pass inputs?**
```hcl
module "ec2_instance" {
  source              = "./modules/ec2_instance"
  ami_value           = "ami-053b0d53c279acc90"
  instance_type_value = "t2.micro"
  subnet_id_value     = "subnet-019ea91ed9b5252e7"
}
```
Inside the module, those become `variable` blocks (`var.ami_value`, etc.), and the module exposes results via `output` (e.g. `public-ip-address`).

**Q34. What can the `source` argument point to?**
A local path (`./modules/ec2_instance`), the **Terraform Registry** (`hashicorp/vpc/aws`), a Git repo, an HTTP URL, or an S3/GCS bucket.

**Q35. What are local values (`locals`) and when do you use them?**
Named expressions computed once and reused, to simplify complex/repeated expressions. Defined in a `locals` block and referenced as `local.<name>`. Use them to avoid repeating the same calculation and to make code readable. (Unlike variables, they can't be set from outside.)

**Q36. What are data sources?**
Read-only lookups that **fetch information about existing resources** or external systems (e.g. latest AMI, an existing VPC) without managing them. Declared with `data "<type>" "<name>" {}` and referenced as `data.<type>.<name>.<attr>`. They make configs dynamic instead of hardcoding IDs.

**Q37. Variable vs local vs data source — quick distinction.**
- **Variable** — input set from outside (CLI, tfvars, env).
- **Local** — internal named expression, computed in the config.
- **Data source** — value queried from the provider/external system at runtime.

**Q38. What is the Terraform Registry?**
A public catalog of community/HashiCorp-contributed **providers and modules** you can reuse instead of writing your own (e.g. an official AWS VPC module), saving time and encoding best practices.

---

## DAY 4 — Collaboration, Git, Backends, S3 + DynamoDB State

**Q39. Why use Git/version control with Terraform?**
To collaborate, track every infra change, review via pull requests, and roll back. Core commands: clone, pull, push.

**Q40. How do you handle sensitive data in version control?**
Never commit secrets or state. Use a **`.gitignore`** to exclude sensitive files (state files, `*.tfvars` with secrets, `.terraform/`). Keep secrets in a secret manager or environment variables instead.

**Q41. What is a Terraform backend?**
A backend determines **where the state file is stored** and how operations run. The **local** backend stores `terraform.tfstate` on disk; **remote** backends (S3, Azure Blob, Terraform Cloud) store it centrally for collaboration, security, and reliability.

**Q42. Why is storing state in a VCS a bad idea?**
- **Security risk** — state can contain secrets (keys/passwords) exposed to everyone with repo access.
- **Versioning complexity** — merge conflicts and inconsistent state when multiple people edit the same infra.

**Q43. How do you configure an S3 remote backend?**
```hcl
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "path/to/your/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "your-dynamodb-table"
  }
}
```
S3 stores state (with `encrypt = true`); DynamoDB handles locking.

**Q44. What is state locking and why does it matter?**
A mechanism that ensures **only one user/process modifies state at a time**, preventing concurrent writes that corrupt state or cause conflicts. With the S3 backend it's implemented via a **DynamoDB** table.

**Q45. How do you set up DynamoDB for state locking?**
Create a table whose **hash key is `LockID`** (type String):
```bash
aws dynamodb create-table --table-name terraform-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```
Then reference it via `dynamodb_table` in the backend block.

**Q46. (Scenario) Two engineers run `terraform apply` at the same time on the same state. What happens with DynamoDB locking enabled?**
The first acquires the lock (a `LockID` item is written); the second is blocked and gets a "state locked" error until the first finishes and releases the lock. This prevents state corruption.

**Q47. How is the backend infrastructure itself created in the course's Day-4 example?**
The same config provisions an EC2 instance, the **S3 bucket** (`aws_s3_bucket`), and the **DynamoDB table** (`aws_dynamodb_table` with `PAY_PER_REQUEST` billing and a `LockID` hash key) — i.e. it bootstraps its own remote-state resources.

---

## DAY 5 — Provisioners

**Q48. What is a provisioner?**
A mechanism to **execute actions on a resource during creation or destruction** — running scripts/commands or copying files to customize an instance beyond what the provider API does.

**Q49. What are the three provisioners covered, and what does each do?**
- **`file`** — copies files/directories from local machine to the remote instance.
- **`remote-exec`** — runs commands/scripts **on the remote machine** over SSH/WinRM.
- **`local-exec`** — runs commands **locally** on the machine running Terraform.

**Q50. Show a `file` provisioner.**
```hcl
provisioner "file" {
  source      = "local/path/file.txt"
  destination = "/path/on/remote/file.txt"
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
  }
}
```

**Q51. Show a `remote-exec` provisioner.**
```hcl
provisioner "remote-exec" {
  inline = [
    "sudo yum update -y",
    "sudo yum install -y httpd",
    "sudo systemctl start httpd",
  ]
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = aws_instance.example.public_ip
  }
}
```

**Q52. Show a `local-exec` provisioner.**
```hcl
resource "null_resource" "example" {
  triggers = { always_run = "${timestamp()}" }
  provisioner "local-exec" {
    command = "echo 'This is a local command'"
  }
}
```
The `timestamp()` trigger forces it to run on every apply.

**Q53. What is the `connection` block?**
It tells provisioners how to reach the remote machine — `type` (ssh/winrm), `user`, `private_key`/`password`, and `host`. Often `host = self.public_ip`.

**Q54. How do you handle provisioner failures?**
Use the **`on_failure`** attribute (`continue` to ignore and proceed, or `fail` to error out — the default), along with retry/timeout settings, to control behavior when a provisioner fails.

**Q55. (Best-practice gotcha) Are provisioners recommended?**
They're a **last resort**. HashiCorp recommends using built-in mechanisms (e.g. cloud-init / `user_data`, baked images via Packer, config management tools) first, because provisioners run only at create/destroy time, aren't tracked in state, and can leave resources in an inconsistent state on failure.

**Q56. What is `null_resource` used for?**
A resource that does nothing on its own but lets you attach provisioners or use `triggers` to run actions based on changing values — handy for `local-exec` glue logic.

---

## DAY 6 — Workspaces & Environment Management

**Q57. What is a Terraform workspace?**
A way to manage **multiple environments** (dev/stage/prod) from the same configuration, each with its **own isolated state file**, so the same code can produce separate infrastructure per environment.

**Q58. What are the key workspace commands?**
```bash
terraform workspace new dev        # create
terraform workspace list           # list
terraform workspace select prod    # switch
terraform workspace show           # current
```

**Q59. How do you make a config behave differently per workspace?**
Reference **`terraform.workspace`** and combine with `lookup`/maps. Example from the course — pick instance size by environment:
```hcl
variable "instance_type" {
  type = map(string)
  default = {
    dev   = "t2.micro"
    stage = "t2.medium"
    prod  = "t2.xlarge"
  }
}

instance_type = lookup(var.instance_type, terraform.workspace, "t2.micro")
```
`lookup` returns the workspace-specific value, defaulting to `t2.micro`.

**Q60. What are the benefits and limits of workspaces?**
Benefits: isolated state per environment, less code duplication, easy switching. Limits: all environments share one backend/config, so for strongly separated environments (different accounts, very different infra) many teams prefer **separate directories/state per environment** instead.

---

## DAY 7 — Security & HashiCorp Vault

**Q61. What are the main ways to secure sensitive data in Terraform?**
- **`sensitive = true`** attribute on variables/outputs so values aren't printed in console output.
- **Secret management systems** — HashiCorp Vault, AWS Secrets Manager.
- **Encrypted remote backend** — e.g. S3 with encryption or Terraform Cloud.
- **Environment variables** — keep secrets out of code (`TF_VAR_*`, exported env vars).

**Q62. What exactly does `sensitive = true` do (and not do)?**
It prevents the value from being shown in CLI output and plan output. **It does not encrypt the value in the state file** — the secret is still stored in state, so the state itself must be protected (encrypted remote backend + access control).

**Q63. What is HashiCorp Vault?**
A dedicated tool for **secret management and data protection** — storing, accessing, and distributing secrets (passwords, API keys, tokens) securely, with access policies and auditing.

**Q64. At a high level, how do you integrate Terraform with Vault?**
1. Run a Vault server (course uses an EC2 Ubuntu instance, dev mode for demo).
2. Enable an auth method (**AppRole**) and write a **policy** granting read/list (and CRUD on secret paths).
3. Create an AppRole and generate a **Role ID** (static) and **Secret ID** (dynamic).
4. Configure the **Vault provider** in Terraform with `auth_login` using those IDs, then read secrets via a data source and use them in resources.

**Q65. Show the Vault provider + secret data source pattern.**
```hcl
provider "vault" {
  address = "<vault-addr>:8200"
  auth_login {
    path = "auth/approle/login"
    parameters = { role_id = "<>", secret_id = "<>" }
  }
}

data "vault_kv_secret_v2" "example" {
  mount = "secret"
  name  = "test-secret"
}

resource "aws_instance" "my_instance" {
  ami           = "ami-053b0d53c279acc90"
  instance_type = "t2.micro"
  tags = { Secret = data.vault_kv_secret_v2.example.data["foo"] }
}
```

**Q66. Why AppRole instead of a static token?**
AppRole gives **machine-friendly auth** with short-lived, tightly scoped credentials (configurable TTLs and use counts on the Secret ID/token), reducing the blast radius if a credential leaks — better than embedding a long-lived root token.

**Q67. (Gotcha) If you pull a secret from Vault into a resource, is it safe in state?**
No — the resolved secret value ends up in the **state file**, so you still must encrypt and lock down state. Vault improves how secrets are *sourced*, not how state stores them.

---

## DAY 8 — Migration & Drift Detection

**Q68. What is configuration drift?**
**Drift** is when the real infrastructure differs from what Terraform's state/config says — typically because someone made manual ("out-of-band") changes in the cloud console or another tool.

**Q69. How do you detect drift?**
Run `terraform plan` (or `terraform plan -refresh-only`): Terraform refreshes state against the real world and shows the differences. A non-empty plan with no code change indicates drift.

**Q70. How do you remediate drift?**
- **Re-apply** to bring infrastructure back to the config's desired state, **or**
- **Update the code** to match the intentional manual change, **or** use `terraform refresh`/`-refresh-only` to reconcile state.

**Q71. What does it mean to "migrate to Terraform," and how do you bring existing resources under management?**
Migration = taking resources that already exist (created manually or by another tool) and managing them with Terraform. You **import** them:
- `terraform import <resource_address> <real_id>` brings an existing resource into state.
- In newer Terraform, an **`import` block** can import (and even auto-generate config) during `plan`/`apply`.
After importing, write matching config so future plans show no diff.

---

## CROSS-CUTTING / RAPID-FIRE INTERVIEW QUESTIONS

**Q72. Difference between `terraform plan` and `terraform apply`?**
`plan` previews changes without making any; `apply` actually executes them.

**Q73. What does `terraform init` do under the hood?**
Initializes the working directory: downloads providers, installs modules, and configures the backend.

**Q74. What is `terraform fmt` vs `terraform validate`?**
`fmt` formats code to canonical style; `validate` checks for syntax errors and internal consistency (it does not contact the cloud).

**Q75. `count` vs `for_each` — when to use which?**
`count` creates N identical instances indexed by number (good for simple replication / on-off toggles). `for_each` iterates a map/set and keys instances by a stable identifier (better when items can be added/removed without re-indexing/recreating others).

**Q76. What does `depends_on` do?**
Forces an explicit dependency/ordering when Terraform can't infer it from references — e.g. waiting on a module or resource before creating another.

**Q77. How do you reference another resource's attribute?**
`<resource_type>.<name>.<attribute>` — e.g. `aws_vpc.myvpc.id` or `aws_instance.example.public_ip`.

**Q78. What is `self` in a connection/provisioner block?**
A reference to the resource the block is attached to — e.g. `host = self.public_ip`.

**Q79. How would you structure a multi-environment project? (design question)**
Either **workspaces** (same code, isolated state via `terraform.workspace` + maps/`lookup`) or **separate directories/state per environment** with shared modules. Use remote state (S3 + DynamoDB lock), tfvars per environment, and pinned provider versions.

**Q80. What's the typical end-to-end workflow you'd describe in an interview?**
Write HCL → `terraform fmt` & `validate` → `terraform init` (providers + backend) → `terraform plan` (review diff) → `terraform apply` (with state locked in S3/DynamoDB) → store outputs → on changes, plan/apply again → `terraform destroy` to tear down. Secrets via Vault/env vars; environments via workspaces; existing resources via import; drift caught via plan.
