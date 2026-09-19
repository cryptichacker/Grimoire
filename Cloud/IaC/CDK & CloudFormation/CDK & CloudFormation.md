---
tags: [cloud, iac, aws, cdk, cloudformation]
type: cheatsheet
source: compiled reference (AWS CDK & CloudFormation)
last-verified: 2026-09-16
---

# CDK & CloudFormation

## Up
- [[IaC]]

AWS's native IaC. **CloudFormation (CFN)** is the underlying engine — declarative JSON/YAML templates that AWS provisions as **stacks**. **AWS CDK** lets you write infrastructure in real languages (TypeScript/Python/Java/Go/C#) that **synthesizes to CloudFormation**. Cross-links to [[AWS]]; compare with [[Terraform]] (multi-cloud) and [[Pulumi]].

---

## CloudFormation (the engine)

A template describes resources; AWS creates/updates/deletes them as a **stack** (with rollback on failure).

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  BucketName: { Type: String }
Resources:
  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      VersioningConfiguration: { Status: Enabled }
Outputs:
  BucketArn: { Value: !GetAtt Bucket.Arn }
```

```bash
aws cloudformation deploy --template-file tpl.yaml --stack-name app \
  --parameter-overrides BucketName=my-bucket --capabilities CAPABILITY_IAM
aws cloudformation describe-stack-events --stack-name app
aws cloudformation delete-stack --stack-name app
```

- **Intrinsic functions:** `!Ref`, `!GetAtt`, `!Sub`, `!Join`, `!FindInMap`, `Fn::If`.
- **Change sets** = preview diffs before applying; **drift detection** flags manual changes; **nested stacks** + **StackSets** (multi-account/region).
- Verbose to hand-write — which is why CDK exists.

---

## AWS CDK (code → CloudFormation)

```typescript
import * as cdk from "aws-cdk-lib";
import { aws_s3 as s3, aws_lambda as lambda } from "aws-cdk-lib";

class AppStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);
    const bucket = new s3.Bucket(this, "Data", { versioned: true, encryption: s3.BucketEncryption.S3_MANAGED });
    const fn = new lambda.Function(this, "Fn", {
      runtime: lambda.Runtime.NODEJS_20_X, handler: "index.handler",
      code: lambda.Code.fromAsset("lambda"),
    });
    bucket.grantRead(fn);           // L2 constructs wire IAM for you
  }
}
const app = new cdk.App();
new AppStack(app, "AppStack");
```

```bash
cdk init app --language typescript
cdk bootstrap                      # one-time per account/region (deploy toolkit)
cdk synth                          # → CloudFormation template
cdk diff                           # preview changes vs deployed stack
cdk deploy                         # deploy
cdk destroy
```

### Construct levels
| Level | What |
|---|---|
| **L1 (Cfn*)** | Raw 1:1 CFN resources (`CfnBucket`) |
| **L2** | Sensible-default, higher-level resources (`s3.Bucket`) — most used; auto IAM/wiring |
| **L3 (patterns)** | Opinionated multi-resource patterns (e.g. `ApplicationLoadBalancedFargateService`) |

---

## CDK vs CloudFormation vs Terraform

| | CloudFormation | AWS CDK | [[Terraform]] |
|---|---|---|---|
| Form | YAML/JSON templates | Real code → CFN | HCL |
| Cloud | AWS only | AWS only (synth→CFN) | Multi-cloud |
| Abstraction | Low | High (L2/L3 constructs) | Modules |
| State | Managed by AWS (stacks) | Same (via CFN) | TF state |

CDK's `cdk synth` output *is* CloudFormation, so you get CFN's managed rollback/drift with code ergonomics. Pick CDK/CFN for AWS-only shops that want native integration; [[Terraform]]/[[Pulumi]] for multi-cloud.

---

## Tips
- **`cdk bootstrap`** once per account+region before first deploy (creates the CDK toolkit stack/bucket).
- Use **L2 constructs** and helpers like `grantRead`/`grant*` — they generate least-privilege IAM automatically.
- Always **`cdk diff`** (or CFN **change sets**) before deploy; enable **termination protection** on prod stacks.
- CFN rolls back a failed stack automatically — good, but a stuck `UPDATE_ROLLBACK_FAILED` needs manual attention.
- `cdk synth` to review the generated template and to hand off pure CFN to teams that require it.
