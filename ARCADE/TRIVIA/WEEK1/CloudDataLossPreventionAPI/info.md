# Google Cloud Data Loss Prevention (DLP) API - Quickstart Guide

## Task 1: Inspect a string for sensitive information

This section shows you how to ask the service to scan sample text using the `projects.content.inspect` REST method. The JSON file you create contains an `InspectConfig` and a `ContentItem` object.

### Step 1: Create a JSON request file

Using your preferred editor (nano, vim, etc.) or Cloud Shell, create a JSON request file with the following text, and save it as `inspect-request.json`:

```json name=inspect-request.json
{
  "item": {
    "value": "My phone number is (206) 555-0123."
  },
  "inspectConfig": {
    "infoTypes": [
      {
        "name": "PHONE_NUMBER"
      },
      {
        "name": "US_TOLLFREE_PHONE_NUMBER"
      }
    ],
    "minLikelihood": "POSSIBLE",
    "limits": {
      "maxFindingsPerItem": 0
    },
    "includeQuote": true
  }
}
```

### Step 2: Obtain an authorization token

Run the following command to obtain an authorization token using your account:

```sh
gcloud auth print-access-token
```

A huge string is returned. You need this token for the next step.

> **Note**: If you receive an error that no service account is being used, wait a few minutes and run the command again.

### Step 3: Make a content:inspect request

Use `curl` to make a `content:inspect` request, replacing `ACCESS_TOKEN` with the string that was returned in the previous step:

```sh
curl -s \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  https://dlp.googleapis.com/v2/projects/$PROJECT_ID/content:inspect \
  -d @inspect-request.json -o inspect-output.txt
```

> **Note**: To pass a filename to `curl` you use the `-d` option (for "data") and precede the filename with an `@` sign. This file should be in the same directory in which you execute the `curl` command.

### Step 4: Check the output

Run the following command to check the output:

```sh
cat inspect-output.txt
```

You should see a response similar to the following:

```json name=inspect-output.txt
{
  "result": {
    "findings": [
      {
        "quote": "(206) 555-0123",
        "infoType": {
          "name": "PHONE_NUMBER"
        },
        "likelihood": "LIKELY",
        "location": {
          "byteRange": {
            "start": "19",
            "end": "33"
          },
          "codepointRange": {
            "start": "19",
            "end": "33"
          }
        },
        "createTime": "2018-07-03T02:20:26.043Z"
      }
    ]
  }
}
```

### Step 5: Upload output to Cloud Storage

Run the following command to upload the `curl` response to Cloud Storage for activity tracking validation:

```sh
gsutil cp inspect-output.txt gs://bucket_name_filled_after_lab_start
```

## Task 2: Redacting sensitive data from text content

The DLP API can automatically redact sensitive data from text files instead of giving you a list of findings.

### Step 1: Create a new JSON request file

Create a new JSON file (called `new-inspect-file.json`) that includes the following:

```json name=new-inspect-file.json
{
  "item": {
     "value": "My email is test@gmail.com"
   },
   "deidentifyConfig": {
     "infoTypeTransformations": {
          "transformations": [
            {
              "primitiveTransformation": {
                "replaceWithInfoTypeConfig": {}
              }
            }
          ]
        }
    },
    "inspectConfig": {
      "infoTypes": [
        {
          "name": "EMAIL_ADDRESS"
        }
      ]
    }
}
```

### Step 2: Make a content:deidentify request

Use `curl` to make a `content:deidentify` request (ACCESS_TOKEN has been replaced with a command to print the access token):

```sh
curl -s \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json" \
  https://dlp.googleapis.com/v2/projects/$PROJECT_ID/content:deidentify \
  -d @new-inspect-file.json -o redact-output.txt
```

### Step 3: Check the output

Run the following command to check the output:

```sh
cat redact-output.txt
```

You should see a response similar to the following:

```json name=redact-output.txt
{
  "item": {
    "value": "My email is [EMAIL_ADDRESS]"
  },
  "overview": {
    "transformedBytes": "14",
    "transformationSummaries": [
      {
        "infoType": {
          "name": "EMAIL_ADDRESS"
        },
        "transformation": {
          "replaceWithInfoTypeConfig": {}
        },
        "results": [
          {
            "count": "1",
            "code": "SUCCESS"
          }
        ],
        "transformedBytes": "14"
      }
    ]
  }
}
```

### Step 4: Upload output to Cloud Storage

Run the following command to upload the `curl` response to Cloud Storage for activity tracking validation:

```sh
gsutil cp redact-output.txt gs://bucket_name_filled_after_lab_start
```

## Congratulations!

You used the Cloud Data Loss Prevention (DLP) API to inspect for, and then redact sensitive data from text content.