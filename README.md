# AWS RDS Point-In-Time Recovery (PITR) and Disaster Recovery Guide

This repository provides comprehensive instructions and AWS CLI commands to perform Point-In-Time Recovery (PITR) and establish Disaster Recovery (DR) strategies for Amazon Relational Database Service (RDS). The guide covers restoring RDS instances, copying snapshots across regions, and setting up automated backup replication to ensure data availability and resilience.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Point-In-Time Recovery (PITR) with AWS Backup RDS](#point-in-time-recovery-pitr-with-aws-backup-rds)
  - [Restore DB Instance to a Specific Point in Time](#restore-db-instance-to-a-specific-point-in-time)
  - [Example Command](#example-command)
- [AWS RDS Disaster Recovery Steps](#aws-rds-disaster-recovery-steps)
  - [1. Create a New KMS Key in Disaster Recovery Region](#1-create-a-new-kms-key-in-disaster-recovery-region)
  - [2. Copy Snapshot to DR Region](#2-copy-snapshot-to-dr-region)
  - [3. Backup Replication](#3-backup-replication)
- [Environment Configuration](#environment-configuration)
- [Additional Resources](#additional-resources)

## Prerequisites

Before proceeding, ensure you have the following:

- **AWS CLI Installed:** Ensure the AWS Command Line Interface is installed and configured on your machine. [Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- **AWS Credentials:** Properly configured AWS credentials with necessary permissions to perform RDS operations.
- **Knowledge of AWS Regions:** Understand the regions you are operating in, especially your primary and disaster recovery regions.
- **KMS Permissions:** Permissions to create and use AWS Key Management Service (KMS) keys.

## Point-In-Time Recovery (PITR) with AWS Backup RDS

Point-In-Time Recovery allows you to restore your RDS database instance to any specific time within your backup retention period.

### Restore DB Instance to a Specific Point in Time

The `aws rds restore-db-instance-to-point-in-time` command is used to restore an RDS DB instance to a specific point in time.

#### Command Structure

```bash
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier <source-db-id> \
    --target-db-instance-identifier <target-db-id> \
    --restore-time <YYYY-MM-DDTHH:MM:SSZ> \
    [additional-options]
```

#### Available Options

- `--source-db-instance-identifier`: (Required) The identifier of the source DB instance.
- `--target-db-instance-identifier`: (Required) The identifier for the new DB instance.
- `--restore-time`: The specific time to restore to.
- `--use-latest-restorable-time`: Use the latest available restorable time.
- `--db-instance-class`: The compute and memory capacity of the DB instance.
- `--port`: The port number on which the DB instance accepts connections.
- `--availability-zone`: The Availability Zone for the DB instance.
- `--db-subnet-group-name`: A DB subnet group to associate with the instance.
- `--multi-az`: Specifies if the DB instance is a Multi-AZ deployment.
- `--publicly-accessible`: Specifies the accessibility of the DB instance.
- `--auto-minor-version-upgrade`: Indicates if minor engine upgrades are applied automatically.
- `--license-model`: License model information for the DB instance.
- `--db-name`: The name of the database to create.
- `--engine`: The database engine to use.
- `--iops`: The amount of Provisioned IOPS.
- `--option-group-name`: The option group name.
- `--copy-tags-to-snapshot`: Copies all tags from the DB instance to snapshots.
- `--tags`: The tags to assign to the restored DB instance.
- `--storage-type`: The storage type to be associated with the DB instance.
- `--tde-credential-arn` and `--tde-credential-password`: Credentials for Transparent Data Encryption.
- `--vpc-security-group-ids`: VPC security groups to associate.
- `--domain` and `--domain-iam-role-name`: Active Directory domain and IAM role.
- `--enable-iam-database-authentication`: Enables IAM DB authentication.
- `--enable-cloudwatch-logs-exports`: Enables exporting of logs to CloudWatch.
- `--processor-features`: Specifies processor features.
- `--use-default-processor-features`: Uses default processor features.
- `--db-parameter-group-name`: The DB parameter group to associate.
- `--deletion-protection`: Enables deletion protection.
- `--source-dbi-resource-id`: The resource ID of the source DB instance.
- `--cli-input-json` or `--cli-input-yaml`: Provide input parameters as JSON or YAML.
- `--generate-cli-skeleton`: Generates a CLI skeleton.
- `--cli-auto-prompt`: Enables CLI auto-prompt.

### Example Command

```bash
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier dbprod \
    --target-db-instance-identifier dbprod-dr-test \
    --restore-time 2022-09-20T23:45:00Z \
    --no-publicly-accessible
```

**Explanation:**

- **Source DB Instance:** `dbprod`
- **Target DB Instance:** `dbprod-dr-test`
- **Restore Time:** September 20, 2022, at 23:45 UTC
- **Accessibility:** The restored DB instance will not be publicly accessible.

## AWS RDS Disaster Recovery Steps

Establishing a robust disaster recovery strategy ensures minimal downtime and data loss in case of unexpected failures.

### 1. Create a New KMS Key in Disaster Recovery Region

Before copying snapshots or setting up replication, create a new KMS key in your disaster recovery (DR) region to encrypt your backups.

```bash
aws kms create-key --description "dr-rds-prod-test" --region <DR_REGION>
```

- **Key Name:** `dr-rds-prod-test`
- **Region:** Replace `<DR_REGION>` with your disaster recovery region (e.g., `us-east-1`).

### 2. Copy Snapshot to DR Region

Copying snapshots to a DR region ensures that you have backups available even if the primary region experiences an outage.

#### Snapshot Details

- **Environment Name:** `dbprod`
- **Snapshot Date:** `2022-09-19`
- **Snapshot Name:** `dr-dbprod-19-09-2022`

#### Example Command

```bash
aws rds copy-db-snapshot \
    --source-db-snapshot-identifier dbprod-snapshot-2022-09-19 \
    --target-db-snapshot-identifier dr-dbprod-19-09-2022 \
    --kms-key-id arn:aws:kms:<DR_REGION>:098194264563:key/63e638ce-20e1-4208-a034-6d0ea57f67d6 \
    --source-region eu-west-1 \
    --region us-east-1 \
    --no-verify-ssl \
    --profile bayer
```

**Parameters:**

- **Source Snapshot Identifier:** `dbprod-snapshot-2022-09-19`
- **Target Snapshot Identifier:** `dr-dbprod-19-09-2022`
- **KMS Key ID:** Replace with your KMS key ARN in the DR region.
- **Source Region:** `eu-west-1`
- **Destination Region:** `us-east-1`
- **Profile:** `bayer` (AWS CLI profile)
- **SSL Verification:** Disabled with `--no-verify-ssl`

### 3. Backup Replication

Automate the replication of automated backups to the DR region to ensure up-to-date backups are always available.

#### Example Command

```bash
aws rds start-db-instance-automated-backups-replication \
    --source-db-instance-arn arn:aws:rds:eu-west-1:098194264563:db:dbprod \
    --backup-retention-period 2 \
    --kms-key-id arn:aws:kms:us-east-1:098194264563:key/63e638ce-20e1-4208-a034-6d0ea57f67d6 \
    --source-region eu-west-1 \
    --region us-east-1 \
    --profile bayer \
    --no-verify-ssl
```

**Parameters:**

- **Source DB Instance ARN:** `arn:aws:rds:eu-west-1:098194264563:db:dbprod`
- **Backup Retention Period:** `2` days
- **KMS Key ID:** Replace with your KMS key ARN in the DR region.
- **Source Region:** `eu-west-1`
- **Destination Region:** `us-east-1`
- **Profile:** `bayer`
- **SSL Verification:** Disabled with `--no-verify-ssl`

#### Command Options

- `--source-db-instance-arn`: The ARN of the source DB instance.
- `--backup-retention-period`: Number of days to retain backups.
- `--kms-key-id`: The KMS key ID for encryption.
- `--source-region`: The region where the source DB instance resides.
- `--region`: The target region for backup replication.
- `--profile`: AWS CLI profile to use.
- `--no-verify-ssl`: Disables SSL certificate verification.

## Environment Configuration

Ensure your environment is correctly set up to perform the necessary AWS operations.

### Check DB Instance Configuration

Use the following command to verify the configuration of your DB instance:

```bash
aws rds describe-db-instances \
    --db-instance-identifier dbprod \
    --profile bayer \
    --region eu-west-1 \
    --no-verify-ssl
```

**Parameters:**

- **DB Instance Identifier:** `dbprod`
- **Profile:** `bayer`
- **Region:** `eu-west-1`
- **SSL Verification:** Disabled with `--no-verify-ssl`

### Point-In-Time Recovery with CLI

Restore a DB instance to a specific point in time using the AWS CLI.

#### Example Command

```bash
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier mysourcedbinstance \
    --target-db-instance-identifier mytargetdbinstance \
    --restore-time 2017-10-14T23:45:00.000Z \
    --max-allocated-storage 1000
```

**Parameters:**

- **Source DB Instance Identifier:** `mysourcedbinstance`
- **Target DB Instance Identifier:** `mytargetdbinstance`
- **Restore Time:** October 14, 2017, at 23:45 UTC
- **Max Allocated Storage:** `1000` GB

**Note:** The same command is listed twice in the provided files. Ensure you run it as needed without duplication.

### Restore Older SIT Automated Snapshot in DR Region

For environments like SIT (System Integration Testing), you may need to restore older snapshots.

#### Snapshot Details

- **Environment:** `sit`
- **Snapshot Date:** `2022-09-05-02-43`
- **Snapshot Name:** `mdbsit-2022-09-05-02-43`
- **Recovery Date:** `2022-09-19`

#### Steps

1. **Copy Snapshot to N. Virginia Region**

    ```bash
    aws rds copy-db-snapshot \
        --source-db-snapshot-identifier mdbsit-2022-09-05-02-43 \
        --target-db-snapshot-identifier mdbsit-2022-09-05-02-43-copy \
        --kms-key-id arn:aws:kms:us-east-1:098194264563:key/63e638ce-20e1-4208-a034-6d0ea57f67d6 \
        --source-region eu-west-1 \
        --region us-east-1 \
        --no-verify-ssl \
        --profile bayer
    ```

    **Parameters:**

    - **Source Snapshot Identifier:** `mdbsit-2022-09-05-02-43`
    - **Target Snapshot Identifier:** `mdbsit-2022-09-05-02-43-copy`
    - **KMS Key ID:** Replace with your KMS key ARN in `us-east-1`.
    - **Source Region:** `eu-west-1`
    - **Destination Region:** `us-east-1`
    - **Profile:** `bayer`
    - **SSL Verification:** Disabled with `--no-verify-ssl`

## Additional Resources

- **AWS RDS Point-In-Time Recovery Documentation:** [AWS PITR](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
- **AWS CLI RDS Commands Reference:** [AWS CLI RDS](https://docs.aws.amazon.com/cli/latest/reference/rds/index.html)
- **AWS KMS Documentation:** [AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
- **Disaster Recovery Best Practices:** [AWS DR Strategies](https://aws.amazon.com/architecture/disaster-recovery/)

---

For any issues or questions, please open an issue in this repository or contact the maintainer.
