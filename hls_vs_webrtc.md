# HLS
HLS is created by Apple to solve the problems with the conventional web video streaming issues.
* Unable to adapt to the client resolution/network constraints
* Have to have a server deamon to host, resulting in regional servers to scale globally
* Blocked by most firewalls as they supports HTTP and Email protocols only

So Apple comes up with the idea of HTTP Live Streaming protocol to tackle these problems. The advantages it promotes are as follows. ([from official site](https://developer.apple.com/documentation/http_live_streaming))
* Live broadcasts and prerecorded content (video on demand, or VOD)
* Multiple alternate streams at different bit rates
* Intelligent switching of streams in response to network bandwidth changes
* Media encryption and user authentication

To conclude, live/prerecorded support, multi-rate support, bandwidth adaptive, and encryption/auth.

## HLS in 2 minutes
`Adaptive Live Streaming` is the idea to dynamically provide streaming depending on the network condition and screen resolution. In order to do that, chunks of videos of different resolution and rate is prepared before hand. Client decides which video to request. Because the chunks are designed to be small(seconds length of content), the switch between different video would be instant.

The main idea is abstracted as indexing of chunks of videos. The M3U8 files are used to describe the indexing.

The conventional M3U8 (UTF-8 encoded text file) looks as follows.
640x360_1200.m3u8
```m3u8
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:11
#EXTINF:5.215111,
00000.ts
#EXTINF:10.344822,
00001.ts
#EXTINF:10.344822,
00002.ts
#EXTINF:9.310344,
00003.ts
#EXTINF:10.344822,
00004.ts
...
#EXT-X-ENDLIST
```
First 5 lines are metadata for this index file. Every 2 lines afterwards are the real indexing.
```m3u8
#EXTINF:5.215111,
00000.ts
```
It means the `00000.ts` file contains `5.215111` seconds of video. So in runtime, these files would be downloaded one by one.

`Adaptive Live Streaming` offers two layers of indexing. Let's take a look at the following 

adaptive_live_streaming_master.m3u8
```
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=1296,RESOLUTION=640x360
https://.../640x360_1200.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=264,RESOLUTION=416x234
https://.../416x234_200.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=464,RESOLUTION=480x270
https://.../480x270_400.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1628,RESOLUTION=960x540
https://.../960x540_1500.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2628,RESOLUTION=1280x720
https://.../1280x720_2500.m3u8
```
The master indexing file points to different media indexing files that contains indexing to the real video chunks. *640x360_1200.m3u8* can be the example we listed above.

HLS client starts with the first media indexing file and changes to the different indexing file depending on the network bandwidth with ensuring the resolution makes sense.

One easy optimization is to prepare different master indexing file depending on the client request, in which client network status could be included, so that client starts with the one that make most sense.

The conventional way to store video content is through the MPEG-2 TS files (.ts) and encoded with the H.264 format with audio in MP3, HE-AAC, or AC-3. The nature with the video formatting is not all the frames are complete images, it is just diffs added for each consequent frame. Usually only the first frame of our ts file is complete image. If we jump to 3s, the diffs of the 3s need to be calculate to produce the complete image of the 3s-image. A solution to this is to provide multiple key frames along the whole video streaming.

A handy tag to use for switching configuration of the m3u8 playlist is `#EXT-X-DISCONTINUITY`, which tells the player "recalculate everything from this point". It is useful you want to have some customization for the specific user.

Above focuses on the video on demond topic. Live streaming is a big topic. The general solution to this is to have new .ts files appended to the m3u8 file. Every time new batch of .ts files are added, a `#EXT-X-MEDIA-SEQUENCE:<counter>` will be added before the new list. `counter` starts from 1, which is the original media indexing file. `#EXT-X-ENDLIST` must not be added for this case. Client keeps refreshing the indexing file, so make sure the indexing file is served with the no-cache headers.

# References
[Introduction to HTTP Live Streaming: HLS on Android and More, from *TOMO KRAJINA*](https://www.toptal.com/apple/introduction-to-http-live-streaming-hls)