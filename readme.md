# vosk-arm

https://github.com/alphacep/vosk-server

## Docker Compose File

- Create `docker-compose.yml`:
  
  ```yaml
  version: '3.6'
  
  services:
    vosk:
      image: uburuntu/kaldi-vosk-server:arm64
      container_name: vosk
      ports:
        - 2700:2700
      restart: always
  ```

- Run:
  
  `docker-compose up`