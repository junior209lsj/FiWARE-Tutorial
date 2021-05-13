

# FiWARE-Tutorial

2021학년도 1학기 분산처리특강 FiWARE 설치 및 기본 사용법

작성자: 이성재

## 배경 지식

FiWARE tutorial을 시작하기 전에, 예제 실행 및 FiWARE의 시스템 구조를 이해하는 데 필수적인 요소를 먼저 설명한다.

### Docker

Docker는 어플리케이션의 개발, 배포, 실행을 infrastructure 독립적으로 가능하게 해 주는 개방형 플랫폼이다. Docker는 다양한 프로그램, 실행환경을 컨테이너로 추상화하고, 동일한 인터페이스를 제공하여 프로그램의 배포 및 관리를 단순하게 해 준다.

#### Docker container

Container는 소프트웨어의 실행 환경을 독립적으로 유지할 수 있게 해 주는 운영체제 수준의 격리 기술을 의미한다. 즉 docker 

### RESTful API

#### Query string

쿼리 스트링을 이용하여 요청 URL에 파라미터를 넘길 수 있다. 아래와 같은 방식으로 파라미터를 전달한다.

```
http://[host name]:[port]/[path of url]/[file]?[key1]=[value1]&[key2]=[value2]&...&[key n]=[value n]
```

= 연산자로 key와 value를 구분한다.

## 사전 준비사항

[Docker 홈페이지](https://docs.docker.com/engine/install/)의 설명에 기반하여 docker를 설치한다. 본 문서는 Ubuntu 18.04.5를 기준으로 진행하지만 docker에서 지원하는 운영체제를 가지고 있다면 그에 맞추어 설치해도 무방하다.

### 실습 환경 정보

* Ubuntu 18.04.5에서 진행하였다.
* Docker를 설치할 수 있는 환경이라면 플랫폼 독립적으로 실행 가능

### Docker 설치

1) 구버전이 설치되어 있는 경우 아래 명령어로 제거한다.

```sh
$ sudo apt purge docker docker-engine docker.io docker-ce docker-ce-cli containerd.io runc
```

2) 아래 명령어를 통해 사전 준비사항을 설치한다.

```sh
$ sudo apt update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

3) Docker의 공식 GPG key를 추가한다.

```sh
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

4) Stable repository setup을 진행한다.

```sh
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5) Docker engine을 설치한다.

```sh
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli containerd.io
```

6) 설치가 잘 되었는지 확인한다.

```sh
$ sudo docker run hello-world
```

아래와 같은 창이 출력되면 설치가 성공한 것이다.

```sh
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### FiWARE와 MongoDB 이미지 연결

Orion context broker는 데이터 모델로 NGSI-V2를 사용하고 데이터베이스로 MongoDB를 사용한다. Docker를 이용하여 FiWARE 사용 환경을 구축하기 위해 `fiware/orion` 컨테이너와 `mongo:4.2` 컨테이너를 연결한다. network 명령어를 이용하여 두 컨테이너를 연결할 수 있다.

```sh
$ docker pull mongo:4.2
$ docker pull fiware/orion
$ docker network create fiware_default
```

아래 명령어를 이용하여 받아온 docker image, 설정한 docker network 정보를 확인할 수 있다.

```sh
$ docker images # docker image 목록 확인
$ docker network ls # docker network 목록 확인
$ docker network inspect fiware_default # fiware_default 정보 확인
```

아래 명령어로 MongoDB 컨테이너를 실행한다.

```sh
$ docker run -d --name=mongo-db --network=fiware-default \
    --expose=27017 mongo:4.2 --bind_ip_all
```

아래 명령어로 fiware-orion 컨테이너를 실행한다.

```sh
$ docker run -d --name fiware-orion -h orion --network=fiware_default \
    -p 1026:1026 fiware/orion -dbhost mongo-db
```

### (선택사항) POSTMAN 프로그램 사용

Postman은 무료 REST API 테스트 프로그램이며, REST API 기반으로 통신하는 경우가 많은 FiWARE 실습에서 아주 유용하게 사용할 수 있다. Windows, Mac, Linux에서 모두 사용 가능하며 홈페이지에서 다운로드받을 수 있다. Ubuntu에서는 아래 명령어로 설치 가능하다.

```
$ sudo snap install postman
```

## Hello World!

컨테이너를 실행한 후 `docker ps` 명령어를 이용하여 실행되고 있는 컨테이너를 확인할 수 있다. 아래와 같이 두 개의 컨테이너가 실행되고 있음을 확인할 수 있다.

**Request**

```sh
$ docker ps
```

**Response**

```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                       NAMES
3ea461f2e849   fiware/orion   "/usr/bin/contextBro…"   14 minutes ago   Up 14 minutes   0.0.0.0:1026->1026/tcp, :::1026->1026/tcp   fiware-orion
1d8c8bbb7847   mongo:4.2      "docker-entrypoint.s…"   16 minutes ago   Up 16 minutes   27017/tcp                                   mongo-db
```

컨테이너가 실행되고 있는 것을 확인한 상태에서, 아래 curl 명령어 또는 postman request로 Orion context manager의 버전 정보를 `GET` 메소드로 요청하여 정상 작동을 확인할 수 있다.

**Request**

```sh
curl --location --request GET 'http://localhost:1026/version'
```

**Response**

```json
{
"orion" : {
  "version" : "3.0.0-next",
  "uptime" : "0 d, 3 h, 26 m, 9 s",
  "git_hash" : "686c4f10cb647eab70a674cfaa88ca7977b53223",
  "compile_time" : "Wed May 12 13:30:11 UTC 2021",
  "compiled_by" : "root",
  "compiled_in" : "46c2c05dcde4",
  "release_date" : "Wed May 12 13:30:11 UTC 2021",
  "machine" : "x86_64",
  "doc" : "https://fiware-orion.rtfd.io/",
  "libversions": {
     "boost": "1_66",
     "libcurl": "libcurl/7.61.1 OpenSSL/1.1.1g zlib/1.2.11 nghttp2/1.33.0",
     "libmicrohttpd": "0.9.70",
     "openssl": "1.1",
     "rapidjson": "1.1.0",
     "mongoc": "1.17.4",
     "bson": "1.17.4"
  }
}
}
```

## Data entitiy 생성

RESTful API의 POST method를 이용하여 data entitiy를 만들어 본다. Orion context manager에 다음과 같은 상점 정보를 `.json` 형태로 생성할 것이다.

```json
// store_test1.json
{
  "id": "urn:ngsi-ld:Store:001",
  "type": "Store",
  "address": {
      "type": "PostalAddress",
      "value": {
          "streetAddress": "Bornholmer Straße 65",
          "addressRegion": "Berlin",
          "addressLocality": "Prenzlauer Berg",
          "postalCode": "10439"
      },
      "metadata": {
          "verified": {
              "value": true,
              "type": "Boolean"
          }
      }
  },
  "location": {
      "type": "geo:json",
      "value": {
           "type": "Point",
           "coordinates": [13.3986, 52.5547]
      }
  },
  "name": {
      "type": "Text",
      "value": "Bösebrücke Einkauf"
  }
}
```

이 `.json` 파일은 NGSIV2 API를 따르고 있다??. 모든 entitiy는 unique id를 가지고 있어야 한다. 각 Attribute는 `type`, `value` 쌍으로 구성되어 있다. 이 내용의 entitiy를 Postman request를 통해서 POST하거나 아래와 같은 curl 명령어를 이용하여 POST할 수 있다.

**Request**

```sh
$ curl --location --request POST 'http://localhost:1026/v2/entities' \
--header 'Content-Type: application/json' \
--data-raw '{
  "id": "urn:ngsi-ld:Store:001",
  "type": "Store",
  "address": {
      "type": "PostalAddress",
      "value": {
          "streetAddress": "Bornholmer Straße 65",
          "addressRegion": "Berlin",
          "addressLocality": "Prenzlauer Berg",
          "postalCode": "10439"
      },
      "metadata": {
          "verified": {
              "value": true,
              "type": "Boolean"
          }
      }
  },
  "location": {
      "type": "geo:json",
      "value": {
           "type": "Point",
           "coordinates": [13.3986, 52.5547]
      }
  },
  "name": {
      "type": "Text",
      "value": "Bösebrücke Einkauf"
  }
}'
```

Create가 성공하면 HTTP response로 201을 받는다. 똑같은 명령을 한 번 더 실행하면 id가 충돌하기 때문에 422 (unprocessable entitiy)를 받는다.

**Response**

```sh
# Error
{"error":"Unprocessable","description":"Already Exists"}
```

아래 json file을 다시 post하게 되면 id가 겹치지 않기 때문에 정상적으로 create된다.

```json
// store_test2.json
{
  "type": "Store",
  "id": "urn:ngsi-ld:Store:002",
  "address": {
      "type": "PostalAddress",
      "value": {
          "streetAddress": "Friedrichstraße 44",
          "addressRegion": "Berlin",
          "addressLocality": "Kreuzberg",
          "postalCode": "10969"
      },
      "metadata": {
          "verified": {
              "value": true,
              "type": "Boolean"
          }
      }
  },
  "location": {
      "type": "geo:json",
      "value": {
           "type": "Point",
           "coordinates": [13.3903, 52.5075]
      }
  },
  "name": {
      "type": "Text",
      "value": "Checkpoint Markt"
  }
}
```

## Data Entity 요청

`"id": "urn:ngsi-ld:Store:001"`, `"id": "urn:ngsi-ld:Store:002"`의 entitiy를 create한 상태에서 curl 명령어나 postman 프로그램으로 `GET` 메소드를 이용하여 Context manager에게 data entitiy를 요청하면 아래와 같이 모든 내용의 entitiy를 받게 된다.

**Request**

```sh
# curl 요청 명령어
$ curl --location --request GET 'http://localhost:1026/v2/entities'
```

**Response**

```json
// 응답
[
    {
        "id": "urn:ngsi-ld:Store:001",
        "type": "Store",
        "address": {
            "type": "PostalAddress",
            "value": {
                "streetAddress": "Bornholmer Straße 65",
                "addressRegion": "Berlin",
                "addressLocality": "Prenzlauer Berg",
                "postalCode": "10439"
            },
            "metadata": {
                "verified": {
                    "type": "Boolean",
                    "value": true
                }
            }
        },
        "location": {
            "type": "geo:json",
            "value": {
                "type": "Point",
                "coordinates": [
                    13.3986,
                    52.5547
                ]
            },
            "metadata": {}
        },
        "name": {
            "type": "Text",
            "value": "Bösebrücke Einkauf",
            "metadata": {}
        }
    },
    {
        "id": "urn:ngsi-ld:Store:002",
        "type": "Store",
        "address": {
            "type": "PostalAddress",
            "value": {
                "streetAddress": "Friedrichstraße 44",
                "addressRegion": "Berlin",
                "addressLocality": "Kreuzberg",
                "postalCode": "10969"
            },
            "metadata": {
                "verified": {
                    "type": "Boolean",
                    "value": true
                }
            }
        },
        "location": {
            "type": "geo:json",
            "value": {
                "type": "Point",
                "coordinates": [
                    13.3903,
                    52.5075
                ]
            },
            "metadata": {}
        },
        "name": {
            "type": "Text",
            "value": "Checkpoint Markt",
            "metadata": {}
        }
    }
]
```

## Data entitiy 쿼리

### 모든 data entitiy 요청하기

`"id": "urn:ngsi-ld:Store:001"` entitiy만 가지고 오고 싶으면 아래와 같은 명령어로 데이터를 쿼리할 수 있다.

**Request**

```sh
curl --location --request GET 'http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001'
```

**Response**
```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "type": "PostalAddress",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "metadata": {
            "verified": {
                "type": "Boolean",
                "value": true
            }
        }
    },
    "location": {
        "type": "geo:json",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
        },
        "metadata": {}
    },
    "name": {
        "type": "Text",
        "value": "Bösebrücke Einkauf",
        "metadata": {}
    }
}
```

### options=와 attrs= 사용하기


`type`을 제외한 `key`:`value`쌍만 보고 싶으면 `options=keyValues`를 이용하여 간결하게 정보를 쿼리할 수 있다.

**Request**

```sh
curl --location --request GET 'http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001?options=keyValues'
```

**Response**

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "streetAddress": "Bornholmer Straße 65",
        "addressRegion": "Berlin",
        "addressLocality": "Prenzlauer Berg",
        "postalCode": "10439"
    },
    "location": {
        "type": "Point",
        "coordinates": [
            13.3986,
            52.5547
        ]
    },
    "name": "Bösebrücke Einkauf"
}
```

이외에도 `value`만 쿼리하고 싶으면 `options=values`를 이용할 수 있다. 이 경우에는 `attrs=[attribute]`와 조합하여 특정 `key`의 `value`만 쿼리할 수 있다. 예를 들어 location만 쿼리하고 싶으면 아래와 같이 요청한다.

**Request**

```sh
curl --location --request GET 'http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001?options=values&attrs=location'
```

**Response**

```json
[
    {
        "type": "Point",
        "coordinates": [
            13.3986,
            52.5547
        ]
    }
]
```

### URL 주소로 특정 attribute 받아오기

또한 URL에 `/v2/entities/{id}/attrs/{attrsName}/value`와 같은 형식으로도 데이터를 요청할 수 있다. 예를 들어 바로 위 예제와 동일한 결과를 얻기 위해 아래와 같은 요청을 할 수도 있다.

```sh
curl --location --request GET 'http://localhost:1026/v2/entities/urn:ngsi-ld:Store:001/attrs/location/value'
```

### 데이터 필터링

특정 `value`를 가지고 있는 data entitiy를 필터링하고 싶은 경우 `q=[key]==[value]`를 이용한다. 예를 들어 가게 이름이 'Checkpoint Markt'인 data entitiy (`"id": "urn:ngsi-ld:Store:001"`인 entitiy)의 전체 정보를 보고 싶은 경우 아래와 같이 요청한다. 웹 표준에서 `'`는 `%27`이고, 공백은 `%20`이다 ([참고](https://ghdwn0217.tistory.com/76)).

**Request**

```sh
curl --location --request GET 'http://localhost:1026/v2/entities/?q=name==%27Checkpoint%20Markt%27'
```

**Response**

```json
[
    {
        "id": "urn:ngsi-ld:Store:002",
        "type": "Store",
        "address": {
            "type": "PostalAddress",
            "value": {
                "streetAddress": "Friedrichstraße 44",
                "addressRegion": "Berlin",
                "addressLocality": "Kreuzberg",
                "postalCode": "10969"
            },
            "metadata": {
                "verified": {
                    "type": "Boolean",
                    "value": true
                }
            }
        },
        "location": {
            "type": "geo:json",
            "value": {
                "type": "Point",
                "coordinates": [
                    13.3903,
                    52.5075
                ]
            },
            "metadata": {}
        },
        "name": {
            "type": "Text",
            "value": "Checkpoint Markt",
            "metadata": {}
        }
    }
]
```

