# Wowza Streaming Engine
## Set up
Default port  | Usage
------------- | -----
TCP 1935      | RTMP/RTMPE/RTMPT/RTSP-interleaved streaming and WOWZ™ streaming
TCP 8086-8088 | Administration
UDP 6970-9999 | RTP UDP streaming
TCP 80        | HLS, MPEG-DASH, and RTMPT streaming
TCP 443       | SSL/TLS streaming (RTMPS and HTTPS)
TCP 554       | RTSP streaming

Note that all ports can be customized in Server.xml and VHost.xml. For instance, KOC Customize setting (8086 - 8088 --> 8286 - 8288):
- http://localhost:8286/connectioncounts
- http://localhost:8286/serverinfo
- http://localhost:8287/ (version info)
- http://localhost:8288/ (manager page)

Ref to create a server: https://www.wowza.com/docs/how-to-get-started-as-a-wowza-streaming-engine-manager-administrator#create-applications

Ref to set up Video On Demand (VOD) streaming: https://www.wowza.com/docs/how-to-set-up-video-on-demand-streaming#configure-playback

## Config
Working dir: `/usr/local/WowzaStreamingEngine`
- admin password: `conf/admin.password`
  - only username and password in admin.password can have admin right, eg restart/edit in management page
  - can also set in docker's env_file
- Allow show connection count / server info: `conf/VHost.xml`
  - `com.wowza.wms.http.HTTPConnectionCountsXML`
  - change the `AuthenticationMethod` from `admin-basic` to `none` to disable error 401

## Play video
Shortcut:
- http://localhost:10280/vod/sample.mp4/playlist.m3u8
- http://localhost:10380/vod/sample.mp4/playlist.m3u8
- http://localhost:10480/vod/sample.mp4/playlist.m3u8

Mobile HLS:
- M3U8 is a computer file format that contains a multimedia playlist. Normally the playlist points to a stream on the Internet.
- HLS stands for HTTP Live Streaming. It's a HTTP-based media streaming communications protocol implemented by Apple Inc. It works by breaking the overall stream into a sequence of small file downloads, each download loading one short chunk of an overall potentially unbounded transport stream. HLS streams are usually described using M3U8 playlist.
- After sending the first HLS request (http://localhost:10280/vod/sample.mp4/playlist.m3u8), it will return a response of `Content-type: application/vnd.apple.mpegurl` containing a chunklist_xxx.m3u8. *Each new connection will have a new chunklist title.*
  ```
  #EXTM3U
  #EXT-X-VERSION:3
  #EXT-X-STREAM-INF:BANDWIDTH=515781,CODECS="avc1.4d401e,mp4a.40.2",RESOLUTION=640x360
  chunklist_w2003238606.m3u8

  // Each new connection will have a new chunklist title
  // The following chunklists point to the same video source
  chunklist_w2014397019.m3u8
  chunklist_w1500485781.m3u8
  ```
- Then, it sends the second HLS request (http://localhost:10280/vod/sample.mp4/chunklist_w2003238606.m3u8), which will return a chunklist response of `Content-type: application/vnd.apple.mpegurl`. In this example, the `sample.mp4` duration is 41:50.
  ```
  #EXTM3U                    # m3u8 declaration, must on first line
  #EXT-X-VERSION:3
  #EXT-X-TARGETDURATION:14   # Max duration for each segment (sec)
  #EXT-X-MEDIA-SEQUENCE:0    # The sequence number of the first segment
  #EXTINF:10.0,              
  media_w2003238606_0.ts
  #EXTINF:13.68,             # Duration for that segment
  media_w2003238606_1.ts     # Title for that segment
  #EXTINF:14.0,    
  media_w2003238606_2.ts

  (3.ts to 176.ts)

  #EXTINF:14.0,    
  media_w2003238606_178.ts
  #EXTINF:8.745,   
  media_w2003238606_179.ts
  #EXT-X-ENDLIST
  ```
- Then, according to the list of segment, it starts sending a series of HLS request (for example, starting from http://localhost:10280/vod/sample.mp4/media_w2003238606_0.ts to http://localhost:10280/vod/sample.mp4/media_w2003238606_179.ts), which will return a response of `Content-Type: video/MP2T`

MPEG Dash (Video test player):
- URL: http://hktv.edge01.com/vod/sample.mp4/manifest.mpd

Video Testing:
- official video test player: https://www.wowza.com/testplayers
- google chrome: `play HLS` extension
- firefox: `Native MPEG-Dash + HLS Playback` extension

## Load balancing
Ref: https://www.wowza.com/docs/how-to-get-dynamic-load-balancing-addon
```
Both server/client model in Live/KOC
                                              |--> hktv.edge01.com (loal balancing client)
request --> hktv.edge01.com (load balancer) --|--> hktv.edge02.com (loal balancing client)
        |                                     |--> hktv.edge03.com (loal balancing client)
        |
        |                                     |--> hktv.edge01.com (loal balancing client)
        --> hktv.edge02.com (load balancer) --|--> hktv.edge02.com (loal balancing client)
        |                                     |--> hktv.edge03.com (loal balancing client)
        |
        |                                     |--> hktv.edge01.com (loal balancing client)
        --> hktv.edge03.com (load balancer) --|--> hktv.edge02.com (loal balancing client)
                                              |--> hktv.edge03.com (loal balancing client)

```
### Config
- Load balancer server only need to set its own listen IP and port. **No need set load balancing clients' IPs, ports.**
- Load balancing clients need to set **which load balancer can redirect to them.**
```xml
<Properties>
  <!-- Load balancer server is Server, loal balancing client is Client -->
  <Property>
    <Name>loadbalanceType</Name>
    <Value>Server,Client</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceKey</Name>
    <Value>0123456789012345</Value>
    <Type>String</Type>
  </Property>
  <!-- Load balancer server config -->
  <Property>
    <Name>loadbalanceServerListenIP</Name>
    <Value>172.16.213.42</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceServerListenPort</Name>
    <Value>80</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceServerDecisionOrder</Name>
    <Value>Connection,Bandwidth</Value>
    <Type>String</Type>
  </Property>
  <!-- Load balancing client config -->
  <Property>
    <!-- these load balancers can redirect to me -->
    <Name>loadbalanceServerIP</Name>
    <Value>hktv.edge01.com,hktv.edge02.com,hktv.edge03.com</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceServerPort</Name>
    <Value>80</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceClientCommunicationScheme</Name>
    <Value>http</Value>
    <Type>String</Type>
  </Property>
  <!-- Name of the client shown in the /loadbalanceinfo page -->
  <Property>
    <Name>loadbalanceClientName</Name>
    <Value>Edge_01</Value>
    <Type>String</Type>
  </Property>
  <!-- which client is redirected in case this client is used -->
  <Property>
    <Name>loadbalanceClientForceIP</Name>
    <Value>hktv.edge01.com</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceClientForcePort</Name>
    <Value>80</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceClientForceScheme</Name>
    <Value>http</Value>
    <Type>String</Type>
  </Property>
  <!-- For connection balancing -->
  <Property>
    <Name>loadbalanceClientConnectionEnable</Name>
    <Value>On</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceClientConnectionLimit</Name>
    <Value>0</Value> <!-- 0 is unlimited -->
    <Type>String</Type>
  </Property>
  <!-- For bandwidth balancing -->
  <Property>
    <Name>loadbalanceClientBandwidthEnable</Name>
    <Value>On</Value>
    <Type>String</Type>
  </Property>
  <Property>
    <Name>loadbalanceClientBandwidthLimit</Name>
    <Value>0</Value> <!-- 0 is unlimited kbps -->
    <Type>String</Type>
  </Property>
  <Property>
    <Name>MediaCacheDebugLog</Name>
    <Value>true</Value>
    <Type>Boolean</Type>
  </Property>
</Properties>
```

### Usage and Testing
- (LOCAL) Fiddler setting because of using docker containers (only for Firefox)
  ```
  localhost:10280 hktv.edge01.com
  localhost:10380 hktv.edge02.com
  localhost:10480 hktv.edge03.com
  ```
- After access a load balancer server via a `/redirect` URL:
  - http://hktv.edge01.com/redirect/vod/sample.mp4?type=m3u8
  - http://hktv.edge02.com/redirect/vod/sample.mp4?type=m3u8
  - http://hktv.edge03.com/redirect/vod/sample.mp4?type=m3u8
- it will redirect to one of the load balancing servers:
    - http://hktv.edge01.com/vod/sample.mp4/playlist.m3u8
    - http://hktv.edge02.com/vod/sample.mp4/playlist.m3u8
    - http://hktv.edge03.com/vod/sample.mp4/playlist.m3u8
- Load balance info: http://hktv.edge01.com/loadbalanceinfo

### Potential Problem
- Need to check if the load balancer server can connect to each load balancing server, sometimes may be blocked by firewall. One checking method can be using `nc -v 192.168.0.175 80`.

## Appendix
### Docker file
```yml
##### docker-compose.yml -----------------------------
  hktv_wowza_edge_1:
    build:
      context: ./edge-01-context
      dockerfile: Dockerfile-edge
      args:
          - IMAGE_TAG=$IMAGE_TAG
          - TRANSCODER_SSH_KEYSTORE_FILE=$TRANSCODER_SSH_KEYSTORE_FILE
    hostname: hktv.edge01.com
    env_file:
      - .env.edge01
    working_dir: /usr/local/WowzaStreamingEngine
    ports:
      - "10280:80"
      - "12443:443"
      - "1937:1935"
      - "8286:8086"
      - "8287:8087"
      - "8288:8088"
      - "8289:8089"
    networks:
      hktv_wowza_system_network:
        ipv4_address: 172.16.213.42
    container_name: hktv_wowza_edge_1

##### .env.edge01 -----------------------------
WSE_MGR_USER=admin
WSE_MGR_PASS=admin

# important, if expired, cannot use any service
WSE_LIC=ET1B4-Nf8Jn-XKzDk-df7NC-HJJEN-ydkRp-69MH9urfeUpP

IMAGE_TAG=latest

loadbalanceType=Server,Client
loadbalanceServerListenPort=443
loadbalanceServerIP=172.16.213.42,172.16.213.43,172.16.213.44
loadbalanceServerPort=443
loadbalanceClientCommunicationScheme=https
loadbalanceServerDecisionOrder=Connection,Bandwidth
loadbalanceKey=0123456789012345
loadbalanceClientForceScheme=https
loadbalanceClientConnectionEnable=On
loadbalanceClientConnectionLimit=0
loadbalanceClientBandwidthEnable=On
loadbalanceClientBandwidthLimit=0

loadbalanceServerListenIP=172.16.213.42
loadbalanceClientName=Edge_01
loadbalanceClientForceIP=172.16.213.42
loadbalanceClientForcePort=443

##### Dockerfile-edge -----------------------------
ARG IMAGE_TAG
ARG TRANSCODER_SSH_KEYSTORE_FILE

FROM wowzamedia/wowza-streaming-engine-linux:$IMAGE_TAG

COPY conf/$TRANSCODER_SSH_KEYSTORE_FILE /usr/local/WowzaStreamingEngine/conf/$TRANSCODER_SSH_KEYSTORE_FILE
COPY conf/VHost.xml /usr/local/WowzaStreamingEngine/conf/VHost.xml
COPY conf/Server.xml /usr/local/WowzaStreamingEngine/conf/Server.xml
COPY conf/MediaCache.xml /usr/local/WowzaStreamingEngine/conf/MediaCache.xml
COPY conf/live/Application.xml /usr/local/WowzaStreamingEngine/conf/live/Application.xml
COPY conf/koc/Application.xml /usr/local/WowzaStreamingEngine/conf/koc/Application.xml
COPY lib/* /usr/local/WowzaStreamingEngine/lib

EXPOSE 80 443 1935 8086 8087 8088

ENTRYPOINT ["/sbin/entrypoint.sh"]
```