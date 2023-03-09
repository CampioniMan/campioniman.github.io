---
author: "Daniel Campioni"
title: "Translation in Unity"
date: 2023-03-08
description: "Guide to multi-language game support"
tags: ["unity", "translation", "location", "guide"]
thumbnail: "/translation_example.jpg"
draft: true
---

Whenever you start a new Unity project you usually already have some codebase which to build upon, mostly with "Utils" classes/files. In this post I'll talk about a simple one I have implemented for some personal projects of mine, more specifically a translation system (or a Location Provider) using a Google Sheet as the main database.

## Requirements
- Unity 2022+ with an account
- Google account


....


## Improvements
1. Add [Addressables](https://docs.unity3d.com/Manual/com.unity.addressables.html) to the mix!
Quality of life. Things that are directly linked to a ScriptableObject available in your build will be in memory, so every language file will be in memory at all times, not something you want in big projects. To avoid this you can add Addressables to your project and indirectly link the language files in the ScriptableObject, this way you'll need to handle some async calls but it will be worthwhile (maybe take a look at [UniTask](https://github.com/Cysharp/UniTask) if async confuses/scares you).

2. Hide credentials!
If other people are using the same project as you and you don't fancy giving them your secrets/tokens for free you can create a file in your project containing your secrets/tokens and add it to .gitignore, then every time you start the proccess you can get the information from there if the file exists! This way other people that might also want to use the Google Sheet will be able to add their own secrets/tokens file without sharing as well.

3. Save disk space & time with [Protobuf](https://protobuf.dev/)!
Imagine you found yourself using this solution with 80k entries with 10+ words each in more than 15 languages, the language files will start to get to the double or triple digits in megabytes, which would be a huge problem when trying to make things fit in a small app bundle. One of the possible solutions is to use Protobuf instead of JSON, losing the human readability but getting back a lot of wasted disk space.
I haven't crunched the numbers but most probably the conversion time from a .proto file to a C# class is lower than a .json file, also saving some precious load time at the start of your application.

## Questions

### Why Google Sheets?
Free, accessible, very simple to use and understand, with a friendly interface for non-programmers (game designers, artists, producers, ...). What more do you want?

### Where can I find the full implementation?

### What about security?
This whole proccess is Editor only, which means no access to the Google Sheet itself will be available in the final build, no matter the targeted platform.

### Is there a package for .NET? (Unityless)
There are some .NET packages available for it with lots of features if you want to have a Location Provider outside of Unity, like [this one](https://github.com/valdisiljuconoks/LocalizationProvider) for example.
