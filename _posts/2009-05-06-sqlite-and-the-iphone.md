---
layout:             post
title:              SQLite and the iPhone
date:               2009-05-06 12:48:00 -0400
tags:               
---

I'm pretty new to iPhone development, so when I ran into a problem with implementing an iPhone app with a SQLite database, I naturally went to Google for answers. Alas, Google did not have the answer for me! If anyone else runs into this problem, hopefully I can help them solve it.

## The Problem

> "EXC_BAD_ACCESS": "Unable to disassemble sqlite3Prepare."

This happened when calling sqlite3_prepare_v2 on my sqlite3 database.

## The Reason

It seems that an "Unable to disassemble..." error occurs when the database hasn't been opened before trying to use it. I already called `sqlite3_open` on the database, so that baffled me.

It turns out my real problem was with the fact that the app code needs to make sure that the database has been copied to the correct location. You may be including the database in your Xcode project, but that's not quite enough.

## The Solution

Before opening your database, check that it exists in the correct location. 

`[NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0]
` returns the directory your database should be in. Use NSFileManager to find out whether the database exists at that location.

I had to copy the database from the app bundle to this location if it wasn't already there (basically meaning that this is the first time the app has been used on the iPhone). The database is located in 
`[[NSBundle mainBundle] resourcePath]`. I used my NSFileManager to copy the database from the bundle to the correct location.

Another option here would be to create the database file from scratch and have some scripts run to setup the database structure.

From here, using my database worked just fine!

## Analysis

Sqlite doesn't error when the database file you're trying to open doesn't exist. It returns an integer error code. I believe the lesson here is to pay attention to error codes when they're provided. It can really help not only with the debugging process, but also with having your application run smoothly even when problems arise.
