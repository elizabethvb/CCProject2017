# CCProject2017
Twitter Analysis Project: PageRank - Cloud Computing Fall 2017 - ITCS 8190
Elizabeth von Briesen

# Overview & Motivation
This project seeks to use Hadoop's MapReduce libraries to calculate the PageRank of users inside the Twitter dataset collected
during the 2016 protests in Charlotte, NC after the police shooting of Keith Lamont Scott. The PageRank is calculated for users whose 
tweets are retweeted by others in the dataset. PageRank provides a measure of influence, as it indicates how many Twitter users implicitly
classify another user as an "authority" by retweeting his or her content. In the calculation, these "authority" nodes in the network in
turn divide their PageRank value among the nodes/users for whom they themselves retweet content.

Once the PageRank of Twitter users whose tweets have been retweeted by others has been established, we can then begin to look at the 
aggregate content of their set of tweets in the dataset. If we take a high PageRank as a measure of influence in a network, the content
of the tweets/communication of any user with that rank is worth examining. News outlets naturally appear near the top of the PageRank
list, and are often ignored in favor of examining "civilian" users. However, the reach and influence of news outlets is great, and the
polarity, sentiment, and more of their communication is certainly important.

Identifying the most influential users according to how many people retweet their posts, is a task that yields interesting and relevant 
results. Examining the content of tweets alone in a dataset such as this one provides a window into the conversation happening around the 
events of 2016. However, linking that content with the level of user influence according to his or her PageRank or some other influence 
measure (for example - eigencentrality) allows us to zoom in on the content of the conversation specific to the most influential actors.
We can also then compare the qualities of this text to that of "lower level" users who are not often retweeted, but are still part of the 
network.

The final result of this project thus far is a ranked list of users in the dataset (in decreasing order) who have had their content
retweeted by another user.

# Data
The data for this project is held on UNCC's Sophi cluster, managed by DSI. It is the CharlotteProtest data, and specifics about this set
of Twitter data, and others, is saved here: GnipDatasets 2-9-2016.xlsx location: https://cci-hadoopm3.uncc.edu/filebrowser/#/twitter/gnip/public

From the above file we have:
Subject: Charlotte Protest
Dates 9/20-26/2016
Terms: 7 protest related hashtags
Number of Tweets: 1,358,469
Retweet Percentage: 86.3%

The original json file is approximately 6GB, but we filtered that data to extract only the Tweet ID, Tweet body, posting user, and
user who originated the post. This final data includes only posts that have been retweeted, discarding the original post and other
posts that were never retweeted.

In sum, the number of unique users who retweeted the tweets of other users (unique meaning that we do not double count those who 
retweet more than one time) is 396,452. 

The number of unique users whose content was retweeted by others (unique here meaning that we do not double count those who have their
posts retweeted more than once) is 27,616.

The final features used are:
Tweet ID
Tweet Body
Username of person retweeting
Username of person who originated the retweeted content

# Algorithms & Techniques
## Data Cleaning:
We used basic Python code to extract the above features and clean the Tweet Body text of special characters and urls. In this process,
we also filterd by "verb," only extracting posts with this feature set as "share," indicating a retweet.

## PageRank Algorithm
In order to calculate the PageRank of users whose content was retweeted, we followed the process below:

Let:
InNodeUser = user whose content was retweeted
OutNodeUser = user who retweeted someone else's content
n_out = total number of OutNodeUsers

We used Hadoop's MapReduce framework to compute PageRank as follows:
1. Compute n_out from the data by aggregating according to unique users and counting the total number of unique users who 
retweeted the content of others.

2. Create a LinkGraph which holds the following information: node_Username and delimiter separated string containing that nodes's
PageRank and a list of all nodes for which node_Username retweeted content. Note that initial PageRank is set to 1/n_out.
  a. Mapper: reads in a tweet from the original data file, and sends <OutNodeUser, InNodeUser> to Reducer
  b. Reducer: gathers all nodes for that OutNodeUser (who retweeted something), and writes out <OutNodeUser, "InitPageRank_
    InNodeUser1_InNodeUser2_..._InNodeUsern"> (where _ indicates a delimiter) as a row in the LinkGraph of the overall network.

3. Iteratively calculate and update the PageRank of each node_Username from Step 2 as follows:
  a. Mapper: receives output from Step 2 and
    - for every OutNodeUser, writes <OutNodeUser, string from 2b Reducer task>
    - then writes out <InNodeUserx, PageRank contribution from associated OutNodeUser> where
      (PageRank contribution) = (OutNodeUser PageRank)/(number of InNodeUsers in list)
  b. Reducer: 
    - if the Reducer sees (due to presense of unique character string on value received from Mapper) that its input is an 
      OutNodeUser's LinkGraph, it saves the OutNodeUser and its associated LinkGraph
    - otherwise the Reducer knows that the value associated with the OutNodeUser key is a PageRank contribution from another node
      and it sums those values to a NewPageRank
    - calculate FinalPageRank according to the following formula: finalPageRank = ((1-df) + (df * NewPageRank)) where df
      is the damping factor, and is set to .85 in this case.
    - when finished with the above steps, the Reducer writes out <OutNodeUser, "finalPageRank_
      InNodeUser1_InNodeUser2_..._InNodeUsern"> (where _ indicates a delimiter) as a row in the LinkGraph of the overall network.
  c. Repeat step 3 for a pre-defined number of iterations (in this case 10) to obtain convergence of PageRank values
  
4. Sort results from Step 3:
  a. Mapper: receives final output from Step 3 and writes <-PageRank, OutNodeUser> (we take -PageRank so reducer will ultimately
    sort in descending order.)
  b. Reducer: receives mapper output and writes <OutNodeUser, PageRank> to output folder

# Results
See included .txt file with OutNodeUser list ranked in descending order.

# Performance evaluation
This was not a predicitive machine learning task, and therefore there are no performance metrics. However, it would have been good
to use another method for calculating PageRank in this network using a standard library or software package and then compare. The 
absolute values would likely have been different, but ranking order should be the same.

# Tasks accomplished
Definitely do:
This was a modification of my original project proposal because issues with the University Research Cluster prevented me from working
on the original proposed task. I mentioned at the end of the proposal document that if I had to go to my backup project (which is what you
see above), I would perform some network analytics on the above Twitter dataset. PageRank is only one such measure, but it was good 
to finally get the data cleaned, prepared, and run through the algorithms in order to rank the users.

Likely will do:
I also indicated that I would implement some unsupervised machine learning algorithms, and I felt this was likely at the time I made the 
proposal. However, it took quite a lot of time to simply perform the PageRank computation, and I did not have the ability to use Spark 
to perform any unsupervised machine learning on this dataset. I also think that from the perspective of understanding the data around
influential users, it would have been better to perform WordCount and other similar information retrieval techniques on the corpus of tweets
associated with the most influential users.

Ideally will do:
Ideally I would have taken the words associated with influential users' tweets and performed sentiment and topic analysis on those in order
to better understand the tone of the conversation around the protests. Additionally, it would have been good to visualize the graph,
at least for the top 100 most influential users according to their PageRank.

# Update 1 - WordCount calculation
We were able to do a very preliminary look at the WordCount of the total corpus of tweets associated with the most influential user
according to PageRank (Stevenwhirsch99). The file is attached (user1WordCounts.txt), and is not sorted. This is a starting point for 
continued analysis. Stopwords must be removed, words must be ranked, and then sentiment analysis applied. Many of the tweets are repeated
numerous times, and while this may initially seem to be problematic, it indicates the WEIGHT with which those words appear in the 
protest conversation. Their sentiment carries further because they are seen more often by more people.

# Update 2 - Sentiment Analysis for Top Users by PageRank
In continuing to work toward obtaining average sentiment by user in ranked order, we were able to obtain some preliminary results.

The top 5 users by PageRank (who were retweeted by others) in the data set are (full list can be found in the finalUserSortbyPR.txt file):
a.	DOJSuedFerguson	28.3622887396919
b.	kemba722	26.134394384073403
c.	maxrafaelwaller	16.10228462705418
d.	jmo201	15.215956236974769
e.	FeltzBilly	14.648017400070158

For the above users, we combined all their tweets into a single corpus, allowing for repeated words since they represent a flow of communication across the entire network. The goal is to weight the sentiment of a particular user's Twitter communication by the volume of each individual word as it is used.

Using TweetBlob's polarity measure of sentiment, we calculated the weighted sentiment for each of the top 5 users. As a measure of influence, we then multiplied the sentiment by the user PageRank in order to understand the magnitude of the user's impact given either a positive or negative sentiment on the protests. The results can be found in the finalUserPolarityData.pdf file.

                                          
# Future Work
We need to refine the above process, error check the code and methodology, and then compute the Sentiment * PageRank scores for the entire set of retweeted users. The same process can be followed across other data sets in order to better understand how sentiment, as well as measures of influence like PageRank, differ across events and their corresponding corpora. This can ultimately provide us with a quantification of sentiment that can be used to initialize simulation parameters for models of social dynamics.









