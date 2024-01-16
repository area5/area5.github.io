---
title:  "How to create MediaConvert job using AWS CLI"
tags: AWS MediaConvert
---

The MediaConvert CLI show many parameters to create a job:

```bash
> aws mediaconvert create-job help

CREATE-JOB()                                                      CREATE-JOB()

NAME
       create-job -

DESCRIPTION
       Create a new transcoding job. For information about jobs and job
       settings, see the User Guide at
       http://docs.aws.amazon.com/mediaconvert/latest/ug/what-is.html

       See also: AWS API Documentation

SYNOPSIS

            create-job
          [--acceleration-settings <value>]
          [--billing-tags-source <value>]
          [--client-request-token <value>]
          [--hop-destinations <value>]
          [--job-template <value>]
          [--priority <value>]
          [--queue <value>]
          --role <value>
          --settings <value>
          [--simulate-reserved-queue <value>]
          [--status-update-interval <value>]
          [--tags <value>]
          [--user-metadata <value>]
          [--cli-input-json | --cli-input-yaml]
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
          [--cli-binary-format <value>]
          [--no-cli-pager]
          [--cli-auto-prompt]
          [--no-cli-auto-prompt]
```

All parameters can be passed into the CLI using 1 JSON document. Let's create the following file
`job.json`:

```json
{
  "Settings": {
    "Inputs": [
      {
        "FileInput": "s3://my-input-bucket/my-input-file.mp4"
      }
    ],
    "OutputGroups": [
      {
        "Name": "File Group",
        "OutputGroupSettings": {
          "Type": "FILE_GROUP_SETTINGS",
          "FileGroupSettings": {
            "Destination": "s3://my-output-bucket/path/"
          }
        },
        "Outputs": [
          {
            "VideoDescription": {
              "CodecSettings": {
                "Codec": "H_264",
                "H264Settings": {
                  "RateControlMode": "QVBR",
                  "MaxBitrate": 200000
                }
              }
            },
            "ContainerSettings": {
              "Container": "MP4"
            }
          }
        ]
      }
    ]
  },
  "Role": "arn:aws:iam::511591261676:role/service-role/MediaConvert_Default_Role"
}
```

Then we can use the following command to create job:

```bash
aws mediaconvert create-job --cli-input-json file://valid_min_job.json --region us-west-2
```

# Analysis

The MediaConvert CLI options are divided into 2 groups: job parameters, and AWS CLI options. The
difference is that job parameters are specific to MediaConvert jobs, and AWS CLI options apply to all
AWS CLI. The `job.json` above can only contain MediaConvert specific settings.

MediaConvert specific options are first part of the list from `--acceleration-settings` to
`--user-metadata`.

The AWS CLI options are the latter part of the list from `--cli-input-json` to
`--no-cli-auto-prompt`.

For example, `--role` is a MediaConvert job settings; so, it can be specified inside `job.json`.
While, `--region` is a AWS CLI option; therefore, it cannot be place inside the job JSON.
