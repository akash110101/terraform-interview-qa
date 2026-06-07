# Terraform Zero to Hero — Revision Checklist

A topic checklist covering everything in [Terraform-Interview-QA-Notes](./README.md). Tick each item once you can confidently explain it out loud.

## Day 1 — Fundamentals & Setup
- [ ] Define IaC and the problems it solves (vs manual config)
- [ ] Why Terraform over other IaC tools (multi-cloud, state, plan/apply, HCL)
- [ ] Declarative vs imperative + idempotency
- [ ] What HCL is
- [ ] Core terminology: provider, resource, module, variable, output, state, plan, apply, workspace, backend
- [ ] Install Terraform + AWS setup (CLI, IAM user, `aws configure`)
- [ ] Write a basic provider + resource config
- [ ] Core workflow: `init` → `plan` → `apply` → `destroy`
- [ ] Desired vs current state; why the state file matters
- [ ] Scenario: re-apply with no changes (idempotency)

## Day 2 — Providers, Variables, Outputs, Functions
- [ ] What a provider is technically (plugin/API)
- [ ] Three ways to configure providers (root, child, `required_providers`)
- [ ] Multiple providers (AWS + Azure) in one project
- [ ] Multi-region with `alias`
- [ ] `required_providers` (source + version pinning)
- [ ] Version constraint operators (`~>`, `>=`, `=`, ranges)
- [ ] Input variables (`variable` block, `var.x`)
- [ ] Output variables (`output`, `module.x.y`)
- [ ] Variable types (string/number/bool + collections)
- [ ] `.tfvars` files and `-var-file`
- [ ] Variable precedence order
- [ ] Conditional expressions (ternary)
- [ ] Conditional resource creation with `count` (feature flag)
- [ ] Conditional value assignment
- [ ] Built-in functions (`concat`, `element`, `length`, `lookup`, `join`)
- [ ] Debug & format: `fmt`, `validate`, `TF_LOG`
- [ ] Day-2 project: VPC + subnets + IGW + route table + SG + EC2 + ALB

## Day 3 — Modules, Locals, Data Sources, Registry
- [ ] What a module is (root vs child)
- [ ] Benefits of modules
- [ ] Calling a module + passing inputs (`source`)
- [ ] What `source` can point to (local, registry, Git, HTTP, S3)
- [ ] Local values (`locals`, `local.x`) and when to use
- [ ] Data sources (read-only lookups)
- [ ] Variable vs local vs data source
- [ ] Terraform Registry

## Day 4 — Collaboration, Backends, State Locking
- [ ] Git/version control with Terraform
- [ ] Handling sensitive data + `.gitignore`
- [ ] What a backend is (local vs remote)
- [ ] Why state in VCS is bad
- [ ] Configure S3 remote backend
- [ ] State locking (why it matters)
- [ ] DynamoDB locking setup (`LockID` hash key)
- [ ] Scenario: concurrent applies with locking
- [ ] Bootstrapping backend resources (S3 + DynamoDB)

## Day 5 — Provisioners
- [ ] What a provisioner is (create/destroy time)
- [ ] The three provisioners: `file`, `remote-exec`, `local-exec`
- [ ] Write each of the three
- [ ] `connection` block
- [ ] Handling failures (`on_failure`)
- [ ] Best practice: provisioners are a last resort
- [ ] `null_resource` + `triggers`

## Day 6 — Workspaces
- [ ] What a workspace is (isolated state per env)
- [ ] Workspace commands (`new`/`list`/`select`/`show`)
- [ ] Per-workspace behavior via `terraform.workspace` + `lookup`/maps
- [ ] Benefits and limits (vs separate directories)

## Day 7 — Security & Vault
- [ ] Ways to secure sensitive data
- [ ] What `sensitive = true` does and does **not** do
- [ ] What HashiCorp Vault is
- [ ] Terraform ↔ Vault integration flow (AppRole, policy, Role/Secret ID)
- [ ] Vault provider + secret data source pattern
- [ ] Why AppRole over a static token
- [ ] Gotcha: Vault secrets still land in state

## Day 8 — Migration & Drift
- [ ] What configuration drift is
- [ ] Detecting drift (`plan`, `-refresh-only`)
- [ ] Remediating drift (re-apply / update code / refresh)
- [ ] Migrating existing resources (`terraform import`, `import` block)

## Cross-Cutting / Rapid-Fire
- [ ] `plan` vs `apply`
- [ ] What `init` does under the hood
- [ ] `fmt` vs `validate`
- [ ] `count` vs `for_each`
- [ ] `depends_on`
- [ ] Referencing attributes (`type.name.attr`)
- [ ] `self` in connection/provisioner
- [ ] Structuring a multi-environment project (design)
- [ ] End-to-end workflow narrative
