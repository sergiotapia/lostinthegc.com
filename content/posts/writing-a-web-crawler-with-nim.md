---
title: "Writing a web crawler with Nim"
date: 2020-07-23T11:49:00-04:00
---

Here's how you can easily build a web crawler with Nim.

Let's create a simple type called `Torrent` to keep the data we crawl in
a nice object.

{{< highlight nim >}}
import times

type
  Torrent* = object
    name*: string
    source*: string
    uploaded_at*: DateTime
    canonical_url*: string
    magnet_url*: string
    size*: string
    seeders*: int
    leechers*: int

proc newTorrent*(): Torrent =
  result = Torrent(uploaded_at: now())
{{< /highlight >}}

We need to write a crawler, let's build one that grabs the latest
torrent uploads on eztv. This will download the XML feed of the latest
torrents.

{{< highlight nim >}}
proc fetchXml(): Future[XmlNode] {.async} =
  let client = newAsyncHttpClient()
  let xml = await client.getContent("https://eztv.io/ezrss.xml")
  let xmlStream = newStringStream(xml)
  client.close()
  return parseXML(xmlStream)
{{< /highlight >}}

Now we can parse that XML pretty easily.

{{< highlight nim >}}
proc fetchLatest*() {.async} =
  echo "[eztv] Starting EZTV crawl"

  var xmlRoot = await fetchXml()
  for item_node in xmlRoot.findAll("item"):
    var torrent: Torrent = newTorrent()
    torrent.name = item_node.child("title").innerText
    torrent.source = "eztv"
    torrent.canonical_url = item_node.child("link").innerText
    torrent.size = item_node.child("torrent:contentLength").innerText
    torrent.seeders = parseInt(item_node.child("torrent:seeds").innerText)
    torrent.leechers = parseInt(item_node.child("torrent:peers").innerText)
    for ic in item_node.child("torrent:magnetURI").items:
      torrent.magnet_url = ic.text
      
    echo "Torrent: {torrent}"  
{{< /highlight >}}

Finally we need a way to start our crawler and periodically run it:

{{< highlight nim >}}
proc startCrawl*() {.async} =
  while true:
    try:
      await fetchLatest()
      await sleepAsync(30000)
    except:
      echo "[eztv] Crawler error, restarting..."
{{< /highlight >}}

Special notice:
- The {.async} pragma.
- The sleepAsync function call
- The await

In our main module, we can start a simple Jester API, and keep
our crawlers running indefinitely.

{{< highlight nim >}}
import htmlgen
import jester
import threadpool
import "database"
import "./helpers/datetime"
from "./crawlers/eztv" import nil
from "./crawlers/leetx.nim" import nil
from "./crawlers/nyaa.nim" import nil
from "./crawlers/nyaa_pantsu.nim" import nil
from "./crawlers/yts.nim" import nil
from "./crawlers/torrent_downloads.nim" import nil

when isMainModule:
  asyncCheck eztv.startCrawl()
  asyncCheck leetx.startCrawl()
  asyncCheck nyaa.startCrawl()
  asyncCheck nyaa_pantsu.startCrawl()
  asyncCheck yts.startCrawl()
  asyncCheck torrentdownloads.startCrawl()
  
  router apiRouter:
    get "/":
      resp "Jester is running!"

  let port = Port 5000
  var jesterServer = initJester(apiRouter, settings=newSettings(port=port))
  jesterServer.serve()
{{< /highlight >}}

asyncCheck: https://nim-lang.org/docs/asyncfutures.html#asyncCheck%2CFuture[T]

>Sets a callback on future which raises an exception if the future 
>finished with an error.
>This should be used instead of discard to discard void futures, or use waitFor 
>if you need to wait for the future's completion.
