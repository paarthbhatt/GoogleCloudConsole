# Google Cloud Natural Language API Lab

## Task 1: Create an API Key

### Step 1: Set Environment Variable
First, set an environment variable with your `PROJECT_ID` which you will use throughout this lab:
```sh
export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value core/project)
```

### Step 2: Create Service Account
Create a new service account to access the Natural Language API:
```sh
gcloud iam service-accounts create my-natlang-sa \
  --display-name "my natural language service account"
```

### Step 3: Create Credentials
Create credentials to log in as your new service account. Save these credentials as a JSON file `~/key.json` using the following command:
```sh
gcloud iam service-accounts keys create ~/key.json \
  --iam-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
```

### Step 4: Set Credentials Environment Variable
Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to the full path of the credentials JSON file you created:
```sh
export GOOGLE_APPLICATION_CREDENTIALS="/home/USER/key.json"
```

### Step 5: Create an API Key
Follow the instructions to create an API key for accessing the Cloud Natural Language API.

## Task 2: Make an Entity Analysis Request

### Step 1: Connect to VM Instance
Connect to the instance provisioned for you via SSH. Open the navigation menu and select Compute Engine. Find the provisioned Linux instance and click on the SSH button to open an interactive shell. Remain in this SSH session for the rest of the lab.

### Step 2: Run Entity Analysis
Try out the Natural Language API's entity analysis with the following sentence:
```sh
gcloud ml language analyze-entities --content="Michelangelo Caravaggio, Italian painter, is known for 'The Calling of Saint Matthew'." > result.json
```

### Step 3: Preview Output
Run the command below to preview the output of the `result.json` file:
```sh
cat result.json
```

### Sample Output
You should see a response similar to the following in the `result.json` file:
```json
{
  "entities": [
    {
      "name": "Michelangelo Caravaggio",
      "type": "PERSON",
      "metadata": {
        "wikipedia_url": "http://en.wikipedia.org/wiki/Caravaggio",
        "mid": "/m/020bg"
      },
      "salience": 0.83047235,
      "mentions": [
        {
          "text": {
            "content": "Michelangelo Caravaggio",
            "beginOffset": 0
          },
          "type": "PROPER"
        },
        {
          "text": {
            "content": "painter",
            "beginOffset": 33
          },
          "type": "COMMON"
        }
      ]
    },
    {
      "name": "Italian",
      "type": "LOCATION",
      "metadata": {
        "mid": "/m/03rjj",
        "wikipedia_url": "http://en.wikipedia.org/wiki/Italy"
      },
      "salience": 0.13870546,
      "mentions": [
        {
          "text": {
            "content": "Italian",
            "beginOffset": 25
          },
          "type": "PROPER"
        }
      ]
    },
    {
      "name": "The Calling of Saint Matthew",
      "type": "EVENT",
      "metadata": {
        "mid": "/m/085_p7",
        "wikipedia_url": "http://en.wikipedia.org/wiki/The_Calling_of_St_Matthew_(Caravaggio)"
      },
      "salience": 0.030822212,
      "mentions": [
        {
          "text": {
            "content": "The Calling of Saint Matthew",
            "beginOffset": 69
          },
          "type": "PROPER"
        }
      ]
    }
  ],
  "language": "en"
}
```
Read through your results. For each "entity" in the response, you'll see:
- The entity name and type (person, location, event, etc.).
- Metadata, including an associated Wikipedia URL if available.
- Salience, a number in the [0,1] range that refers to the centrality of the entity to the text as a whole.
- Mentions, which list the same entity mentioned in different ways.

### Conclusion
Congratulations! You have successfully extracted entities from a snippet of text using the Cloud Natural Language API.