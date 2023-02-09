## libcurl.nim
Nim wrapper for libcurl (v7.x)

## libcurl v7.x
- source/binary packages - all platforms: https://curl.haxx.se/download.html
- buid instructions - all platforms: https://curl.haxx.se/docs/install.html
- binary package (.dll) - windows users: https://curl.haxx.se/windows/

## basic example

```nim
import libcurl

proc curlWriteFn(
  buffer: cstring,
  size: int,
  count: int,
  outstream: pointer): int =
  
  let outbuf = cast[ref string](outstream)
  outbuf[] &= buffer
  result = size * count
  
let webData: ref string = new string
let curl = easy_init()

discard curl.easy_setopt(OPT_USERAGENT, "Mozilla/5.0")
discard curl.easy_setopt(OPT_HTTPGET, 1)
discard curl.easy_setopt(OPT_WRITEDATA, webData)
discard curl.easy_setopt(OPT_WRITEFUNCTION, curlWriteFn)
discard curl.easy_setopt(OPT_URL, "http://localhost/")

let ret = curl.easy_perform()
if ret == E_OK:
  echo(webData[])
```

## Upload a file with FTPS

```
import libcurl
import os

const connectionTimeoutSeconds = 5
const transferFileTimeoutSeconds = 20
const testFile = "foo.txt"
let remoteUri = cast[cstring]("ftp://username:password@host:port/" & testFile)
var headerlist: PsList
headerlist = headerlist.slist_append("RNFR " & testFile)
headerlist = headerlist.slist_append("RNTO " & testFile)
let file = open(testFile)
let fileInfo = getFileInfo(file)

let curl = easy_init()
discard curl.easy_setopt(OPT_UPLOAD, 1)
discard curl.easy_setopt(OPT_URL, remoteUri)
discard curl.easy_setopt(OPT_POSTQUOTE, headerlist)
discard curl.easy_setopt(OPT_READDATA, file)
discard curl.easy_setopt(OPT_FTP_SSL, FTPSSL_CONTROL)
discard curl.easy_setopt(OPT_TIMEOUT, transferFileTimeoutSeconds)
discard curl.easy_setopt(OPT_CONNECTTIMEOUT, connectionTimeoutSeconds)
discard curl.easy_setopt(OPT_INFILESIZE_LARGE, fileInfo.size)
slist_free_all(headerlist)
let ret = curl.easy_perform()
easy_cleanup(curl)
global_cleanup()
if ret == E_OK or ret == E_PARTIAL_FILE:
    echo "OK"
```

see also: https://curl.haxx.se/libcurl/c/example.html
