# RTMP Go Away

[Real Time Messaging Protocol](https://www.adobe.com/content/dam/acom/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf) (RTMP) is a popular media streaming protocol that uses TCP persistent connections. When a connection between a live streaming client and the platform is interrupted, data from the live event is lost until the client can reconnect to a new server. **RTMP Go Away** is a new mechanism that allows for the live server to send a signal to the client indicating that it needs to terminate the existing connection. This allows the client to create a new connection at a logical media boundary, incurring zero data loss.

The purpose of this repository is to provide the RTMP Specifications required to implement this feature in both streaming clients and live servers. Additionally, a reference client implementation in [FFmpeg](https://www.ffmpeg.org/) is provided.

# Design Overview

1. The streaming client will announce to the server that it supports the new feature upon connection. This will be done by adding a field to the RTMP connect packet object. The live server will parse this new field on connection and save the result for later use.
2. Before the live server shuts down (e.g. for maintenance) it sends a new RTMP Go Away packet to the client. The live server should only send the Go Away packet to clients that support the feature.
3. Upon receiving the Go Away packet, the streaming client will wait until the next logical media boundry (e.g. IDR frame), disconnect and create a new connection to a new server. Once a new connection is established, the client can resume streaming from the logical boundry without experiencing any data loss.

# RTMP Specification

The RTMP connect packet will indicate support of the feature with new field `supportsGoAway` set to true.

```json
NetConnection Command {
   "CommandName": "connect",
   "CommandObject": {
     "flashVer": "...",
     "supportsGoAway": 1
   }
}
```

Before the live server shuts down it sends a new RTMP Go Away packet to the client using `messageTypeId` 32.

```json
Protocol Control Message {
   "messageTypeId": 32
}
```

# Reference Implementation

A working client implementation in FFmpeg is provided [here](https://patchwork.ffmpeg.org/project/ffmpeg/patch/20210926205201.1163-1-jordi.cenzano@gmail.com/) as reference. This patch has been submitted to FFmpeg for consideration and review.

# License

RTMP Go Awat is licensed under the [MIT license](LICENSE).
