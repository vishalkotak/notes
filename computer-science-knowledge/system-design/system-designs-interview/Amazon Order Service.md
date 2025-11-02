Reference: https://drive.google.com/file/d/1IrVFbAosM8fcqJU4IUL-Zl3_8oN2ZrGY/view

The below design focuses on order service retrieval based on ETL and CDC. 
- User searches for a OrderId
- We check the hot transactional database for details
	- YES: Return to the user
	- NO: Check in WARM/COLD storage
- Databases send data to the staging area (S3) using CDC mechanisms
- Spark jobs read the data from S3 and write it to WARM storage

![[Pasted image 20250406193235.png]]

