### 1. what are pros and cons of terraform over cloudformation?
**Ans:**
| Feature              | **Terraform**                                                                                              | **CloudFormation**                                         |
| -------------------- | ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Provider Support** | Multi-cloud (AWS, Azure, GCP, Kubernetes, etc.)                                                            | AWS-only                                                   |
| **Language**         | HCL (HashiCorp Configuration Language) – simple & modular                                                  | JSON / YAML (verbose, harder to maintain at scale)         |
| **State Management** | External state file (local or remote in S3, Terraform Cloud, etc.) → gives flexibility but must be secured | State managed automatically by AWS                         |
| **Modularity**       | Modules (highly reusable, community-driven registry)                                                       | Nested stacks (less flexible, limited community modules)   |
| **Speed**            | Generally faster plan/apply cycle                                                                          | Slower deployments (waits for each resource step by step)  |
| **Ecosystem**        | Large open-source community, many providers, flexible integrations                                         | Tight AWS integration, strong support for new AWS features |
| **Drift Detection**  | `terraform plan` shows differences before applying                                                         | Drift detection supported, but not as intuitive            |
| **Learning Curve**   | Easier due to simple HCL syntax                                                                            | YAML/JSON can be harder to maintain/debug                  |
| **Cost**             | Free (open source, paid Terraform Cloud optional)                                                          | Free (pay only for AWS resources)                          |
