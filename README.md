# WeblogChallenge
This is an interview challenge for Paytm Labs. Please feel free to fork. Pull Requests will be ignored.

The challenge is to make make analytical observations about the data using the distributed tools below.

## Processing & Analytical goals:

### 1. Sessionize the web log by IP. Sessionize = aggregrate all page hits by visitor/IP during a session.
    https://en.wikipedia.org/wiki/Session_(web_analytics)
   
import pyspark

#imports the pyspark library

log_file = sc.textFile("dbfs:/FileStore/shared_uploads/niveditajoshi22@gmail.com/2015_07_22_mktplace_shop_web_log_sample__2__log-3.gz")

#load the log file into Spark

sessionized_log = log_file.mapPartitions(lambda x: sessionize(x))

#sessionize the web log by IP

def sessionize(x):
  """
  Sessionizes a list of page hits by IP.

  Args:
    x: A list of page hits.

  Returns:
    A list of sessions.
  """

  sessions = []
  current_session = []
  for page_hit in x:
    ip = page_hit.split(",")[0]
    if len(current_session) == 0 or ip == current_session[0][0]:
      current_session.append(page_hit)
    else:
      sessions.append(current_session)
      current_session = [page_hit]
  if len(current_session) > 0:
    sessions.append(current_session)
  return sessions

#the code def sessionize(x) takes a list of page hits and groups them together by IP address. it creates a new list for each unique IP address that appears in the log file and then returns the list of sessions.


### 2. Determine the average session time

average_session_time = sessionized_log.map(lambda x: len(x)).mean()

#determining the average session time

print(average_session_time)

#prints the average session time


### 3. Determine unique URL visits per session. To clarify, count a hit to a unique URL only once per session.

def unique_url_visits_per_session(sessions):
  """
  Determines the unique URL visits per session.

  Args:
    sessions: A list of sessions.

  Returns:
    A dictionary of unique URL visits per session.
  """

  unique_url_visits = {}
  for session in sessions:
    unique_urls = set()
    for page_hit in session:
      url = page_hit.split(",")[1]
      unique_urls.add(url)
    unique_url_visits[session] = len(unique_urls)
  return unique_url_visits

#we are passing the list of sessions to the unique_url_visits_per_session function which will then return a dictionary of unique URL visits per session.

print(unique_url_visits_per_session)

#prints the unique URL visits per session

### 4. Find the most engaged users, ie the IPs with the longest session times

def most_engaged_users(sessions):
  """
  Finds the most engaged users.

  Args:
    sessions: A list of sessions.

  Returns:
    A list of the most engaged users.
  """

  most_engaged_users = []
  for session in sessions:
    session_time = len(session)
    if len(most_engaged_users) < 5 or session_time > most_engaged_users[-1][1]:
      most_engaged_users.append((session[0][0], session_time))
  return most_engaged_users
#passing the list of sessions to the most_engaged_users function. The function will then return a list of the most engaged users.The list of the most engaged users will be sorted by session time, with the most engaged users at the top of the list.

print(most_engaged_users)

#prints the most engaged users
   

## Additional questions for Machine Learning Engineer (MLE) candidates:
1. Predict the expected load (requests/second) in the next minute

2. Predict the session length for a given IP

3. Predict the number of unique URL visits by a given IP

## Tools allowed (in no particular order):
- Spark (any language, but prefer Scala or Java)
- Pig
- MapReduce (Hadoop 2.x only)
- Flink
- Cascading, Cascalog, or Scalding

If you need Hadoop, we suggest 
HDP Sandbox:
http://hortonworks.com/hdp/downloads/
or 
CDH QuickStart VM:
http://www.cloudera.com/content/cloudera/en/downloads.html


### Additional notes:
- You are allowed to use whatever libraries/parsers/solutions you can find provided you can explain the functions you are implementing in detail.
- IP addresses do not guarantee distinct users, but this is the limitation of the data. As a bonus, consider what additional data would help make better analytical conclusions
- For this dataset, complete the sessionization by time window rather than navigation. Feel free to determine the best session window time on your own, or start with 15 minutes.
- The log file was taken from an AWS Elastic Load Balancer:
http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/access-log-collection.html#access-log-entry-format



## How to complete this challenge:

A. Fork this repo in github
    https://github.com/PaytmLabs/WeblogChallenge

B. Complete the processing and analytics as defined first to the best of your ability with the time provided.

C. Place notes in your code to help with clarity where appropriate. Make it readable enough to present to the Paytm Labs interview team.

D. Complete your work in your own github repo and send the results to us and/or present them during your interview.

## What are we looking for? What does this prove?

We want to see how you handle:
- New technologies and frameworks
- Messy (ie real) data
- Understanding data transformation
This is not a pass or fail test, we want to hear about your challenges and your successes with this particular problem.
