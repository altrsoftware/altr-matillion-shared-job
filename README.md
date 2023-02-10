This integration is a Matillion shared job that will tokenize any columns in a table that match a user-created list of classifiers. ALTR will then govern the columns to allow policy to be placed over that column to allow for role-based detokenization.

## Prerequisites
- An Enterprise+ ALTR account with API keys created (Settings -> Preferences -> API)
- ALTR service user is created - Credentials for your ALTR Service user https://docs.altr.com/tutorial-and-user-guide/connecting-your-snowflake-database

## Setup
1. Create the following API integration and External functions in Snowflake. Please replace "<api_key>" and "<api_secret>" with your ALTR api key and secret. Contact support@altr.com for the "ALTR_role_arn" value. 
```
create or replace api integration "altr_tokenization"
  api_provider = aws_api_gateway
  API_KEY = '<api_key>:<api_secret>'
  api_aws_role_arn = '<ALTR_role_arn>'
  api_allowed_prefixes = ('https://0p1p63orv8.execute-api.us-east-1.amazonaws.com/v1')
  enabled = true;

create or replace external function altr_tokenize(string_col varchar)
    returns variant
    api_integration = "altr_tokenization"
    headers = ('determinism'='true')
    as 'https://0p1p63orv8.execute-api.us-east-1.amazonaws.com/v1/tokenize';

create or replace external function altr_detokenize(string_col varchar, disposition varchar)
    returns variant
    api_integration = "altr_tokenization"
    headers = ('determinism'='true')
    as 'https://0p1p63orv8.execute-api.us-east-1.amazonaws.com/v1/detokenize';
```
2. Import the API Extract Profile. On the top left of Matillion click the "Project" dropdown, then "Manage API Profiles", then Manage Extract Profiles. Click the import symbol, then choose the .json file.
3. Import the Shared Job into Matillion. On the top left of Matillion click the "Project" dropdown, then "Manage Shared Jobs". Then click the import button and import the .METL shared job file.
4. Add the ALTR component to your job. On the bottom left of Matillion, click the "Shared Jobs" section, then navigate to the "altr" -> "demo" -> "tokenize" -> "Classifier_tokenize" job. Click and drag to add it to your job.
5. Fill in the fields in the component. The fields are
	1. **database**, **schema**, **table** - Location of the table to classify and protect
	2. **ef_database**, **ef_schema**, **ef_tokenize_name**, **ef_detokenize_name** - sections of the fully qualified names of the external functions created in step 1
	3. **ALTR_api_key** - ALTR API key (used when creating the api integration in step 1)
	4. **ALTR_api_secret_name** - name of the ALTR API secret (stored under "Project" -> "Manage Project Group Passwords").
	5. snowflake_hostname - Your Snowflake hostname in the format "example.snowflakecomputing.com"
	6. Altr_service_user_name - The Snowflake username of the ALTR service user
	7. Altr_service_user_password - The password for the ALTR service user
	8. Classifiers_to_tokenize - a list of classifiers that should be tokenized
		1. Full List: "US_PASSPORT","US_SOCIAL_SECURITY_NUMBER","US_STATE","VEHICLE_IDENTIFICATION_NUMBER","US_DRIVERS_LICENSE_NUMBER","US_EMPLOYER_IDENTIFICATION_NUMBER","US_INDIVIDUAL_TAXPAYER_IDENTIFICATION_NUMBER","ICD9_CODE", "ICD10_CODE","IP_ADDRESS", "LOCATION", "MAC_ADDRESS", "PASSPORT", "PHONE_NUMBER", "STREET_ADDRESS","CREDIT_CARD_TRACK_NUMBER", "DATE_OF_BIRTH", "DATE", "FIRST_NAME", "LAST_NAME", "PHONE_NUMBER", "EMAIL_ADDRESS", "CREDIT_CARD_NUMBER", "PERSON_NAME"
6. Run the Component

