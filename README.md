# Food Hygiene Ratings

## Introduction
In this project, NoSQL database is utilized to address the evaluation of food hygiene ratings provided by the UK Food Standards Agency. Leveraging MongoDB and Pymongo, we set up a database, perform necessary updates, and conduct exploratory analysis to extract valuable insights. This dataset included over 38,000 restuarants.




## Part 1: Database and Jupyter Notebook Set Up

Utilizing NoSQL_setup.ipynb, we import establishments data into MongoDB, configure the database, and ensure proper data loading.

* Import the data provided in the `establishments.json` file from your Terminal. Name the database `uk_food` and the collection `establishments`.

* Before running any of the following code, make sure you import the dataset with 
`mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json`
<!-- `mongoimport --type json -d met -c artifacts --drop --jsonArray artifacts.json` -->

Import essential libraries

    # Import dependencies
    from pymongo import MongoClient
    from pprint import pprint

Instantiate the Mongo Client

    # Create an instance of MongoClient
    mongo = MongoClient(port=27017)

Confirm database creation

    print(mongo.list_database_names())

    Output: ['admin', 'autosaurus', 'classDB', 'config', 'epa', 'fruits_db', 'gardenDB', 'local', 'met', 'petsitly_marketing', 'trabel_db', 'uk_food']

Prepare the establishments collection for analysis

    # assign the uk_food database to a variable name
    db = mongo['uk_food']

    # review the collections in our new database
    print(db.list_collection_names())
    
    Output: ['establishments']

Assign the collection to a variable

    establishments = db['establishments']

## Part 2: Update the Database
In the same notebook, a requested modifications to the establishments collection: 

**This includes adding a new restaurant entry, "Penang Flavours"**

    # Create a dictionary for the new restaurant data
    new = {
        "BusinessName":"Penang Flavours",
        "BusinessType":"Restaurant/Cafe/Canteen",
        "BusinessTypeID":"",
        "AddressLine1":"Penang Flavours",
        "AddressLine2":"146A Plumstead Rd",
        "AddressLine3":"London",
        "AddressLine4":"",
        "PostCode":"SE18 7DY",
        "Phone":"",
        "LocalAuthorityCode":"511",
        "LocalAuthorityName":"Greenwich",
        "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
        "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
        "scores":{
            "Hygiene":"",
            "Structural":"",
            "ConfidenceInManagement":""
        },
        "SchemeType":"FHRS",
        "geocode":{
            "longitude":"0.08384000",
            "latitude":"51.49014200"
        },
        "RightToReply":"",
        "Distance":4623.9723280747176,
        "NewRatingPending":True
    }

Insert the new restaurant into the collection

    establishments.insert_one(new)

Check that the new restaurant was inserted

    results  = establishments.find(new)
    for result in results:
        pprint(result)


    Output: {'AddressLine1': 'Penang Flavours',
    'AddressLine2': '146A Plumstead Rd',
    'AddressLine3': 'London',
    'AddressLine4': '',
    'BusinessName': 'Penang Flavours',
    'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': '',
    'Distance': 4623.972328074718,
    'LocalAuthorityCode': '511',
    'LocalAuthorityEmailAddress': 'health@royalgreenwich.gov.uk',
    'LocalAuthorityName': 'Greenwich',
    'LocalAuthorityWebSite': 'http://www.royalgreenwich.gov.uk',
    'NewRatingPending': True,
    'Phone': '',
    'PostCode': 'SE18 7DY',
    'RightToReply': '',
    'SchemeType': 'FHRS',
    '_id': ObjectId('6583597a21f3b261257b0fc9'),
    'geocode': {'latitude': '51.49014200', 'longitude': '0.08384000'},
    'scores': {'ConfidenceInManagement': '', 'Hygiene': '', 'Structural': ''}}

Updating business types: Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the `BusinessTypeID` and `BusinessType` fields

    query = {'BusinessType': {'$regex': "Restaurant/Cafe/Canteen"}}
    fields = {"BusinessTypeID": 1, "BusinessType": 1}
    sorts = sort = [("BusinessTypeID", -1)]

    #Results shows that the new restaurant has an empty BusinessTypeID field
    results = list(establishments.find(query, fields).sort(sorts).limit(5))
    pprint(results)

    Output: [{'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': '',
    '_id': ObjectId('6583597a21f3b261257b0fc9')},
    {'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': 1,
    '_id': ObjectId('65835793c397fa253815ac58')},
    {'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': 1,
    '_id': ObjectId('65835793c397fa253815ac59')},
    {'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': 1,
    '_id': ObjectId('65835793c397fa253815ac55')},
    {'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': 1,
    '_id': ObjectId('65835793c397fa253815ac4f')}]


Update the new restaurant with the correct BusinessTypeID

    establishments.update_one(new, {'$set': {'BusinessTypeID': 1}})

Confirm that the new restaurant was updated

    establishments.find_one({'BusinessName': 'Penang Flavours'})

    Output: {'_id': ObjectId('6583597a21f3b261257b0fc9'),
    'BusinessName': 'Penang Flavours',
    'BusinessType': 'Restaurant/Cafe/Canteen',
    'BusinessTypeID': 1,
    'AddressLine1': 'Penang Flavours',
    'AddressLine2': '146A Plumstead Rd',
    'AddressLine3': 'London',
    'AddressLine4': '',
    'PostCode': 'SE18 7DY',
    'Phone': '',
    'LocalAuthorityCode': '511',
    'LocalAuthorityName': 'Greenwich',
    'LocalAuthorityWebSite': 'http://www.royalgreenwich.gov.uk',
    'LocalAuthorityEmailAddress': 'health@royalgreenwich.gov.uk',
    'scores': {'Hygiene': '', 'Structural': '', 'ConfidenceInManagement': ''},
    'SchemeType': 'FHRS',
    'geocode': {'longitude': '0.08384000', 'latitude': '51.49014200'},
    'RightToReply': '',
    'Distance': 4623.972328074718,
    'NewRatingPending': True}


**Eliminating unwanted data. Uninterested in restuarant in Dover.**

    query = {"LocalAuthorityName":"Dover"}

Find how many documents have LocalAuthorityName as "Dover"

    establishments.count_documents(query)

    Output: 994

Delete all documents where LocalAuthorityName is "Dover"

    establishments.delete_many(query)

    Output: DeleteResult({'n': 994, 'ok': 1.0}, acknowledged=True)

Check if any remaining documents include Dover
    establishments.count_documents(query)

    Output: 0


**Converting relevant string values to numeric types**

Change the data type from String to Decimal for longitude and latitude

    establishments.update_many({}, [ {'$set':{ 'geocode.longitude' : {'$toDecimal': '$geocode.longitude'},'geocode.latitude': {'$toDecimal': '$geocode.latitude'}}} ])

    Output: UpdateResult({'n': 38786, 'nModified': 38786, 'ok': 1.0, 'updatedExisting': True}, acknowledged=True)

Set non 1-5 Rating Values to Null

    non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
    
    establishments.update_many({"RatingValue": {"$in": non_ratings}}, [ {'$set':{ "RatingValue" : None}} ])

    Output: UpdateResult({'n': 4091, 'nModified': 4091, 'ok': 1.0, 'updatedExisting': True}, acknowledged=True)


Change the data type from String to Integer for RatingValue

    establishments.update_many({}, [ {'$set':{ "RatingValue" : {'$toInt': "$RatingValue"}}} ]) 

    Output: UpdateResult({'n': 38786, 'nModified': 34695, 'ok': 1.0, 'updatedExisting': True}, acknowledged=True)  


## Part 3: Exploratory Analysis
Moving to NoSQL_analysis_starter.ipynb, we delve into the dataset to answer specific questions posed by "Eat Safe, Love" magazine. We explore establishments based on hygiene scores, ratings, proximity to specified locations, and other criteria, employing count_documents, pprint, and Pandas DataFrame to present findings effectively.

For each question, we provide comprehensive analyses, highlighting key insights and offering actionable recommendations for the magazine's editorial team. Through this project, we aim to streamline their decision-making process and enhance the quality of future articles.

## Conclusion


### References


 
- ($toDecimal) Conversion of object to decimal for geocode of longitude and latitude - https://www.mongodb.com/docs/manual/reference/operator/aggregation/toDecimal/

- ($toInt) RatingValue String to intger - https://www.mongodb.com/docs/manual/reference/operator/aggregation/toInt/
