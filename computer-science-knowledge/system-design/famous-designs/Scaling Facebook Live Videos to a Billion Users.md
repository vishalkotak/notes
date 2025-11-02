Reference: https://www.youtube.com/watch?v=IO4teCbHvZw

![[Pasted image 20250323080104.png]]
#### Path from Broadcasting client to the data center

Proxygen hosts accept incoming connections from POP and use consistent hashing on the stream id to find the right Encoding hosts. If there is a connection loss, the next proxygen hosts can connect to the same encoding server.

![[Pasted image 20250323080202.png]]

![[Pasted image 20250323080134.png]]
#### Broadcast receiving side

![[Pasted image 20250323080420.png]]

![[Pasted image 20250323075644.png]]
#### Gemini Generated Summary

Based on the information available, including insights from the provided YouTube video and related sources, here's a breakdown of how Facebook handles live broadcasting:

**1. Ingesting the Live Stream:**

- **Broadcast Client:**
    - The process begins with the user's device (mobile or desktop) acting as the broadcast client.
    - This client captures the video and audio, and then transmits it to Facebook's infrastructure.
    - Facebook utilizes RTMPS (a secure version of RTMP) for this initial transmission, chosen for its efficiency in handling real-time video streams.
- **Points of Presence (POPs):**
    - Facebook has strategically placed POPs around the globe.
    - These POPs act as the first point of contact for incoming live streams, handling the initial connection and routing.
    - The POPs help to reduce latency by ensuring that streams are ingested as close to the source as possible.

**2. Processing and Encoding:**

- **Data Centers:**
    - From the POPs, the streams are forwarded to Facebook's data centers.
    - Here, the video is processed and encoded into multiple bitrates.
    - This adaptive bitrate streaming is crucial for ensuring a smooth viewing experience, as it allows the playback quality to adjust based on the viewer's internet connection.
- **Encoding:**
    - The encoding process breaks the live stream into small, manageable segments.
    - These segments are then stored, ready for distribution to viewers.

**3. Distributing the Live Stream:**

- **DASH (Dynamic Adaptive Streaming over HTTP):**
    - For playback, Facebook uses DASH.
    - DASH allows for efficient delivery of the segmented video, enabling viewers to receive the optimal video quality based on their bandwidth.
- **Caching:**
    - Facebook employs extensive caching at both the data center and POP levels.
    - This caching helps to reduce the load on the servers and ensures that viewers can receive the stream with minimal delay.
    - This is very important for handling the "Thundering Herd" problem, that occurs when very large numbers of viewers join a live stream at the same time.
- **Playback Client:**
    - The viewer's device acts as the playback client, recieving the video segments and reconstructing the live stream.

**Key Technical Considerations:**

- **Low Latency:**
    - Minimizing latency is a critical challenge in live streaming.
    - Facebook addresses this through its distributed infrastructure and efficient protocols.
- **Scalability:**
    - Handling massive concurrent viewers requires a highly scalable system.
    - Facebook's architecture is designed to handle sudden spikes in viewership.
- **Reliability:**
    - Ensuring a reliable live streaming experience is paramount.
    - Facebook uses redundant systems and robust error handling to minimize disruptions.

In essence, Facebook's live broadcasting infrastructure is a complex, globally distributed system designed to handle the challenges of real-time video delivery at massive scale.