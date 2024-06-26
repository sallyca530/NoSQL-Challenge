# Food Hygiene Ratings

## Introduction
In this project, a NoSQL database is utilized to address the evaluation of food hygiene ratings provided by the UK Food Standards Agency. Leveraging MongoDB and Pymongo, a database is set up, performing necessary updates, and conducting exploratory analysis to extract valuable insights. This dataset included over 38,000 restuarants.

## Part 1: Database and Jupyter Notebook Set Up

**Utilizing NoSQL_setup.ipynb, we import 'establishments' data into MongoDB, configure the database, and ensure proper data loading.**

* Import the data provided in the `establishments.json` file from your *Terminal*. Name the database `uk_food` and the collection `establishments`.

* Before running any of the following code, make sure you import the dataset with 
`mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json`

**Import essential libraries**

    # Import dependencies
    from pymongo import MongoClient
    from pprint import pprint

**Instantiate the Mongo Client**

    # Create an instance of MongoClient
    mongo = MongoClient(port=27017)

**Confirm database creation**

    print(mongo.list_database_names())

    Output: ['admin', 'autosaurus', 'classDB', 'config', 'epa', 'fruits_db', 'gardenDB', 'local', 'met', 'petsitly_marketing', 'trabel_db', 'uk_food']

**Prepare the establishments collection for analysis**

    # assign the uk_food database to a variable name
    db = mongo['uk_food']

    # review the collections in our new database
    print(db.list_collection_names())
    
    Output: ['establishments']

**Assign the collection to a variable**

    establishments = db['establishments']

## Part 2: Update the Database
In the same notebook, requested modifications to the establishments collection should be performed: 

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

**Insert the new restaurant into the collection**

    establishments.insert_one(new)

**Check that the new restaurant was inserted**

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

**Updating business types: find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the `BusinessTypeID` and `BusinessType` fields**

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


**Update the new restaurant with the correct BusinessTypeID**

    establishments.update_one(new, {'$set': {'BusinessTypeID': 1}})

**Confirm that the new restaurant was updated**

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


**Eliminating unwanted data: uninterested in restaurants in Dover.**

    query = {"LocalAuthorityName":"Dover"}

**Find how many documents have LocalAuthorityName as "Dover"**

    establishments.count_documents(query)

    Output: 994

**Delete all documents where LocalAuthorityName is "Dover"**

    establishments.delete_many(query)

    Output: DeleteResult({'n': 994, 'ok': 1.0}, acknowledged=True)

**Check if any remaining documents include Dover**

    establishments.count_documents(query)

    Output: 0


**Converting relevant string values to numeric types: change the data type from String to Decimal for longitude and latitude**

    establishments.update_many({}, [ {'$set':{ 'geocode.longitude' : {'$toDecimal': '$geocode.longitude'},'geocode.latitude': {'$toDecimal': '$geocode.latitude'}}} ])

    Output: UpdateResult({'n': 38786, 'nModified': 38786, 'ok': 1.0, 'updatedExisting': True}, acknowledged=True)

**Set non 1-5 Rating Values to Null**

    non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
    
    establishments.update_many({"RatingValue": {"$in": non_ratings}}, [ {'$set':{ "RatingValue" : None}} ])

    Output: UpdateResult({'n': 4091, 'nModified': 4091, 'ok': 1.0, 'updatedExisting': True}, acknowledged=True)


**Change the data type from String to Integer for RatingValue**

    establishments.update_many({}, [ {'$set':{ "RatingValue" : {'$toInt': "$RatingValue"}}} ]) 

    Output: UpdateResult({'n': 38786, 'nModified': 34695, 'ok': 1.0, 'updatedExisting': True}, acknowledged=True)  


## Part 3: Exploratory Analysis
Moving to NoSQL_analysis.ipynb, the establishments collection is explored. Based on hygiene scores, ratings, proximity to specified locations, employing count_documents, pprint, and Pandas DataFrame (found in Jupyter notebook), findings are presented effectively.

#### 1. Which establishments have a hygiene score equal to 20?

    # Find the establishments with a hygiene score of 20
    query = {"scores.Hygiene": 20}

    # Use count_documents to display the number of documents in the result
    twenty_hygiene = establishments.count_documents(query)
    print(f"{twenty_hygiene} documents show a hygiene score equal to 20.")

    Output: 41 documents show a hygiene score equal to 20.


#### 2. Which establishments in London have a `RatingValue` greater than or equal to 4?

    # Find the establishments with London as the Local Authority and has a RatingValue greater than or equal to 4.
    query = {'LocalAuthorityName': {'$regex': 'London'}, 'RatingValue' : {"$gte" : 4}}

    # Use count_documents to display the number of documents in the result
    four_rating_value = establishments.count_documents(query)
    print(f"{four_rating_value} documents have a rating value greater than or equal to 4.")

    Output: 33 documents have a rating value greater than or equal to 4.

#### 3. What are the top 5 establishments with a `RatingValue` rating value of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?

    # Search within 0.01 degree on either side of the latitude and longitude.
    # Rating value must equal 5
    # Sort by hygiene score

    degree_search = 0.01
    latitude = 51.49014200
    longitude = 0.08384000

    query = {'RatingValue': {'$eq' : 5},
            'geocode.latitude' : {'$lte': (latitude + degree_search), '$gte': (latitude - degree_search)},
            'geocode.longitude' : {'$lte': (longitude + degree_search), '$gte': (longitude - degree_search)}}

    sort =  [('scores.Hygiene', 1)]

    # Print the results
    print(f"{establishments.count_documents(query)} documents are shown with this criteria.")

    Output: 87 documents are shown with this criteria.

#### 4. How many establishments in each Local Authority area have a hygiene score of 0?

    # Create a pipeline that: 
    # 1. Matches establishments with a hygiene score of 0
    match = {'$match' : {'scores.Hygiene' : 0}}

    # 2. Groups the matches by Local Authority
    group = {'$group' : {'_id' : '$LocalAuthorityName', 'count' : {'$sum' : 1}}}

    # 3. Sorts the matches from highest to lowest
    sort = {'$sort' : {'count': -1}}

    #Pipeline
    pipeline = [match, group, sort]

    #Results
    results = list(establishments.aggregate(pipeline))

    # Print the number of documents in the result
    print("Number of documents in result: ", len(results))

    Output: Number of documents in result:  55

Print the first 10 results

    pprint(results[0:10])

    Output: [{'_id': 'Thanet', 'count': 1130},
    {'_id': 'Greenwich', 'count': 882},
    {'_id': 'Maidstone', 'count': 713},
    {'_id': 'Newham', 'count': 711},
    {'_id': 'Swale', 'count': 686},
    {'_id': 'Chelmsford', 'count': 680},
    {'_id': 'Medway', 'count': 672},
    {'_id': 'Bexley', 'count': 607},
    {'_id': 'Southend-On-Sea', 'count': 586},
    {'_id': 'Tendring', 'count': 542}]

Convert the result to a Pandas DataFrame

    aggregated_df = pd.json_normalize(results)

    # Display the number of rows in the DataFrame
    print(f"{len(aggregated_df)} rows are in this Dataframe.")

    Output: 55 rows are in this Dataframe.


Display the first 10 rows of the DataFrame
    
    aggregated_df.head(10)

![image](NoSQL_Analysis/Resources/aggregated_df.png)


## References


 
- ($toDecimal) Conversion of object to decimal for geocode of longitude and latitude - https://www.mongodb.com/docs/manual/reference/operator/aggregation/toDecimal/

- ($toInt) RatingValue String to intger - https://www.mongodb.com/docs/manual/reference/operator/aggregation/toInt/
