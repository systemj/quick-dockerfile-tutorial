
# DockerFile Tutorial


## Part 1 - As basic as possible
### hello.sh
```
#!/bin/bash
echo "Hello World!"
```

### DockerFile
```
FROM centos
COPY hello.sh .
```

### Build:
```
docker build -t hello .
```

### Run:
```
docker run --rm hello                 # does nothing
docker run -i -t --rm hello           # does nothing
docker run -i -t --rm hello hello.sh  # error not in path
docker run -i -t --rm hello /hello.sh # works
docker run -i -t --rm hello uname -a  # also works
```

## Part 2 - CMD
### hello.sh
```
#!/bin/bash
echo "Hello World!"
```

### DockerFile
```
FROM centos
COPY hello.sh .
CMD /hello.sh
```

### Build:
```
docker build -t hello .
```

### Run:
```
docker run --rm hello                 # works 
docker run -i -t --rm hello           # works
docker run -i -t --rm hello hello.sh  # error not in path
docker run -i -t --rm hello /hello.sh # still works
docker run -i -t --rm hello uname -a  # also still works
```

## Part 3 - WORKDIR
### hello.sh
```
#!/bin/bash
echo "Hello World!"
```

### DockerFile
```
FROM centos
WORKDIR /app
COPY hello.sh .
CMD /app/hello.sh
```

### Build:
```
docker build -t hello .
```

### Run:
```
docker run --rm hello                     # works 
docker run -i -t --rm hello               # works
docker run -i -t --rm hello /hello.sh     # file doesn't exist
docker run -i -t --rm hello /app/hello.sh # works
```

## Part 4 - RUN
### hello.sh
```
#!/bin/bash
figlet "Hello World!"
```

### DockerFile
```
FROM ubuntu:latest
WORKDIR /app
RUN apt update
RUN apt install -y figlet
COPY hello.sh .
CMD /app/hello.sh
```

### Build:
```
docker build -t hello .
```

### Run:
```
docker run --rm hello  # big hello
```

## Part 5 - ADD (url), COPY (whole dir)
### hello.sh
```
#!/bin/bash
figlet "Hello World!"
cat ascii.txt
```

### DockerFile
```
FROM ubuntu:latest
WORKDIR /app
RUN apt update
RUN apt install -y figlet
COPY . .
ADD https://systemj.net/ascii.txt .
CMD /app/hello.sh
```

### Build:
```
docker build -t hello .
```

### Run:
```
docker run --rm hello  # big hello + dragon
```


## Part 6 - ENTRYPOINT + CMD
### hello.sh
```
#!/bin/bash
figlet "${*}"
```

### DockerFile
```
FROM ubuntu:latest
WORKDIR /app
RUN apt update
RUN apt install -y figlet
COPY . .
ENTRYPOINT ["/app/hello.sh"]
CMD ["Hello", "World!"]
```

### Build:
```
docker build -t hello .
```

### Run:
```
docker run --rm hello         # big hello world
docker run --rm hello dfw uug # big dfw uug
```


## Part 7 - Python app - EXPOSE
### hello.py
```
#!/usr/local/bin/python3
from flask import Flask
app = Flask(__name__)
 
@app.route("/")
def hello():
    return "Hello World!\n"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
```

### requirements.txt
_capitalization important..._
```
Flask
```

### DockerFile
Install requirements early (rebuild cache)
```
FROM python:3
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY hello.py .
ENTRYPOINT ["/usr/local/bin/python3"]
CMD ["/app/hello.py"]
EXPOSE 8080
```

### Build:
```
docker build -t hellopy .
```

### Run:
```
docker run -i -t --rm hellopy               # unreachable
docker run -i -t --rm -p 8080:8080 hellopy  # can reach on http://localhost/
```

EXPOSE doesn't actually do anything, and is more for documentation.

## Docker Top
Allows you to see container just processes at the host level
```
docker ps
docker top <container>
```

_Note the processes run as root by default_

## Part 8 - Python app - USER
### hello.py
```
#!/usr/local/bin/python3
from flask import Flask
app = Flask(__name__)
 
@app.route("/")
def hello():
    return "Hello World!\n"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
```

### requirements.txt
_capitalization important..._
```
Flask
```

### DockerFile
Install requirements early (rebuild cache)
```
FROM python:3
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY hello.py .
ENTRYPOINT ["/usr/local/bin/python3"]
CMD ["/app/hello.py"]
EXPOSE 8080
USER 1234:1234
```

### Build:
```
docker build -t hellopy .
```

### Run:
```
docker run -i -t --rm -p 8080:8080 hellopy  # can reach on http://localhost:8080/
docker ps                                   # list containers
docker top <container>                      # show running as user 1234
```

## Part 9 - Compiling Code (pile on dependencies)
### hello.c
```
#include <stdio.h>
int main(){
  printf("Hello World!\n");
  return(0);
}
```

### DockerFile
```
FROM fedora:latest
WORKDIR /src
RUN yum install -y gcc
COPY hello.c .
RUN gcc -Wall -o hello hello.c
WORKDIR /app
RUN cp /src/hello .
ENTRYPOINT ["/app/hello"]
```

### Build:
```
docker build -t helloc .
```

### Run:
```
docker run -i -t --rm helloc # works
```

### Compare Container Sizes
```
docker images | grep fedora
docker images | grep helloc  # significantly bigger due to build tools
```

## Part 10 - Multistage Builds
### hello.c
```
#include <stdio.h>
int main(){
  printf("Hello World!\n");
  return(0);
}
```

### DockerFile
```
FROM fedora:latest AS my_builder
WORKDIR /src
RUN yum install -y gcc
COPY hello.c .
RUN gcc -Wall -o hello hello.c

FROM fedora:latest 
WORKDIR /app
COPY --from=my_builder /src/hello /app/hello
ENTRYPOINT ["/app/hello"]
```

### Build:
```
docker build -t helloc .
```

### Run:
```
docker run -i -t --rm helloc # works
```

### Compare Container Sizes
```
docker images | grep fedora
docker images | grep helloc  # essentially the same
```

## Part 11 - Alpine
### hello.c
```
#include <stdio.h>
int main(){
  printf("Hello World!\n");
  return(0);
}
```

### DockerFile
```
FROM alpine:latest AS my_builder
WORKDIR /src
RUN apk update
RUN apk add gcc libc-dev
COPY hello.c .
RUN gcc -Wall -o hello hello.c

FROM alpine:latest 
WORKDIR /app
COPY --from=my_builder /src/hello /app/hello
ENTRYPOINT ["/app/hello"]
```

### Build:
```
docker build -t helloalp .
```

### Run:
```
docker run -i -t --rm helloc # works
```

### Compare Container Sizes
```
docker images | grep fedora   # ~254MB
docker images | grep helloc   # ~254MB
docker images | grep alpine   # ~4.41MB
docker images | grep helloalp # ~4.42MB
```

## Best Practices

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

https://runnable.com/blog/9-common-dockerfile-mistakes













