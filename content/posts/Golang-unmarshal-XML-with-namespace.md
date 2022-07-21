---
title: "Golang 解析带命名空间的 XML"
date: 2022-07-21T16:30:29+08:00
draft: true
tags: ['Golang','XML']
categories: ['Golang']
draft: false
---

## 遇到的问题

在写业余项目时遇到一个xml无法解析的问题，情景如下。使用以下 Golang 解析 YouTube 的 RSS 返回的 XML ，无法获得 XML 内`<media:thumbnail>` 内的数据。

```golang
type YoutubeRSS struct {
	XMLName xml.Name `xml:"feed"`
	Entry   []struct {
		Title string `xml:"title"`
		Link  struct {
			Href string `xml:"href,attr"`
		} `xml:"link"`
		Updated   time.Time `xml:"updated"`
		MediaGrop struct {
			MediaThumbnail struct {
				Url string `xml:"url,attr"`
			} `xml:"media:thumbnail"`
		} `xml:"media:group"`
	} `xml:"entry"`
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<feed xmlns:yt="http://www.youtube.com/xml/schemas/2015" xmlns:media="http://search.yahoo.com/mrss/" xmlns="http://www.w3.org/2005/Atom">
   <link rel="self" href="http://www.youtube.com/feeds/videos.xml?channel_id=UCSI11yVCM5x45_iiL5YcQZw"/>
   <id>yt:channel:UCSI11yVCM5x45_iiL5YcQZw</id>
   <yt:channelId>UCSI11yVCM5x45_iiL5YcQZw</yt:channelId>
   <title>Priscilla Abby 蔡恩雨</title>
   <link rel="alternate" href="https://www.youtube.com/channel/UCSI11yVCM5x45_iiL5YcQZw"/>
   <author>
      <name>Priscilla Abby 蔡恩雨</name>
      <uri>https://www.youtube.com/channel/UCSI11yVCM5x45_iiL5YcQZw</uri>
   </author>
   <published>2009-10-08T06:56:43+00:00</published>
   <entry>
      <id>yt:video:cS-k09EMv-A</id>
      <yt:videoId>cS-k09EMv-A</yt:videoId>
      <yt:channelId>UCSI11yVCM5x45_iiL5YcQZw</yt:channelId>
      <title>蔡恩雨 Priscilla Abby《唯舞唯一》MV Behind The Scene 幕後花絮</title>
      <link rel="alternate" href="https://www.youtube.com/watch?v=cS-k09EMv-A"/>
      <author>
         <name>Priscilla Abby 蔡恩雨</name>
         <uri>https://www.youtube.com/channel/UCSI11yVCM5x45_iiL5YcQZw</uri>
      </author>
      <published>2022-06-22T12:00:17+00:00</published>
      <updated>2022-06-24T00:20:57+00:00</updated>
      <media:group>
         <media:title>蔡恩雨 Priscilla Abby《唯舞唯一》MV Behind The Scene 幕後花絮</media:title>
         <media:content url="https://www.youtube.com/v/cS-k09EMv-A?version=3" type="application/x-shockwave-flash" width="640" height="390"/>
         <media:thumbnail url="https://i4.ytimg.com/vi/cS-k09EMv-A/hqdefault.jpg" width="480" height="360"/>
         <media:description>雖然《唯舞唯一》MV只用了一天的時間拍攝，但裡頭的用心以及誠意滿滿</media:description>
         <media:community>
            <media:starRating count="328" average="5.00" min="1" max="5"/>
            <media:statistics views="7743"/>
         </media:community>
      </media:group>
   </entry>
</feed>
```

## 查询资料

XML 中带冒号的节点呗称为命名空间，Golang 解析时候不需要处理命名空间。

## 修改代码

使用以下代码即可解析该 XML。

```Golang
type YoutubeRSS struct {
	XMLName xml.Name `xml:"feed"`
	Entry   []struct {
		Title string `xml:"title"`
		Link  struct {
			Href string `xml:"href,attr"`
		} `xml:"link"`
		Updated   time.Time `xml:"updated"`
		MediaGrop struct {
			Title          string `xml:"title"`
			MediaThumbnail struct {
				Url string `xml:"url,attr"`
			} `xml:"thumbnail"`
		} `xml:"group"`
	} `xml:"entry"`
}
```