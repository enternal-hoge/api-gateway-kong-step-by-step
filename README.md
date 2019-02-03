# api-gateway-kong-step-by-step
kong 1.0 installation , add service, add route procedure.

# Notice
This Procedure is umcomplete and may be not go on.

## Install docker-ce
```
# apt-get install apt-transport-https ca-certificates curl software-properties-common
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# apt-key fingerprint 0EBFCD88
# add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# apt-get update
# apt-get install docker-ce
```

fefer : https://docs.konghq.com/install/docker/

## Install kong
```
# docker network create kong-net
# docker run -d --name kong-database \
                --network=kong-net \
                -p 5432:5432 \
                -e "POSTGRES_USER=kong" \
                -e "POSTGRES_DB=kong" \
                postgres:9.6

# docker run --rm \
      --network=kong-net \
      -e "KONG_DATABASE=postgres" \
      -e "KONG_PG_HOST=kong-database" \
      -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
      kong:latest kong migrations bootstrap

# docker run -d --name kong \
      --network=kong-net \
      -e "KONG_DATABASE=postgres" \
      -e "KONG_PG_HOST=kong-database" \
      -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
      -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
      -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
      -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
      -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
      -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
      -p 8000:8000 \
      -p 8443:8443 \
      -p 8001:8001 \
      -p 8444:8444 \
      kong:latest

# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
8f45eb54d43a        kong:latest         "/docker-entrypoint.…"   12 seconds ago      Up 11 seconds       0.0.0.0:8000-8001->8000-8001/tcp, 0.0.0.0:8443-8444->8443-8444/tcp   kong
3016b9754ea5        postgres:9.6        "docker-entrypoint.s…"   3 minutes ago       Up 2 minutes        0.0.0.0:5432->5432/tcp                                               kong-database

# curl -i http://localhost:8001/
HTTP/1.1 200 OK
Date: Sun, 03 Feb 2019 05:44:29 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/1.0.3
Content-Length: 5651

# curl -i http://localhost:8001/apis/
HTTP/1.1 404 Not Found
Date: Sun, 03 Feb 2019 05:46:03 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/1.0.3
Content-Length: 23
```

## Ceate Sample Application
```
# apt install python-pip
# pip install flask
# mkdir -p /opt/service
# touch /opt/service/page.py
# chmod 755 /opt/service/page.py
# vi /opt/service/page.py
# cat /opt/service/page.py
# coding: utf-8

from flask import Flask
app = Flask(__name__)

@app.route('/page')
def hello_world():
    return '{ message: "page" }'

@app.route('/ja')
def hello_world_ja():
    return '{ message: "ぱげ"}'

@app.route('/ping')
def ping():
    return '{ result: "ok"}'

if __name__ == '__main__':
    app.run(debug=False, host='0.0.0.0', port=5000)
    
# python /opt/service/page.py
```


## Register Service

execute kong rest api
```
# curl -X POST -H 'Content-Type:application/json' \
 -d '{"name": "page-service", "retries": 5, "protocol": "http", "host": "localhost", "port": 5000, "path": "/page"}' \
 http://localhost:8001/services/
{"host":"localhost","created_at":1549174160,"connect_timeout":60000,"id":"8855bebd-a9d9-4f6f-a8cb-1debd31e622f","protocol":"http","name":"page-service","read_timeout":60000,"port":5000,"path":"\/page","updated_at":1549174160,"retries":5,"write_timeout":60000}root@kong:~#
```

confirm
```
root@kong:~# curl -X GET http://localhost:8001/services/
{"next":null,"data":[{"host":"localhost","created_at":1549174160,"connect_timeout":60000,"id":"8855bebd-a9d9-4f6f-a8cb-1debd31e622f","protocol":"http","name":"page-service","read_timeout":60000,"port":5000,"path":"\/page","updated_at":1549174160,"retries":5,"write_timeout":60000}]}root@kong:~#
```

## Register Route

execute kong rest api
```
root@kong:~# curl -i -X POST -H 'Content-Type:application/json' \
>  -d '{"name": "page-route", "protocols": ["http"], "hosts": ["localhost"], "paths": ["/page"], "regex_priority": 0, "strip_path": true, "preserve_host": false, "service": {"id":"8855bebd-a9d9-4f6f-a8cb-1debd31e622f"}}' \
>  http://localhost:8001/services/page-service/routes
HTTP/1.1 201 Created
Date: Sun, 03 Feb 2019 06:23:41 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/1.0.3
Content-Length: 352

{"created_at":1549175021,"methods":null,"id":"99b17157-2dff-4fad-ae85-449de46a5991","service":{"id":"8855bebd-a9d9-4f6f-a8cb-1debd31e622f"},"name":"page-route","hosts":["localhost"],"updated_at":1549175021,"preserve_host":false,"regex_priority":0,"paths":["\/page"],"sources":null,"destinations":null,"snis":null,"protocols":["http"],"strip_path":true}root@kong:~#
```

confirm
```
root@kong:~# curl -X GET http://localhost:8001/routes
{"next":null,"data":[{"created_at":1549175021,"methods":null,"id":"99b17157-2dff-4fad-ae85-449de46a5991","service":{"id":"8855bebd-a9d9-4f6f-a8cb-1debd31e622f"},"name":"page-route","hosts":["localhost"],"updated_at":1549175021,"preserve_host":false,"regex_priority":0,"paths":["\/page"],"sources":null,"destinations":null,"snis":null,"protocols":["http"],"strip_path":true}]}root@kong:~#


# curl -X GET http://localhost:8001/services/page-service/routes
{"next":null,"data":[{"created_at":1549175021,"methods":null,"id":"99b17157-2dff-4fad-ae85-449de46a5991","service":{"id":"8855bebd-a9d9-4f6f-a8cb-1debd31e622f"},"name":"page-route","hosts":["localhost"],"updated_at":1549175021,"preserve_host":false,"regex_priority":0,"paths":["\/page"],"sources":null,"destinations":null,"snis":null,"protocols":["http"],"strip_path":true}]}root@kong:~#
```

execute kong tutorial procedure
https://docs.konghq.com/1.0.x/getting-started/configuring-a-service/
```
curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=example-service' \
  --data 'url=http://mockbin.org'

curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=example.com'

curl -i -X GET \
  --url http://localhost:8000/ \
  --header 'Host: example.com'
```

retry
```
# curl -i -X POST \
>   --url http://localhost:8001/services/ \
>   --data 'name=hoge-service' \
>   --data 'url=http://localhost:4000'
HTTP/1.1 201 Created
Date: Sun, 03 Feb 2019 06:50:10 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/1.0.3
Content-Length: 255

{"host":"localhost","created_at":1549176610,"connect_timeout":60000,"id":"3731b670-1e51-4875-852e-64a57dc0914e","protocol":"http","name":"hoge-service","read_timeout":60000,"port":4000,"path":null,"updated_at":1549176610,"retries":5,"write_timeout":60000}root@kong:~

curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=127.0.0.1'

curl -i -X GET \
  --url http://localhost:8000/hoge \
  --header 'Host: 127.0.0.1'
```

OOps. Not match cross functional requirements.
I don't need HTTP Header 'Host' value.

