# Spark Structured Streaming - save and summarize a stream of tweets on vaccines

## Table of contents
* [General info](#general-info)
* [Technologies](#technologies)
* [Setup](#setup)
* [](#setup)

## General info
The purpose of the project is to build an application to visualize the sentiment towards vaccines based on tweets, with live updates.
	
## Technologies
Project is created with:
* xx version: 12.3
* xx version: 2.33
* x version: 999
	
## Setup
To run this project, do this:
1. Set up a Twitter developer account.
1.1 Save the access token and the access token secret - they will be used to get access to Twitter API by the application.
2. Register to a Databricks community edition platform.
2.1 Set up a new cluster. 

## Part 1: Start the stream

The notebook "tweetStream clean" contains the code to set up the connection to Twitter and a TweetListener class which specifies what happens once Twitter sends the tweets.

### 1.1 Set up the connection to Twitter

Start by creating a socket and configuring Twitter access with the developer keys (stored in a separate notebook which I import with ```%run ./keys``` at the start). Then, proceed to start the stream of tweets by calling the TweetListener class (described below in section 1.2). I decided to filter the tweets which contain keyword "vaccine". Finally, it closes the connection and socket once the streaming is done (once the limit of 1000 tweets is reached - I've chosen 1000 for illustration purposes. Note that Twitter imposes limits on their service use on their end). 

```python
client_socket = socket.socket()  # create a socket 
    
# app will use localhost and port 9876
client_socket.bind(('localhost', 9876))  
 
print('Waiting for connection')
client_socket.listen()  # wait for client to connect
    
# when connection received, get connection/client address
connection, address = client_socket.accept()  
print(f'Connection received from {address}')
 
# configure Twitter access
auth = tweepy.OAuthHandler(api_key, api_secret_key)
auth.set_access_token(access_token, access_token_secret)
    
# configure Tweepy to wait if Twitter rate limits are reached
api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)               
 
# create the Stream
twitter_stream = tweepy.Stream(api.auth, TweetListener(api, connection))
twitter_stream.filter(track=['vaccine']) 

connection.close()
client_socket.close()
```

### The TweetListener class

The TweetListener class defines what happens when the application connects to Twitter:
1. Notifies that a connection was successfully established.
2. Tracks the number of tweets received.
3. Prints out the body of the tweet text.
4. And, finally, sends the text to the client application which is listening on the socket. Note that I've augmented each tweet before sending it to the client application to make it easier to track the start and the end of the tweet by the client application.

```python
class TweetListener(tweepy.StreamListener):

    def __init__(self, api, connection, limit=1000):
        """Create instance variables for tracking number of tweets."""
        self.connection = connection
        self.tweet_count = 0
        self.TWEET_LIMIT = limit  # 1000 by default
        super().__init__(api)  # call superclass's init

    def on_connect(self):
        """Called when your connection attempt is successful."""
        print('Successfully connected to Twitter\n')

    def on_data(self, data):
        """Called when Twitter pushes a new tweet"""
        
        # track number of tweets processed
        self.tweet_count += 1  
        print(self.tweet_count)
        
        # capture the full tweet message with properties in JSON format:
        json_data = json.loads(data)
        
        # print out the text of the tweet:
        try:
            # in case the tweet is longer than 140 characters, need to access extended_tweet property
            tweetStr = json_data['extended_tweet']['full_text']
        except Exception as e:
            try:
                # in case it's 140 characters, use text property
                tweetStr = json_data['text']
            except KeyError:
                return True
        print(tweetStr)
        
        # send the message to tweet listener
        try:
            # send requires bytes, so encode the string in utf-8 format
            self.connection.send(("Tweet " + str(self.tweet_count) + ": " + tweetStr + "t_end").encode('utf-8'))
        except Exception as e:
            print(f'Error: {e}')

        # if TWEET_LIMIT is reached, return False to terminate streaming
        return self.tweet_count < self.TWEET_LIMIT
    
    def on_error(self, status):
        print(status)
        return True
```

## Part 2: Listen to the streem





```
$ cd ../lorem
$ npm install
$ npm start
```

## Create cluser

Sources:



