# ring-hassio

[Buy Me A Coffee ☕️](https://www.buymeacoffee.com/romanost)

## About
This is a fork of https://github.com/jeroenterheerdt/ring-hassio

A Home Assistant add-on to enable live streams of Ring Cameras.
This add-on wraps around [Dgreif's excellent work](https://github.com/dgreif/ring) and exposes a livestream.
Supervisor not required to run this addon

> **You can run this docker 24/7, the streaming will start by API command

## Installation
1. Clone this GitHub repository or copy and adjust the exmaples from this page
2. Configure your Ring Refresh Token and port (see configuration below).
3. Adjust docker-compose.yaml
4. Start the docker
5. For remote access, open up the ports in your router.
6. Start the app by accessing: http://homeassistant.local:3001/api/start
7. Open the stream at http://homeassistant.local:port/public/stream.m3u8 to make sure it works before going any further. We recommend using VLC or equivalent.
8. Add a camera to Home Assistant, such as:
   ```yaml
   camera:
     - platform: generic
       name: Ring Livestream
       stream_source: http://homeassistant.local:port/public/stream.m3u8
       still_image_url: http://homeassistant.local:port/public/stream.m3u8
    ```
    (Don't worry about the `still_image_url` not pointing to an actual image, we are not going to use it, but it is required.)
9. Add a card to your UI
10. Done! Enjoy your shiny new livestream!

## Configuration
Example configuration:
```yaml
ring_refresh_token: your_refresh_token
camera_name: Front Door
```
* You need to create a refresh token - see https://github.com/dgreif/ring/wiki/Refresh-Tokens on how to do that. Note that you will have to have node and npm installed on your machine.
* The camera name is the name entred when setting up the camera in the Ring app.

## Taking a snapshot
Currently the addon does not support taking snapshots, but when it does this is the configuration you will need:
In order to use the `snapshot` service, you will need to following settings in your `configuration.yaml`:
   ```yaml
   homeassistant:
     whitelist_external_dirs:
       - /config/tmp
   ```
   You can then call the `snapshot` service like this:
   ```yaml
   service: camera.snapshot
   entity_id: [entityID]
   filename: tmp/foo.jpg
   ```

Some users reported success to create a snapshot using:
```camera:
  - platform: ffmpeg
    input: http://hassio.local:port/public/stream.m3u8
```

## Battery conservation
The streaming app will stop automatically after 180 seconds.
To start the app browse to: http://homeassistant.local:3001/api/start

## Docker-compose example
```yaml
  ringhassio:
    restart: always
    container_name: ringhassio
    hostname: ringhassio
    image: romanost/ring-hassio:latest
    network_mode: "bridge"
    ports:
      - 3000:3000 ## port configured in conf for streaming
      - 3001:3001 ## api port - static
    environment:
      TZ: "Asia/Jerusalem"
    volumes:
      - /ring-hassio/config:/data ## options.json file location
```

## Config file example

/ring-hassio/config/options.json
```json
{
    "ring_refresh_token": "enter_token_here",
    "camera_name": "Front Door",
    "port": 3000
}
```

## Automation example
Shell command for starting the app:
```yaml
shell_command:
  mngdocker: 'curl localhost:3001/api/start'
```

Automation with popup
```yaml
- id: '1610000000'
  alias: Ringing
  trigger:
  - platform: state
    entity_id: binary_sensor.front_door_ding
    from: 'off'
    to: 'on'
  action:
  - service: shell_command.mngdocker
  - service: browser_mod.popup
    data:
      large: true
      # hide_header: true
      title: Popup example
      card:
        type: picture-entity
        entity: camera.ring_livestream
        camera_view: live
      deviceID:
        - fafa3454-7788abcd
        - 9f443322-abab3331
  mode: single
```


