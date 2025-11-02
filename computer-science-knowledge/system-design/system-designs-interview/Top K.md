- https://www.youtube.com/watch?v=1lfktgZ9Eeo

![[Pasted image 20250318213118.png]]

- https://www.youtube.com/watch?v=nQpkRONzEQI&t=116s

Functional Requirements
- All user facing operations should be as fast as possible
	- Publishing events
	- Reading the leaderboard
- Note that getting exact results can make fetching the leaderboard slow
	- We have a tradeoff between:
		- Fast Reads
		- Precise Results

#### Top K songs/videos per user
- Redis can be utilized to store the song id for every user
- Key: user:songs:played:123
- Each entry in a ZSET consumes memory for the member (song ID string) and the score (integer), plus some overhead.
- Per User Record Size: 50-70 KB
- **10,000 users:** ~500-700 MB of RAM
- **100,000 users:** ~5-7 GB of RAM
- **1,000,000 users:** ~50-70 GB of RAM

```
ZINCRBY user:songs:played:123 1 song:beatit

ZINCRBY user:songs:played:123 1 song:thriller

ZINCRBY user:songs:played:123 1 song:thriller

# Give me the top 3 songs for the user
ZREVRANGE user:songs:played:123 0 2
```
	