# Cloud Data Loss Prevention (DLP) API Revision

## Task 1: Clone the Repo and Enable APIs

### Clone the Repository
In Cloud Shell, run the following command to download the Cloud Data Loss Prevention Node.js Client repository:
```sh
git clone https://github.com/googleapis/synthtool
```

### Install Node.js Packages
Once you download the project code, change into the samples directory and install the required Node.js packages:
```sh
cd synthtool/tests/fixtures/nodejs-dlp/samples/ && npm install
```
Note: Ignore any warning messages.

### Set the Correct Project
Make sure you're using the correct project by setting it with the following gcloud command:
```sh
export PROJECT_ID="<filled at="" in="" lab="" start="">"
gcloud config set project $PROJECT_ID
```

### Enable Required APIs
Enable the required APIs with the following gcloud command:
```sh
gcloud services enable dlp.googleapis.com cloudkms.googleapis.com --project $PROJECT_ID
```

## Task 2: Inspect Strings and Files

### Inspect a String
Provide the string option and a sample string with some potentially sensitive information:
```sh
node inspectString.js $PROJECT_ID "My email address is jenny@somedomain.com and you can call me at 555-867-5309" > inspected-string.txt
```
Check the output using the following command:
```sh
cat inspected-string.txt
```
The findings for the request above are:
```
Findings:
    Info type: PERSON_NAME
    Likelihood: POSSIBLE
    Info type: EMAIL_ADDRESS
    Likelihood: LIKELY
    Info type: PHONE_NUMBER
    Likelihood: VERY_LIKELY
```

### Inspect a File
Run the following command to review the sample accounts.txt file:
```sh
cat resources/accounts.txt
```
The file includes the following text:
```
My credit card number is 1234 5678 9012 3456, and my CVV is 789.
```
Use the inspectFile.js file to inspect the provided file for sensitive info types:
```sh
node inspectFile.js $PROJECT_ID resources/accounts.txt > inspected-file.txt
```
Check the output using the following command:
```sh
cat inspected-file.txt
```
The results:
```
Findings:
    Info type: CREDIT_CARD_NUMBER
    Likelihood: VERY_LIKELY
```

### Upload Output to Cloud Storage
Run the following commands to upload the responses on Cloud Storage for activity tracking validation:
```sh
gsutil cp inspected-string.txt gs://bucket_name_filled_after_lab_start
gsutil cp inspected-file.txt gs://bucket_name_filled_after_lab_start
```

## Task 3: De-identification

### De-identify with a Mask
Run the following command to use deidentifyWithMask.js to try de-identification with a mask:
```sh
node deidentifyWithMask.js $PROJECT_ID "My order number is F12312399. Email me at anthony@somedomain.com" > de-identify-output.txt
```
Check the output using the following command:
```sh
cat de-identify-output.txt
```
Example output:
```
My order number is F12312399. Email me at *****************************
```

### Upload Output to Cloud Storage
Run the following commands to upload the responses on Cloud Storage for activity tracking validation:
```sh
gsutil cp de-identify-output.txt gs://bucket_name_filled_after_lab_start
```

## Task 4: Redact Strings and Images

### Redact Text
Use redactText.js to redact text from a sample input:
```sh
node redactText.js $PROJECT_ID  "Please refund the purchase to my credit card 4012888888881881" CREDIT_CARD_NUMBER > redacted-string.txt
```
Check the output using the following command:
```sh
cat redacted-string.txt
```
Example output:
```
Please refund the purchase on my credit card [CREDIT_CARD_NUMBER]
```

### Redact an Image
To redact the phone number from the image, run the following command:
```sh
node redactImage.js $PROJECT_ID resources/test.png "" PHONE_NUMBER ./redacted-phone.png
```
To verify, open the samples/redacted-phone.png file using Cloud Shell Code Editor.

Try it again to redact the email address from the image:
```sh
node redactImage.js $PROJECT_ID resources/test.png "" EMAIL_ADDRESS ./redacted-email.png
```
To verify, open the samples/redacted-email.png file in the Cloud Shell Code Editor.

### Upload Output to Cloud Storage
Run the following commands to upload the responses on Cloud Storage for activity tracking validation:
```sh
gsutil cp redacted-string.txt gs://bucket_name_filled_after_lab_start
gsutil cp redacted-phone.png gs://bucket_name_filled_after_lab_start
gsutil cp redacted-email.png gs://bucket_name_filled_after_lab_start
```

---

Congratulations! The Cloud Data Loss Prevention (DLP) API is a powerful tool that provides access to a powerful sensitive data inspection, classification, and de-identification platform. You used the DLP API to inspect strings and files for multiple info types, and then redact data from a string and an image.