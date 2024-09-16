AWS RDS Disaster Recovery Steps:-

1. Create a new KMS key in Disaster Region
    name: dr-rds-prod-test

2. Copy Snapshot in DR Region
    name: dr-dbprod-19-09-2022

"You can restore to any point in time within your backup retention period. To see the earliest restorable time for each DB instance, choose Automated backups in the Amazon RDS console."
-- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html


env name: dbprod

check config:
aws rds describe-db-instances --db-instance-identifier dbprod --profile bayer --region eu-west-1 --no-verify-ssl


Point-in-time Recovery with CLI:

aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier mysourcedbinstance \
    --target-db-instance-identifier mytargetdbinstance \
    --restore-time 2017-10-14T23:45:00.000Z \
    --max-allocated-storage 1000

aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier mysourcedbinstance \
    --target-db-instance-identifier mytargetdbinstance \
    --restore-time 2017-10-14T23:45:00.000Z \
    --max-allocated-storage 1000


##########Restore older sit automated snapshot in DR region############
Env: sit
Snapshot Date: 2022-09-05-02-43
Snapshot Name: mdbsit-2022-09-05-02-43

Recovery Date: 19-09-2022

1. Copy to Snapshot to N. Virginia Region.
    New Snapshot Name: mdbsit-2022-09-05-02-43-copy
    KMS: dr-test

aws rds copy-db-snapshot \
    --source-db-snapshot-identifier mdbsit-2022-09-05-02-43 \
    --target-db-snapshot-identifier mdbsit-2022-09-05-02-43-copy \
    --kms-key-id dr-test \
    --source-region eu-west-1 \
    --region us-east-1 \
    --no-verify-ssl \
    --profile bayer


Backup Replication:
aws rds start-db-instance-automated-backups-replication \
--source-db-instance-arn arn:aws:rds:eu-west-1:098194264563:db:dbprod \
--backup-retention-period 2 \
--kms-key-id arn:aws:kms:us-east-1:098194264563:key/63e638ce-20e1-4208-a034-6d0ea57f67d6 \
--source-region eu-west-1 \
--region us-east-1 \
--profile bayer \
--no-verify-ssl


start-db-instance-automated-backups-replication
--source-db-instance-arn <value>
[--backup-retention-period <value>]
[--kms-key-id <value>]
[--pre-signed-url <value>]
[--source-region <value>]
[--cli-input-json <value>]
[--generate-cli-skeleton <value>]
[--debug]
[--endpoint-url <value>]
[--no-verify-ssl]
[--no-paginate]
[--output <value>]
[--query <value>]
[--profile <value>]
[--region <value>]
[--version <value>]
[--color <value>]
[--no-sign-request]
[--ca-bundle <value>]
[--cli-read-timeout <value>]
[--cli-connect-timeout <value>]
