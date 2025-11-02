#### Manifest files
- Two most common formats:
	- **HLS (HTTP Live Streaming)** using `.m3u8` files
	- MPEG-DASH (Dynamic Adaptive Streaming over HTTP) using `.mpd` files.
##### HLS (HTTP Live Streaming) Manifest Example (`.m3u8`)
- HLS often uses two types of playlists: a Master Playlist and Media Playlists.
	- **Master Playlist:** Lists the different quality streams available.
	- **Media Playlist** (Example for `media/720p/stream.m3u8`): Lists the actual media segments for _one specific quality_ stream.
	
```#EXTM3U
#EXT-X-VERSION:3

# Describes the 480p stream variant
# BANDWIDTH=800000 (bits per second), RESOLUTION=854x480, CODECS="avc1.4d401f,mp4a.40.2"
# Points to the media playlist for this specific stream
#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=854x480,CODECS="avc1.4d401f,mp4a.40.2"
media/480p/stream.m3u8

# Describes the 720p stream variant
# BANDWIDTH=2500000, RESOLUTION=1280x720, CODECS="avc1.64001f,mp4a.40.2"
#EXT-X-STREAM-INF:BANDWIDTH=2500000,RESOLUTION=1280x720,CODECS="avc1.64001f,mp4a.40.2"
media/720p/stream.m3u8

# Describes the 1080p stream variant
# BANDWIDTH=5000000, RESOLUTION=1920x1080, CODECS="avc1.640028,mp4a.40.2"
#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080,CODECS="avc1.640028,mp4a.40.2"
media/1080p/stream.m3u8

# Fallback audio-only stream (example)
#EXT-X-STREAM-INF:BANDWIDTH=128000,CODECS="mp4a.40.2"
media/audio_only/stream.m3u8
```
##### MPEG-DASH Manifest Example (`.mpd`)
DASH uses an XML-based format called the Media Presentation Description (MPD).

```
<?xml version="1.0" encoding="UTF-8"?>
<MPD
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="urn:mpeg:dash:schema:mpd:2011"
    xsi:schemaLocation="urn:mpeg:dash:schema:mpd:2011 DASH-MPD.xsd"
    type="static"  mediaPresentationDuration="PT10M0S" minBufferTime="PT5S" profiles="urn:mpeg:dash:profile:isoff-on-demand:2011">

    <Period id="0" duration="PT10M0S">

        <AdaptationSet contentType="video" mimeType="video/mp4" segmentAlignment="true" startWithSAP="1">

            <Representation id="video_480p" codecs="avc1.4d401f" width="854" height="480" frameRate="30" bandwidth="800000">
                <SegmentTemplate timescale="90000" media="video/480p/segment_$Number$.m4s" initialization="video/480p/init.mp4" duration="900000"/>
                 </Representation>

            <Representation id="video_720p" codecs="avc1.64001f" width="1280" height="720" frameRate="30" bandwidth="2500000">
                <SegmentTemplate timescale="90000" media="video/720p/segment_$Number$.m4s" initialization="video/720p/init.mp4" duration="900000"/>
            </Representation>

            <Representation id="video_1080p" codecs="avc1.640028" width="1920" height="1080" frameRate="30" bandwidth="5000000">
                <SegmentTemplate timescale="90000" media="video/1080p/segment_$Number$.m4s" initialization="video/1080p/init.mp4" duration="900000"/>
            </Representation>
        </AdaptationSet>

        <AdaptationSet contentType="audio" mimeType="audio/mp4" segmentAlignment="true" startWithSAP="1" lang="en">

            <Representation id="audio_en" codecs="mp4a.40.2" audioSamplingRate="48000" bandwidth="128000">
                <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/> <SegmentTemplate timescale="48000" media="audio/en/segment_$Number$.m4s" initialization="audio/en/init.mp4" duration="480000"/>
                 </Representation>
        </AdaptationSet>

    </Period>
</MPD>
```
##### Difference in terms of manifest files
- In **DASH**, that file typically contains all the necessary descriptive information.
- In **HLS**, that initial file (the Master Playlist) provides the overview and pointers to other manifests (Media Playlists) that contain the detailed segment lists for each quality level.

#### Potential Deep Dives
- How can we handle processing a video to support adaptive bitrate streaming?
	- Split original file into segments (ffmpeg)
	- Transcode each segment and process other aspects of the segments
	- Create manifest files referencing the different segments in different video formats
	- Mark the upload as complete
- How do we support resumable uploads?
	- Client divides the file into chunks with a hash
	- Rest of the stuff is similar to Dropbox
- Scaling each component
	- Video Service - Stateless
	- Video Metadata - Cassandra DB 
	- Video Processing Service - Stateless as well as queue
	- S3 - Infinite scale
