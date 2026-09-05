# Store, Process, and Manage Data on Google Cloud - Command Line: Challenge Lab

Google Cloud CLI solution for the ARC102 Challenge Lab — creating a Cloud Storage bucket, Pub/Sub topic, and Cloud Function for image thumbnail generation.

> Replace the placeholders (e.g. `<BUCKET_NAME>`, `<TOPIC_NAME>`, `<FUNCTION_NAME>`, `<REGION>`) with the values shown in your own lab instance before running.

## Author

**Aayush Shrestha (Shinux)** — Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

---

## Setup — Set environment variables

Open **Cloud Shell** and fill in the values given by your lab:

```bash
export REGION=<REGION>
export ZONE=<ZONE>
export BUCKET_NAME=<BUCKET_NAME>
export TOPIC_NAME=<TOPIC_NAME>
export FUNCTION_NAME=<FUNCTION_NAME>

gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

export PROJECT_ID=$(gcloud config get-value project)
```

Enable the required APIs:

```bash
gcloud services enable \
  artifactregistry.googleapis.com \
  cloudfunctions.googleapis.com \
  cloudbuild.googleapis.com \
  eventarc.googleapis.com \
  run.googleapis.com \
  logging.googleapis.com \
  pubsub.googleapis.com
```

---

## Task 1. Create a bucket

```bash
gsutil mb -l $REGION gs://$BUCKET_NAME
```

---

## Task 2. Create a Pub/Sub topic

```bash
gcloud pubsub topics create $TOPIC_NAME
```

---

## Task 3. Create the thumbnail Cloud Function

### Grant the Cloud Storage service account Pub/Sub publisher access

```bash
PROJECT_NUMBER=$(gcloud projects list \
  --filter="project_id:$PROJECT_ID" \
  --format='value(project_number)')

SERVICE_ACCOUNT=$(gsutil kms serviceaccount -p $PROJECT_NUMBER)

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT" \
  --role="roles/pubsub.publisher"
```

### Create the function directory and files

```bash
mkdir func && cd func
touch index.js package.json
```

### `index.js`

Open the file for editing:

```bash
nano index.js
```

> **Important:** Replace `REPLACE_WITH_YOUR_TOPIC ID` on line 15 with your actual `$TOPIC_NAME` value.

```javascript
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require("@google-cloud/storage");
const gcs = new Storage();
const { PubSub } = require("@google-cloud/pubsub");
const imagemagick = require("imagemagick-stream");

exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64";
  const bucket = gcs.bucket(bucketName);
  const topicName = "REPLACE_WITH_YOUR_TOPIC ID";
  const pubsub = new PubSub();
  if (fileName.search("64x64_thumbnail") == -1) {
    var filename_split = fileName.split(".");
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(
      0,
      fileName.length - filename_ext.length,
    );
    if (
      filename_ext.toLowerCase() == "png" ||
      filename_ext.toLowerCase() == "jpg"
    ) {
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename =
        filename_without_ext + size + "_thumbnail." + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
            gcsNewObject.setMetadata(
              {
                contentType: "image/" + filename_ext.toLowerCase(),
              },
              function (err, apiResponse) {},
            );
            pubsub
              .topic(topicName)
              .publisher()
              .publish(Buffer.from(newFilename))
              .then((messageId) => {
                console.log(`Message ${messageId} published.`);
              })
              .catch((err) => {
                console.error("ERROR:", err);
              });
          });
      });
    } else {
      console.log(
        `gs://${bucketName}/${fileName} is not an image I can handle`,
      );
    }
  } else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```

### `package.json`

Open the file for editing:

```bash
nano package.json
```

```json
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```

### Deploy the Cloud Function

```bash
gcloud functions deploy $FUNCTION_NAME \
  --gen2 \
  --runtime nodejs20 \
  --entry-point thumbnail \
  --source . \
  --region $REGION \
  --trigger-bucket $BUCKET_NAME \
  --allow-unauthenticated \
  --trigger-location $REGION \
  --max-instances 5 \
  --quiet
```

---

## Verify — Upload the test image

```bash
wget -q https://storage.googleapis.com/cloud-training/arc102/wildlife.jpg
gsutil cp wildlife.jpg gs://$BUCKET_NAME
```

Wait a moment, then check the bucket for the generated thumbnail:

```bash
gsutil ls gs://$BUCKET_NAME
```

You should see both `wildlife.jpg` and a `wildlife64x64_thumbnail.jpg` file.

---

## Author

**Aayush Shrestha (Shinux)** — Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)
