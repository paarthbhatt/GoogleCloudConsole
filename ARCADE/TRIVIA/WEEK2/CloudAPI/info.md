# Cloud Storage Lab Revision

## Task 1: Create Cloud Storage Buckets

1. **Access the Cloud Storage APIs Explorer tool:**
   - Navigation: **APIs & Services > Library**
   - Search for **Cloud Storage**
   - Select **Google Cloud Storage JSON API**
   - Ensure the API is enabled, click **Enable** if not.

2. **Open the Buckets: insert reference:**
   - This opens the APIs Explorer page.
   - Use the insert method from the Cloud Storage JSON API.

3. **Fill in the details for your storage bucket:**
   - **Project ID:** PROJECT ID (ensure no trailing spaces)
   - **Request Body:** Provide a unique name for your Cloud Storage bucket.
   - Ensure **Google OAuth 2.0** and **API key** checkboxes are selected.

4. **Execute the API request:**
   - Click **Execute**, select your student account, and allow access.
   - Expected response:
   ```json
   {
     "kind": "storage#bucket",
     "id": "qwiklabs-bucket01",
     "selfLink": "https://www.googleapis.com/storage/v1/b/qwiklabs-bucket01",
     "projectNumber": "250399850182",
     "name": "qwiklabs-bucket01",
     "timeCreated": "2019-10-18T13:59:08.300Z",
     "updated": "2019-10-18T13:59:08.300Z",
     "metageneration": "1",
     "iamConfiguration": {
       "bucketPolicyOnly": { "enabled": false },
       "uniformBucketLevelAccess": { "enabled": false }
     },
     "location": "US",
     "locationType": "multi-region",
     "storageClass": "STANDARD",
     "etag": "CAE="
   }
   ```

## Task 2: Make a Second Cloud Storage Bucket

1. **Create another Cloud Storage bucket:**
   - Use the same **insert method** with the Project ID.
   - Provide a unique name for the second bucket.

2. **Execute the API request:**
   - Click **Execute**.
   - Expected response:
   ```json
   {
     "kind": "storage#bucket",
     "id": "qwiklabs-bucket02",
     "selfLink": "https://www.googleapis.com/storage/v1/b/qwiklabs-bucket02",
     "projectNumber": "250399850182",
     "name": "qwiklabs-bucket02",
     "timeCreated": "2019-10-18T13:59:08.300Z",
     "updated": "2019-10-18T13:59:08.300Z",
     "metageneration": "1",
     "iamConfiguration": {
       "bucketPolicyOnly": { "enabled": false },
       "uniformBucketLevelAccess": { "enabled": false }
     },
     "location": "US",
     "locationType": "multi-region",
     "storageClass": "STANDARD",
     "etag": "CAE="
   }
   ```

## Task 3: Upload Files to Your Cloud Storage Bucket

1. **Save the following images to your computer:**
   - `demo-image1.png` (Dog)
   - `demo-image2.png` (Ada Lovelace)

2. **Upload files to the first bucket:**
   - Select the first bucket in the Cloud Storage browser.
   - Click **Upload files** and select `demo-image1.png` and `demo-image2.png`.

## Task 4: Copy Files Between Cloud Storage Buckets

1. **Use the Objects: copy method:**
   - **sourceBucket:** Name of the bucket containing the demo image files.
   - **sourceObject:** `demo-image1.png`
   - **destinationBucket:** Name of the second bucket.
   - **destinationObject:** `demo-image1-copy.png`

2. **Execute the API request:**
   - Click **Execute**.
   - Expected response:
   ```json
   {
     "kind": "storage#object",
     "id": "qwiklabs-bucket02/demo-image1-copy.png/1571408245199237",
     "selfLink": "https://www.googleapis.com/storage/v1/b/qwiklabs-bucket02/o/demo-image1-copy.png",
     "name": "demo-image1-copy.png",
     "bucket": "qwiklabs-bucket02",
     "generation": "1571408245199237",
     "metageneration": "1",
     "contentType": "image/png",
     "timeCreated": "2019-10-18T14:17:25.198Z",
     "updated": "2019-10-18T14:17:25.198Z",
     "storageClass": "STANDARD",
     "timeStorageClassUpdated": "2019-10-18T14:17:25.198Z",
     "size": "401951",
     "md5Hash": "LbpHpwhnApQKQx9IEXjTsQ==",
     "mediaLink": "https://www.googleapis.com/download/storage/v1/b/qwiklabs-bucket02/o/demo-image1-copy.png?generation=1571408245199237&alt=media",
     "owner": { "entity": "user-gcpstaging93416_student@qwiklabs.net" },
     "crc32c": "j5oPrg==",
     "etag": "CIWjgvL/peUCEAE="
   }
   ```

## Task 5: Delete Files from a Cloud Storage Bucket

1. **Use the Objects: delete method:**
   - **bucket:** Name of the bucket containing both demo image files.
   - **object:** `demo-image1.png`

2. **Execute the API request:**
   - Click **Execute**.
   - Expected output: `204 displayed on a green ribbon`

3. **Remove the second image:**
   - **object:** `demo-image2.png`
   - Execute the API request and expect output: `204 displayed on a green ribbon`

## Task 6: Delete Your Cloud Storage Bucket

1. **Use the Buckets: delete method:**
   - **bucket:** Name of your first bucket.

2. **Execute the API request:**
   - Click **Execute**.
   - Expected output: `204 displayed on a green ribbon`

## Task 7: Test Your Understanding

1. **Each bucket has a default storage class, which you can specify when you create your bucket.**
   - **True**

2. **Every bucket must have a unique name across the entire Cloud Storage namespace.**
   - **True**

3. **Cloud Storage offers four storage classes:**
   - **Nearline Storage**
   - **Regional Storage**
   - **Coldline Storage**
   - **Multi-Regional Storage**

Congratulations! You have completed the lab on creating, managing, and interacting with Cloud Storage buckets through the APIs Explorer.