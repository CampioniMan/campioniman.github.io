---
author: "Daniel Campioni"
title: "Translation in Unity"
date: 2023-03-08
description: "Guide to multi-language game support"
tags: ["unity", "translation", "location", "guide"]
thumbnail: "/translation_example.jpg"
draft: true
---

Whenever you start a new Unity project you usually already have some codebase which to build upon, mostly with "Utils" classes/files. In this post I'll talk about a simple one I have implemented for some personal projects of mine, more specifically a translation system (also known as Location Provider) using a Google Sheet as the main database.

## Requirements
- Unity 2022+ with an account (might work on 2021 & 2020, but I didn't test);
- Google account.

## The whole proccess
Let's start by defining which project will be the one to have this new system implementation (new project or an already existing one), I recommend having a project of yours for all the Utils stuff, this way you can import them as a package in the Package Manager. You can also have them separately which is more theoretically correct, but this might be harder to maintain since you'll need to keep checking the repo's name & version of every single package when importing them using the Package Manager.

### The third-party software
Now we'll import the third-party software we need into our project: [Google Sheets to Unity](https://assetstore.unity.com/packages/tools/utilities/google-sheets-to-unity-73410) (GSTU for short) is a nice package from the Asset Store, it's free and enables us to get data from a Google Sheet we have access to, all you need to do is login to your account and get the asset. I'm not gonna go into details about how to setup this package specifically since there's a good enough documentation for you to look at when you download/import the project (in a PDF file), but I'll talk a little about how to use it in the next section.

Some notes I'll add:
- Use the private feature instead of making the sheet public, just to be safe;
- If there are any erros in the Console Window when you import the project just remove the contents from the folder "Google Sheets to Unity/Scripts/v3/", except the TinyJSON.dll. This folder contains the support for an older version of the Google Sheets API, which you won't need;
- You need to set up your Google Sheets API credentials, so read the documentation, it's short.

#### Opening the connection
The connection is not that bad but it is still a sheet connection. To read the contents of your database you'll need to first read it all, yeah, not exactly the most optimized thing in the world, but that's how GSTU works.

{{< highlight csharp >}}

const string SheetID = "...";
const string TabName = "Creative Name";

SpreadsheetManager.Read(new GSTU_Search(SheetId, TabName), UpdateWithGstu);

{{< /highlight >}}

This last line tells GSTU's SpreadsheetManager to search for a tab in a private sheet (TabName and SheetId, respectively), then in this case it calls the method UpdateWithGstu with the data it found, which later I'll talk more about. I would recommend taking a look at the "GSTU_Search" constructors, you need to use some extra parameters if you have merged cells for example.

#### Using the data
{{< highlight csharp >}}

using GoogleSheetsToUnity;
using System.Linq;

...

void UpdateWithGstu(GstuSpreadSheet spreadSheet)
{
    List<string> titles = spreadSheet.rows[1].Select(x => x.value).ToList(); // 1 as int, not string
}

{{< /highlight >}}

This is a little example of what you can do, here the variable "titles" contains every entry from the row "1" in the content area. Usually the first row contains titles so this might be useful some day. (1 is the first, not 0, open Google Sheets and see it yourself if you doubt it, GSTU uses whatever convention Google Sheets uses)

### The nitty-gritty
Here I'll talk about the code you need in order to make this thing work.


....


## Improvements
1. Add [Addressables](https://docs.unity3d.com/Manual/com.unity.addressables.html) to the mix!
Quality of life. Things that are directly linked to a ScriptableObject available in your build will be in memory, so every language file will be in memory at all times, not something you want in big projects. To avoid this you can add Addressables to your project and indirectly link the language files in the ScriptableObject, this way you'll need to handle some async calls but it will be worthwhile (maybe take a look at [UniTask](https://github.com/Cysharp/UniTask) if async confuses/scares you).

2. Hide credentials!
If other people are using the same project as you and you don't fancy giving them your secrets/tokens for free you can create a file in your project containing your secrets/tokens and add it to .gitignore, then every time you start the proccess you can get the information from there if the file exists! This way other people that might also want to use the Google Sheet will be able to add their own secrets/tokens file without sharing as well.

3. Save disk space & time with [Protobuf](https://protobuf.dev/)!
Imagine you found yourself using this solution with 80k entries with 10+ words each in more than 15 languages, the language files will start to get to the double or triple digits in megabytes, which would be a huge problem when trying to make things fit in a small app bundle. One of the possible solutions is to use Protobuf instead of JSON, losing the human readability but getting back a lot of wasted disk space.
I haven't crunched the numbers but most probably the conversion time from a .proto file to a C# class is lower than a .json file, also saving some precious load time at the start of your application.

4. Other Google Sheets packages
GSTU has a limitation where it need to read the entire tab of a sheet for it to be able to use the data from it, but perhabs this this is too slow for you, maybe you have tons of data there and this communication takes a lot of time, you could make a package yourself or find another one (probably a paid one) to solve your problem. But at this point you might want to think about other solutions as a whole, maybe everything could be already in your Unity project and people would have to add/commit new entries directly in Unity, throwing the Google Sheet thing away, idk, it depends a lot on what you want and need.

## Questions

### Why Google Sheets?
Free, accessible, very simple to use and understand, with a friendly interface for non-programmers (game designers, artists, producers, ...). What more do you want?

### Where can I find the full implementation?

### What about security?
This whole proccess is Editor only, which means no access to the Google Sheet itself will be available in the final build, no matter the targeted platform. If this is a problem for you, if you want to actually use a Google Sheet during runtime, maybe you should thing twice, maybe even more, and then find another way to do it, this solution would neither be safe nor optimized.

### Is there a package for .NET? (Unityless)
There are some .NET packages available for it with lots of features if you want to have a Location Provider outside of Unity, like [this one](https://github.com/valdisiljuconoks/LocalizationProvider) for example.
