---
title: "How to parse a json array with Nim"
date: 2021-05-24T23:40:01-04:00
---

Given the following JSON, how can you parse it into a nice array to work with in Nim?

{{< highlight json >}}
[
    {
        "name":"Sergio",
        "favoriteMovie":{
            "title":"Taxi Driver",
            "releaseYear":1976
        }
    },
    {
        "name":"Daniel",
        "favoriteMovie":{
            "title":"Frozen",
            "releaseYear":2013
        }
    }
]
{{< /highlight >}}

It's pretty easy, first let's start by creating the types to marshal the json into.

{{< highlight nim >}}
type
  Movie* = object
    title*: Option[string]
    releaseYear*: Option[int]

type 
  Person* = object
    name*: Option[string]
    favoriteMovie*: Option[Movie]
{{< /highlight >}}

Then it's just a matter of parsing the json and using the `to` function to marshal into a `seq[Person]`.

{{< highlight nim >}}
import json
import strformat
import options

type
  Movie* = object
    title*: Option[string]
    releaseYear*: Option[int]

type 
  Person* = object
    name*: Option[string]
    favoriteMovie*: Option[Movie]

let responseJson = """
[
    {
        "name":"Sergio",
        "favoriteMovie":{
            "title":"Taxi Driver",
            "releaseYear":1976
        }
    },
    {
        "name":"Daniel",
        "favoriteMovie":{
            "title":"Frozen",
            "releaseYear":2013
        }
    }
]
"""
let parsedJson = parseJson(responseJson)
let list = parsedJson.to(seq[Person])
echo $list
{{< /highlight >}}

The output you'll see is as expected.

{{< highlight bash >}}
@[(name: Some("Sergio"), favoriteMovie: Some((title: Some("Taxi Driver"), releaseYear: Some(1976)))), (name: Some("Daniel"), favoriteMovie: Some((title: Some("Frozen"), releaseYear: Some(2013))))]
{{< /highlight >}}
