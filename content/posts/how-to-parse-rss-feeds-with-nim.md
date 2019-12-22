---
title: "How to parse RSS feeds with Nim"
date: 2019-12-22T12:30:00-04:00
---

You have some RSS and you need to parse it into a Nim object.

For this example, we're going to be using the following RSS feed:

{{< highlight xml >}}
<?xml version="1.0" encoding="UTF-8"?>
<rss xmlns:torrent="http://xmlns.ezrss.it/0.1/" version="2.0">
    <channel>
        <title>TV Torrents RSS feed - EZTV</title>
        <link>https://eztv.io/</link>
        <description>TV Torrents RSS feed - EZTV</description>
        <lastBuildDate>Sun, 22 Dec 2019 12:37:01 -0500</lastBuildDate>
        <item>
            <title>Five Day Biz Fix S01E03 480p x264-mSD</title>
            <category>TV</category>
            <link>https://eztv.io/ep/1400104/five-day-biz-fix-s01e03-480p-x264-msd/</link>
            <guid>https://eztv.io/ep/1400104/five-day-biz-fix-s01e03-480p-x264-msd/</guid>
            <pubDate>Sun, 22 Dec 2019 11:22:52 -0500</pubDate>
            <torrent:contentLength>266476297</torrent:contentLength>
            <torrent:infoHash>92424828AFFE4C897D5F4171BCE59B743CAAFA6C</torrent:infoHash>
            <torrent:magnetURI><![CDATA[magnet:?xt=urn:btih:92424828AFFE4C897D5F4171BCE59B743CAAFA6C&dn=Five.Day.Biz.Fix.S01E03.480p.x264-mSD%5Beztv%5D.mkv&tr=udp%3A%2F%2Ftracker.publicbt.com%2Fannounce&tr=udp%3A%2F%2Fopen.demonii.com%3A1337&tr=http%3A%2F%2Ftracker.trackerfix.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969&tr=udp%3A%2F%2Fexodus.desync.com%3A6969]]></torrent:magnetURI>
            <torrent:seeds>0</torrent:seeds>
            <torrent:peers>0</torrent:peers>
            <torrent:verified>0</torrent:verified>
            <torrent:fileName>Five.Day.Biz.Fix.S01E03.480p.x264-mSD[eztv].mkv</torrent:fileName>
            <enclosure url="https://zoink.ch/torrent/Five.Day.Biz.Fix.S01E03.480p.x264-mSD[eztv].mkv.torrent" length="266476297" type="application/x-bittorrent" />
        </item>
        <item>
            <title>Five Day Biz Fix S01E03 480p x264-mSD</title>
            <category>TV</category>
            <link>https://eztv.io/ep/1400104/five-day-biz-fix-s01e03-480p-x264-msd/</link>
            <guid>https://eztv.io/ep/1400104/five-day-biz-fix-s01e03-480p-x264-msd/</guid>
            <pubDate>Sun, 22 Dec 2019 11:22:52 -0500</pubDate>
            <torrent:contentLength>266476297</torrent:contentLength>
            <torrent:infoHash>92424828AFFE4C897D5F4171BCE59B743CAAFA6C</torrent:infoHash>
            <torrent:magnetURI><![CDATA[magnet:?xt=urn:btih:92424828AFFE4C897D5F4171BCE59B743CAAFA6C&dn=Five.Day.Biz.Fix.S01E03.480p.x264-mSD%5Beztv%5D.mkv&tr=udp%3A%2F%2Ftracker.publicbt.com%2Fannounce&tr=udp%3A%2F%2Fopen.demonii.com%3A1337&tr=http%3A%2F%2Ftracker.trackerfix.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969&tr=udp%3A%2F%2Fexodus.desync.com%3A6969]]></torrent:magnetURI>
            <torrent:seeds>0</torrent:seeds>
            <torrent:peers>0</torrent:peers>
            <torrent:verified>0</torrent:verified>
            <torrent:fileName>Five.Day.Biz.Fix.S01E03.480p.x264-mSD[eztv].mkv</torrent:fileName>
            <enclosure url="https://zoink.ch/torrent/Five.Day.Biz.Fix.S01E03.480p.x264-mSD[eztv].mkv.torrent" length="266476297" type="application/x-bittorrent" />
        </item>
    </channel>
</rss>
{{< /highlight >}}

# Downloading the XML into a string we can parse

Import the httpClient into your code and the run the `getContent` function.

{{< highlight nim >}}
import httpClient
import streams
import xmlparser
import xmltree

proc fetchXml(): XmlNode =
    var client = newHttpClient()
    let xml = client.getContent("https://eztv.io/ezrss.xml")
    let xmlStream = newStringStream(xml)
    return parseXML(xmlStream)
{{< /highlight >}}

Now the `fetchXml` function will return our XML parsed and ready to be queried in an XmlNode object.

# Extracting data from our XmlNode

Extracting the info is pretty easy as well.

First, create your Nim type to hold the data you need.

Then you can use the `findAll` function to query for specific items within the XML.

{{< highlight nim >}}
import httpClient
import streams
import xmlparser
import xmltree

type
    Torrent* = object
        name*: string
        canonical_url*: string

proc fetchXml(): XmlNode =
    var client = newHttpClient()
    let xml = client.getContent("https://eztv.io/ezrss.xml")
    let xmlStream = newStringStream(xml)
    return parseXML(xmlStream)

proc latest*(): seq[Torrent] =
    var xmlRoot = fetchXml()
    for item_node in xmlRoot.findAll("item"):
        var torrent: Torrent = Torrent()
        torrent.name = item_node.child("title").innerText
        torrent.canonical_url = item_node.child("link").innerText
        result.add(torrent)
{{< /highlight >}}

The end result is you'll have a `seq[Torrent]` with the data extracted from your XML.