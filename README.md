# COEN 281 Group 7 Term Project

## Initial Setup
Our code is written in Python 3.5.  We have included a *requirements.txt* file that includes all of the packages necessary to run our project.  To install them run `pip install -r requirements.txt`.

### Twitter Social Graph Dataset
Dataset is downloadable from: http://socialcomputing.asu.edu/datasets/Twitter.
After downloading, copy the contents from the `data` directory in the download
into the `data` directory here.

## Formatting Dataset and Getting Tweets
The current format of the dataset is too large for us to work with in a resonable amount of time.
We wrote a script to reformat a subset of the larger dataset and then we wrote a script to assign tweets to those users.

### Formatting Social Graph
Too run the script:
  `python format_data.py --edge_filename data/edges.csv --outfile formatted_data.json --num_edges 10000`

This will process 10,000 edges from the larger dataset into an easier to work with JSON format named "formatted_data.json".  You can change the number of edges accordingly.  You can also have the script only read a certain percentage of the larger dataset by running:
  `python format_data.py --edge_filename data/edges.csv --outfile formatted_data.json --skip_lines 25`

This will tell the script to read 1 in every 25 lines.  This is the command we used for our final output.

### Getting Tweets
In order to gather the tweets, we used the Twitter Streaming API and wrote the corresponding Tweets into a SQLite database.  We also load and write the formatted data from the last step into a different table in the same database.

First, you need to get the Twitter API credentials in order to make API calls.  Go to https://apps.twitter.com/ to create a new app.  This requires that you also have a Twitter account.  Once you've created the app, your app settings will list four different keys:
  1. Consumer Key (API Key)
  2. Consumer Secret (API Secret)
  3. Access Token
  4. Access Token Secret

You will need all of these values and set them as environment variables in order to run our tweet collection script.
The environment variables are:
  1. TWITTER_CONSUMER_KEY
  2. TWITTER_CONSUMER_SECRET
  3. TWITTER_API_KEY
  4. TWITTER_API_SECRET

Once these values are set, you can run the script collection by:
  `python get_tweets.py formatted_data.json "trump, donald trump, realDonaldTrump"`

The first argument is the formatted dataset from the previous step.  The second is a query that you want to gather tweets for.  In our project we used "trump, donald trump, realDonaldTrump" which looks for any tweets that have the phrases "trump", "donald trump", or "realDonaldTrump".

This process will also loaded the social graph from the formatted_data.json file into a table called "users".  That loading process can take some time and only needs to be run once.  You can skip that process by calling:
 `python get_tweets.py formatted_data.json "trump, donald trump, realDonaldTrump" --skip_table_load`

 The user table contains:
  - *id*: a unique integer.  All of the ids are sequential from 0 to the number of users
  - *user_id*: the user id from the formatted dataset

 The script will run and continue collecting Tweets until:
  a) it has collected 10,000,000 tweets (completely arbitrarily chosen number to prevent dataset from growing too large)
  b) you kill the script

The tweets will be written to a table named "tweets" which will contain:
  - *id*: ID of the tweet.  This is a unique value to prevent us from writting the same tweet multiple times
  - *user_id*: the hashed value of the ID of the user who made the tweet.
  - *tweet*: the actual text of the tweet

You can join these tables with the following query:
  `SELECT * FROM users, tweets WHERE users.id = tweets.user_id`
