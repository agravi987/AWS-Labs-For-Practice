# Lab 24 - AWS Backup: Backup and Restore an EBS Volume

> Updated 20 August 2026

![AWS Backup](https://img.shields.io/badge/AWS%20Backup-EBS%20restore-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner%2FIntermediate-F2C94C?style=flat-square)
![Time](https://img.shields.io/badge/Time-30--45%20minutes-2F80ED?style=flat-square)

## What you will learn

You will create an AWS Backup plan, assign an EBS volume to it, run an on-demand backup, restore the backup as a new volume, and clean up only the resources created in this lab.

This lab deliberately uses one resource. The important skill is the backup and restore workflow. RDS, S3, and DynamoDB are optional extensions because each service has different prerequisites and restore behavior.

## Before you start

- Choose one AWS Region and use it for every step.
- Confirm that your account can use AWS Backup, EC2, IAM, and EBS.
- Make sure you are not working in a production account.
- Set an AWS Billing alarm if this is a paid account.

You do not need an EC2 key pair or an RDS database for the core lab. An EC2 instance is optional; an unattached EBS volume is enough.

## Cost and safety

EBS volumes, backup recovery points, and restored volumes can incur charges. The exact price depends on Region, size, storage class, and how long the resources remain.

Use these tags on every resource created here:

| Key | Value |
|---|---|
| `Lab` | `24` |
| `Name` | `ravi-backup-lab` |

At the end, delete only resources with these tags. Do not delete every recovery point in the Default vault: it may contain backups from another lab or workload.

## How AWS Backup fits together

```text
EBS volume
    |
    v
Backup plan -- schedule + retention rule
    |
    v
Backup job ---> recovery point in a backup vault
                         |
                         v
                    restore job
                         |
                         v
                  new EBS volume
```

- A **backup plan** contains one or more rules.
- A rule defines the schedule, backup vault, and lifecycle.
- A **backup job** creates a recovery point.
- A **restore job** creates a new resource from a recovery point.
- A scheduled rule does not normally run immediately. This lab uses an on-demand backup so you can finish without waiting for the clock.

## Step 1 - Create or choose an EBS volume

Use an existing non-production EBS volume, or create a small one:

1. Open the **EC2 console** in your chosen Region.
2. Open **Elastic Block Store > Volumes**.
3. Choose **Create volume**.
4. Select `gp3`, set the size to `1 GiB` or the smallest allowed size, and choose an Availability Zone.
5. Add the tags `Lab=24` and `Name=ravi-backup-lab`.
6. Choose **Create volume**.
7. Record the volume ID and Availability Zone.

The volume can remain unattached. Wait until its state is **Available** before continuing.

## Step 2 - Check AWS Backup settings

1. Open the **AWS Backup console** in the same Region.
2. Open **Settings**.
3. Review **Service opt-in** and confirm that **Amazon Elastic Block Store (Amazon EBS)** is enabled if it is listed.
4. Open **Protected resources** and confirm that the EBS resource is discoverable.

Discovery is not protection. A resource is protected only after it is assigned to a backup plan and a backup job completes.

If you cannot open AWS Backup or see EBS resources, resolve the IAM or Region issue before creating a plan.

## Step 3 - Create the backup plan

1. In AWS Backup, open **Backup plans**.
2. Choose **Create backup plan**.
3. Choose **Build a new plan**. Console labels may change, so do not depend on a template name.
4. Set **Backup plan name** to `ravi-backup-plan`.
5. Add a rule with:
   - **Rule name**: `daily-ebs-backup`
   - **Backup vault**: `Default`
   - **Schedule**: daily at a convenient time
   - **Retention**: `7` days
6. Create the plan.

Use the console schedule picker when possible. If it asks for a cron expression, confirm the displayed timezone and whether the field expects the expression with or without the `cron(...)` wrapper. This schedule is for future runs; it is not the validation backup for this lab.

## Step 4 - Assign the tagged volume

1. Open `ravi-backup-plan`.
2. Open the **Resource assignments** or **Resources** tab. The label may vary in the current console.
3. Choose **Assign resources**.
4. Set **Assignment name** to `ravi-backup-assignment`.
5. Select **EBS** as the resource type.
6. Choose the tag-based resource selection and enter key `Lab` with value `24`.
7. Select the AWS Backup service role offered by the console. If none exists, choose the option to create the default AWS Backup service role.
8. Submit the assignment.

If your console only offers resource IDs, select the volume ID recorded in Step 1. Using a tag or exact ID avoids accidentally selecting an unrelated volume.

## Step 5 - Run an on-demand backup

The daily rule may not run during this lab, so create a backup now:

1. Open **Protected resources**.
2. Choose **Create an on-demand backup**.
3. Select **EBS** as the resource type.
4. Select the volume created in Step 1.
5. Select the **Default** backup vault.
6. Set the retention period to `7` days.
7. Choose the AWS Backup service role.
8. Choose **Create on-demand backup**.
9. Open **Backup jobs** and wait for the job to reach **Completed**.

Do not continue to restore until the job is completed. A running job does not yet provide a usable recovery point.

## Step 6 - Confirm the recovery point

1. Open **Backup vaults > Default**.
2. Find the recovery point created for the tagged volume.
3. Check its resource ID, creation time, status, and expiry date.

The recovery point is evidence that the backup exists. The original EBS volume being present is not evidence that it was backed up.

## Step 7 - Restore the backup

1. Select the recovery point created in Step 6.
2. Choose **Restore**.
3. Leave the restore set to create a new EBS volume.
4. Select the same Availability Zone recorded in Step 1.
5. Keep the original size and encryption settings unless you have a reason to change them.
6. Add `Lab=24` and `Name=ravi-backup-restored` if the restore form allows tags.
7. Submit the restore job.
8. Open **Restore jobs** and wait for **Completed**.
9. Open **EC2 > Volumes** and find the new volume.

The restore is non-destructive: the original volume is not overwritten. The restored volume normally starts unattached. Verify its ID, size, Availability Zone, encryption, tags, and `Available` state.

## Validation checklist

- [ ] The original volume has tags `Lab=24` and `Name=ravi-backup-lab`.
- [ ] `ravi-backup-plan` exists with a 7-day retention rule.
- [ ] `ravi-backup-assignment` targets the tagged EBS volume.
- [ ] The on-demand backup job is **Completed**.
- [ ] The matching recovery point exists in the Default vault.
- [ ] The restore job is **Completed**.
- [ ] A second EBS volume exists and is not attached to the original resource.
- [ ] You can identify both volumes by their IDs and tags.

## Cleanup

Clean up in this order:

1. In **EC2 > Volumes**, delete the restored volume tagged `Name=ravi-backup-restored`.
2. Delete the original volume tagged `Name=ravi-backup-lab`.
3. In AWS Backup, delete the `ravi-backup-assignment` assignment.
4. Delete `ravi-backup-plan`.
5. In **Backup vaults > Default**, delete only the recovery point created by this lab.
6. If you created a dedicated IAM role, delete it only after confirming no other backup plan uses it.
7. Recheck EC2 volumes, Backup jobs, Restore jobs, and the recovery point list.

Do not delete the Default backup vault. Do not delete recovery points that belong to another resource or lab. A recovery point can be protected by retention or vault-lock settings; follow the AWS console message rather than trying to bypass it.

## Optional extensions

Complete these only after the core EBS lab:

- **RDS**: assign an available RDS database, run an on-demand backup, and restore it as a new database instance. This takes longer and costs more.
- **S3**: review current AWS Backup for S3 prerequisites, including bucket versioning, service opt-in, IAM permissions, and the supported backup mode.
- **DynamoDB**: review the current AWS Backup requirements and restore workflow. A restore creates a new table and does not overwrite the source table.
- **Cross-Region copy**: use a dedicated vault in the destination Region and test copy and cleanup separately.
- **Compliance reporting**: configure AWS Backup Audit Manager and a framework before creating compliance reports. A report is not automatically generated just because a backup plan exists.

## Troubleshooting

### The volume is not listed

Confirm the console is in the same Region, the volume is `Available`, and EBS is enabled under AWS Backup service settings. If tag selection is used, check the tag key and value exactly.

### The backup job fails

Open the job details and read the status message. Common causes include an incorrect Region, missing AWS Backup service-role permissions, an unsupported resource state, or a vault/KMS permission issue.

### The recovery point is not visible

Wait for the backup job to reach **Completed**, then refresh **Backup vaults > Default**. Filter by the resource ID instead of relying on the newest item.

### The restore job fails

Check the restore job details, destination Availability Zone, EBS quotas, and KMS permissions. Restore to a new volume and do not overwrite the source volume.

## Official documentation

- [What is AWS Backup?](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [Creating a backup plan](https://docs.aws.amazon.com/aws-backup/latest/devguide/creating-a-backup-plan.html)
- [Assigning resources to a backup plan](https://docs.aws.amazon.com/aws-backup/latest/devguide/assigning-resources.html)
- [Restoring a backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/restoring-a-backup.html)
- [Deleting backups](https://docs.aws.amazon.com/aws-backup/latest/devguide/deleting-backups.html)

## What you learned

AWS Backup is a coordination layer: a plan defines when backups should run, a vault stores recovery points, and a restore job creates a new resource. The reliable habit is simple: create a backup, wait for **Completed**, restore it, verify the result, and clean up deliberately.

**Next:** [Lab 25 - Capstone: Full Stack on AWS](../25%20-%20Capstone%20-%20Full%20Stack%20on%20AWS/README.md)
