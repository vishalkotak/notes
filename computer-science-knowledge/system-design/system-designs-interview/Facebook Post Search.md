##### Functional Requirements
1. Users should be able to create and like posts.
2. Users should be able to search posts by keyword.
3. Users should be able to get search results sorted by recency or like count.
##### Non Functional Requirements
1. The system must be fast, median queries should return in < 500ms.
2. The system must support a high volume of requests (we'll estimate this later).
3. New posts must be searchable in a short amount of time, < 1 minute.
4. All posts must be discoverable, including old or unpopular posts. (we can take more time for these)
5. The system should be highly available.

![[Pasted image 20250411204708.png]]
##### Deep Dives
1. How can we handle multi-keyword, phrase queries?
	- Good Solution: Intersection and Filter
	- Great Solution: Bigrams and Shingles
2. How can we address the large volume of writes?
	- Batch likes before writing to the database
	- Two stage architecture