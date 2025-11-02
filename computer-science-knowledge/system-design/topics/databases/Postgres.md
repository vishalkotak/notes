### pg_trgm
`pg_trgm` is a powerful PostgreSQL extension included in the standard distribution (`contrib` modules). It provides functions and operators to determine the **similarity** of text based on **trigram** matching.
#### What's a Trigram?
A trigram is simply a sequence of **three consecutive characters** taken from a string. To handle the beginning and end of strings consistently, `pg_trgm` typically adds two spaces before and one space after the text before generating trigrams.

**Example:** The string "hello" would be treated as " hello " and broken down into these trigrams:

- " h"
- " he"
- "hel"
- "ell"
- "llo"
- "lo "
#### How Similarity Works
- Generating the set of trigrams for each string.
- Counting how many trigrams are **shared** between the two sets.
- Calculating a similarity score, typically by dividing the number of shared trigrams by the number of trigrams in the longer string (or sometimes the average, depending on the specific function/operator). A score of 1.0 means identical trigram sets (very similar or identical strings), and 0.0 means no shared trigrams.
#### Core Components
##### Functions
- `similarity(text, text)`: Returns a score (0 to 1) indicating how similar the two strings are.
- `show_trgm(text)`: Returns an array of all trigrams for the given text (useful for understanding).
##### Operators
- - `%` (Similarity): `text % text` returns `true` if the similarity between the two strings is greater than the current similarity threshold (default 0.3).
- `<->` (Distance): `text <-> text` returns the "distance" between the strings (0 for identical, 1 for completely dissimilar). It's calculated as `1 - similarity`. Useful for finding the _closest_ matches.
-  `<%` / `%>` (Word Similarity): Similar to `%` but based on shared trigrams between _words_ rather than the whole string. `word <% text` checks if `word` is similar to any word in `text`. `text %> word` is the reverse.
- `<<->` / `<->>` / `<<<->>>` (Word Distance): Distance operators corresponding to word similarity.
##### Index Support: 
`pg_trgm` offers GiST (Generalized Search Tree) and GIN (Generalized Inverted Index) index support. **This is crucial for performance!** Searching large tables without an index using these operators will be very slow.

- `gist_trgm_ops`: Operator class for GiST indexes. Generally good for `<->` (nearest neighbor) searches.
- `gin_trgm_ops`: Operator class for GIN indexes. Generally faster for `%` (similarity threshold) searches.

```
-- Ensure the extension is created in our database
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Create a sample table to store text data
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT
);

-- Insert some sample data with variations and potential typos

-- Create a GIN index on the 'title' column using gin_trgm_ops
-- GIN is often preferred for similarity (%) searches
CREATE INDEX idx_gin_articles_title ON articles USING gin (title gin_trgm_ops);

-- Create a GiST index on the 'content' column using gist_trgm_ops
-- GiST is often preferred for distance (<->) searches (nearest-neighbor)
CREATE INDEX idx_gist_articles_content ON articles USING gist (content gist_trgm_ops);

-- You can verify the extension is active
-- \dx pg_trgm

-- You can see the trigrams generated for a string
-- SELECT show_trgm('PostgreSQL');
-- SELECT show_trgm('Postgres');
```

```
-- Show trigrams for different strings
SELECT show_trgm('PostgreSQL');
SELECT show_trgm('Postgres');
SELECT show_trgm('postgressql'); -- Notice the typo trigrams

-- Calculate similarity score between strings
SELECT similarity('PostgreSQL', 'Postgres');
SELECT similarity('PostgreSQL', 'postgressql'); -- Similarity despite typo
SELECT similarity('PostgreSQL', 'MySQL');      -- Low similarity

-- Set a similarity threshold for the current session
SET pg_trgm.similarity_threshold = 0.3; -- Default is 0.3, let's try 0.2 for broader matches

-- Find articles with titles SIMILAR TO 'Postgres' using the % operator
-- This query will efficiently use the GIN index idx_gin_articles_title
SELECT id, title, similarity(title, 'Postgres') AS score
FROM articles
WHERE title % 'Postgres'
ORDER BY score DESC;

-- Try searching with a typo
SELECT id, title, similarity(title, 'Postgess') AS score
FROM articles
WHERE title % 'Postgess' -- Find titles similar to 'Postgess'
ORDER BY score DESC;

-- Find articles with content CLOSEST TO 'database relational open source' using the <-> operator
-- This query will efficiently use the GiST index idx_gist_articles_content
SELECT id, left(content, 50) AS content_start, content <-> 'database relational open source' AS distance
FROM articles
ORDER BY content <-> 'database relational open source'
LIMIT 5;

-- Exit psql
\q
```