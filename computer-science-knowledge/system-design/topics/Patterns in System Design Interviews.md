#### Simple DB backed CRUD service with caching
![[Pasted image 20250411132103.png]]
#### Async Job worker pool
- System needs to handle job processing and can tolerate some delay, you might use an async job worker pool
- Examples:
	- Social Network needs to process lot of images and videos
- Technologies used:
	- SQS + EC2 (Auto Scaling) + Visibility timeout
	- Kafka
#### Two Stage Architecture
- Scaling an algorithm with poor performance characteristics
- Consider a scenario requiring comparison of image against all the images available. First stage will remove the vast majority of dissimilar images and in the second stage, we will find the match
![[Pasted image 20250411132658.png]]
#### Event Driven Architecture
- Build highly scalable, loosely coupled and responsive services
- Mostly using Kafka
#### Durable Job Processing
- Workers need to report status about the jobs they are executing
- Temporal
#### Proximity-Based Services
- Technologies used: Postgres with PostGIS extension, Redis geospatial data type