Reference: https://www.infoq.com/presentations/Twitter-Timeline-Scalability/

![[Pasted image 20250323081958.png]]
Steps:
- My posting of a tweet goes to Write API
- Fanount service will talk to the social graph service and get the list of people who follow me
- Twitter maintains a Redis cluster where every user will have a record ( logged in the last 30 days)
- (TweetId, UserId, Bits) will be inserted for every user record in Redis that follows me.
- When my follower requests the feed, Redis will have a record for that user which will be returned
- The timeline generated for my followers using Redis will not be stored on disk.
- Only my home timeline will be stored on disk. And if redis crashes, its a slow build out

![[Pasted image 20250323082719.png]]
Steps for searching tweets:
- Searching is not pre-computed
- Ingestor gets the tweet and does tokenization
- EarlyBird is a Lucene based Search Index (similar to ElasticSearch)
- The record is only sent to one EarlyBird instance
- When Blender receives a search query, the query is sent to all replicas because any of the Early Bird instance might have records
- Blender merges, deduplicates and ranks results and sends it back to the client

![[Pasted image 20250323083819.png]]

![[Pasted image 20250323084304.png]]

