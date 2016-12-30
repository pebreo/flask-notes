

Development
----------
```yaml
version: '2'
services:
  web:
    build: 
      context: ./web
      dockerfile: dev-Dockerfile 
    volumes:
      - ./web/web:/usr/src/app
      - data:/data  
    ports:
    - 8080:5000/tcp
    env_file: .env
    environment:
      DEBUG: 'true'
    command: /usr/local/bin/python /usr/src/app/app.py 

volumes:
  data: {}
```

Staging/Production
-----------------
```yaml
version: '2'
services:
  web:
    image: pebreo/dockerflask_web_staging:0.1.1
    depends_on:
      - redis
    stdin_open: true
    tty: true
    ports:
    - 80:5000/tcp
    env_file: .env
    environment:
      DEBUG: 'false'
    command: /usr/local/bin/gunicorn -w 2 -b 0.0.0.0:5000 app:app

  loggly:
    environment:
      TOKEN: 851e3b39-17ca-48b6-b7f1-edd1673b17a1
      TAG: Docker
    tty: true 
    image: sendgridlabs/loggly-docker
    ports:
     - "514:514/udp"

```