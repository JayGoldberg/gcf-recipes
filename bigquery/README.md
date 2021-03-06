# Google Cloud Functions Recipes
## Cloud BigQuery

### Overview
This recipe demonstrates using Cloud Functions to import data into BigQuery in response to a file upload event in Cloud Storage.  Where applicable:

**Replace [PROJECT-ID] with your Cloud Platform project ID**

### Notes on deployment
If you have a large number of dependencies in your `package.json` file and you run `npm install` locally, you may find deployments to be slow because you will be deploying your fully-materialized `node_modules` folder.  If you don't have any private modules in your `node_modules` folder we recommend you `rm -rf node_modules` prior to invoking `deploy`

### Cooking the Recipe
1.	Follow the [Cloud Functions quickstart guide](https://cloud.google.com/functions/quickstart) to setup Cloud Functions for your project

2.	Clone this repository

		cd ~/
		git clone https://github.com/jasonpolites/gcf-recipes.git
		cd gcf-recipes/bigquery

3. 	Create a Cloud Storage Bucket to stage our deployment

		gsutil mb gs://[PROJECT-ID]-gcf-recipes-bucket

4. 	Create a Cloud Storage Bucket to recieve files.  This is the bucket we will watch for changes

		gsutil mb gs://[PROJECT-ID]-bq-in

5. 	Create a Cloud Storage Bucket to house processed files.  This is the bucket we will move files to when they are imported.  This should match the bucket name in [config.js](config.js)

		gsutil mb gs://[PROJECT-ID]-bq-processed

6.	Deploy the "onFileArrived" function with a Cloud Storage trigger.  We'll use the same name as step #4 above.  This will cause our function to execute when files are written to this bucket
	
		gcloud alpha functions deploy onFileArrived --bucket [PROJECT-ID]-gcf-recipes-bucket --trigger-gs-uri [PROJECT-ID]-bq-in

7. 	Upload a file to the bucket.  We're going to use a sample CSV containing DC Comics superhero data

		gsutil cp dc-wikia-data.csv gs://[PROJECT-ID]-bq-in
		
8.	Check the logs for the "onFileArrived" function to make sure the function was triggered and the data was imported

		gcloud alpha functions get-logs onFileArrived --min-log-level INFO --limit 100
		
You should see something like this in your console
```
I ... onFileArrived triggered
I ... Sending file dc-wikia-data.csv to BigQuery...
I ... Importing data from dc-wikia-data.csv into gcf_bq_dataset/gcf_bq_table...
I ... Data import job created
I ... File imported successfully, marking as done...
I ... File marked as done.  Function complete.
```

9.	Now you can query the dataset using the bq command line utility.  Just how many white-haired, green-eyed, living, evil superhero characters are there?

		bq query "SELECT NAME, YEAR FROM [gcf_bq_dataset.gcf_bq_table] WHERE EYE='Green Eyes' AND ALIGN='Bad Characters' AND HAIR='White Hair' AND ALIVE='Living Characters'"


You should see this result:
```
+---------------------------+------+
|           NAME            | YEAR |
+---------------------------+------+
| Joshua Walker (New Earth) | 2008 |
| Doomslayer (New Earth)    | 2011 |
+---------------------------+------+
```

#### Running Tests
This recipe comes with a suite of unit tests.  To run the tests locally, just use `npm test`

```
npm install
npm test
```

The tests will also produce code coverage reports, written to the `/coverage` directory.  After running the tests, you can view coverage with

```
open coverage/lcov-report/index.html 
```
