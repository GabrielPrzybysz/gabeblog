---
layout: post
title: Good Practices to Localize Your Game
date: 2022-01-15 20:32:20 +0300
img: localization.jpg 
tags: [C#, Tools, Python, AWS, Unity]
---


# Good Practices to Localize Your Game

## Intro

Project files: https://github.com/GabrielPrzybysz/liveops-localization-system

**Purpose**

This text aims to talk about the implementation of location systems in digital games. There is no intention to talk about how to write localized text.

**Problem**

Imagine adding thousands of lines and texts from a game with several languages (Russian, Spanish, Portuguese, etc.). A work that this results in is gigantic. The game Frostpunk has hundreds of phrases and several languages, the effort required is very big. Another game in the same situation is Horizon Chase Turbo (I'm currently working on it with the Aquiris team), but it used a solution that makes this task less arduous and less chance of error!

To better understand the problem, see the image.

Here we see the language switch menu. There are several languages to choose from:
![](https://lh3.googleusercontent.com/UzxoVB4ZbXNZaswNNvzn5IaLZmBDSXfhM6UYt1n4WSFO3h1V_TGlOiBdBJmj5XYUEUkUGAhKJVhBFbPyDOo2Xj38GrQLy26S82GB2lX5DAxhiaHDY5JvWzR0U8aj9Jshzapq0fbB)


How does the game dynamically change languages? How does the game change all the text with just one user input? What if developers want to change some phrase in a specific language, a file with thousands and thousands of characters? How to find this particular phrase?

All these problems are smoothed out using the following technique.


**How to Solve That?**

(We performed an implementation in the Unity game development engine)
Several ways to localize our game exist, using .xml, .json files, your file pattern, among many others. Editing these types of files is not intuitive, is repetitive, and with a large margin for error. However, in my opinion, the best way to accomplish this task is what I'm going to teach you here, using Google's great tool, Google Sheets, and .csv files!

**LiveOps**

I'll show you how it's possible to do any text update in a LiveOps way without having to recompile your entire game, create a new build and submit it to some platform, giving us the freedom to update some text in less than 10 minutes, 100% LiveOps.

 We can get an idea of the file path in the following image:

![](https://lh5.googleusercontent.com/YInw_2zXyAbPwmVY9jexFR0LD89ALjRyj8_ynouxtbJONBqAaUo7pbLgaJ6Zz6cE-oPuKo7J4zlk3SecJkOtErnsgLBIvbbToNmb7JMThoEAND84WDYEhUzoQvUI8UiiiSzFTbpI)


**Technologies**

 - Unity 
 - AWS Lambdas 
 - AWS S3  
 - StackPath 
 - C# 
 - Python3

# Why Use this Technique?

1. **Anyone can edit the localization**

Not only programmers can edit localization files. Anyone can it's simple to edit strings within the game. It is no longer a complex and dangerous file. It becomes a spreadsheet that is easy to access and modify.

2. **It is possible to automate repetitive texts** 

With a little more advanced knowledge of spreadsheets and Google Sheet formulas, you can automate various texts. No more writing "speech_99", "speech_100" for your text key, do it with a Google Sheets formula.
 
3. **Finding and editing an error is much easier**

We will see at the time of creating our spreadsheet how to make the margin of error miniscule.
 
4. **Promotional texts anytime**

By adding the LiveOps system, it's possible to build a "What's New" for your game, for example. In which you can put any text. Encouraging the purchase of some DLC or something! All this without any build, recompilation, etc.

# Creating the Spreadsheet
**Struct**
The structure is simple. The "header" of the spreadsheet has the access key and its corresponding languages. For example, see the image:
![](https://lh6.googleusercontent.com/3wIVj2hSFCRoBB60nnHvDi94UC3JBCq0NCjywQM9T9iP2aLIZU3KupqXhs0ICEqTmlHEi4MjPh1h74epX9_NuFJk1Y5lwoJkTgVoWxHekEj_YpmtKhw65gp53m9b9BI9awbr18x9)

**Formulas**

To avoid some errors, some formulas were created to identify them:

1. **Empty Cell** 

An empty cell can harm when our Parser reads the .csv, so, to avoid this type of error, we add a Conditional Format to check if there is any and if so, we paint it red:


![](https://lh4.googleusercontent.com/RFP7NGQNrhzdyjNOwSHryS78Ld1cgNmT5gG6rJ31slV4YuLIjPu-ckw-T9fWkwGZ7mHErmafHZL3qINjE7rya7aCErA0LzRAqRCV6CBkY54AQLtm7rnHevurESX72TLU0CosqxkZ)


2. **Cell with "enter" at the end**

Another thing that can hurt when Parser reads the .csv is the "enter" at the end of the cell. Instead, we use "\n" to prevent another Conditional Format is added:

![](https://lh5.googleusercontent.com/97NkZsf_hjdYyApjrqrHpRhlt5yB0-At8uV7x0r0qa45hBj8U1LIxQ1HYNKsppWCM3gs0L9n5viykEb1GA91JeLeyMlUsbQI-JHYEXZzWfgLZMTnqk1fhQ6o9WvyggOglX5c6e-f)

3. **Cell with space at the end or beginning**

Unnecessary characters are common. Space at the end and beginning of the cell, to prevent another Conditional Format is added:

![](https://lh3.googleusercontent.com/N2mKnHtKjwMeL_spL-n40LwBPlZyeKfudKypLITCc55rjVPwRbJIkKnJCd1NMP-fOaUpKBcE6VCzk0tD2Df1GQIJj4GuiKKeQYKTPtdhH88sScVYa4wm76TOlt9ZRoOe-k1qhgZe) 

**Permissions**

Anyone on the internet with this link can view:


![](https://lh6.googleusercontent.com/4ed_eI57uWsbln_ic4WhZh0qcY1d9jIdj-kNqhMRDtmrrlSvpQwo5dqHNV1T6GiJGtF5QbTGmA0XbuK7OWJsBcgIHIQZLU4sKB_D1xdSzrlI9PMapZ5Y8FXKXtjIoLPA_tBvU3dt)


# Creating Bucket on AWS

This AWS bucket stores the Localization .csv. This .csv comes from a Lambda that we will create in the future, which will feed on our Google Sheets spreadsheet and save the .csv here!


1. Enable bucket ACL

2. Does not block all public access


# Creating Lambda 
After carrying out the previous steps, we need a way to have a version of our spreadsheet in AWS S3 in the correct file format, for we are going to use lambdas AWS

**Why AWS Lambdas?**

-   There’s No Infrastructure to Manage
-   AWS Lambda Has Strong Security Support
-   AWS Lambdas Are Event Driven
-   You Only Pay for What You Use

To create our lambda we use the lambdas system offered by AWS, and python3 (Teaching how to create it in [previous post](https://gabrielprzybysz.github.io/gabeblog/forant/) on "Lambdas" section). The lambda created below can be called by some API or scheduler to constantly update localization.

First of all, the packages used are:
```python
import json
import urllib3
import boto3
 ```

After importing the packages, let's declare some constant variables:

```python
CSV_DOWNLOAD_LINK  =  "https://docs.google.com/spreadsheets/d/{YOUR SHEET ID}/gviz/tq?tqx=out:csv"
 ```
To find your sheet ID look at the image below:
![](https://lh3.googleusercontent.com/b4D4ClvplYQUdtrNTqfEfMeTXrdLgILM9zdGzuf4ULjs58kblv87IxwMCjX3J8ItrkLMuqNoBAm8rAlP1f39od9acxTRCxWCCzJF7WQw8Uj4I-fq8zMimXpW7kZ5hAA6QgWoz4Sd)

Now let's declare the name of the Bucket created on AWS:
```python
BUCKET_NAME  =  'yourgame-localization'
 ```

And finally, the name of the file created inside the bucket (the .csv):
```python
FILE_NAME  =  'localization.csv'
 ```

Now, first of all, let's download the .csv hosted on Google Drive (The spreadsheet itself):
```python
def download_csv():

	http = urllib3.PoolManager()
	resp = http.request("GET", CSV_DOWNLOAD_LINK)

	return resp.data
 ```

And then with this binary data downloaded from Google Drive we will create a new .csv file in AWS:

```python
def create_file_in_s3():

	binary_csv = download_csv()
	s3 = boto3.resource("s3")
	
	s3.Bucket(BUCKET_NAME).put_object(ACL='public-read-write', Key=FILE_NAME, Body=binary_csv)
 ```

# .csv Unity Parser

Now in Unity, we need to find a way to use the created file. It needs to be easily accessible by the UI system. An example: LocalizationController.Localize(string id) - the function that returns a localized string for the current language selected by the player, thus making all texts dynamically change.

To start, we need an async download of our localization file hosted on AWS to occur. For this reason, we created the LocalizationDownloader.cs script.

```csharp
using System;  
using System.Collections;  
using System.Collections.Generic;  
using UnityEngine;  
using UnityEngine.Networking;  
  
public class LocalizationDownloader : MonoBehaviour  
{  
  private const string CSV_URL = "";  
  private static string _rawLocalizationCsv;  
  private static readonly int Loading = Animator.StringToHash("loading");  
  
  [SerializeField]  
  private Animator _loadingAnimator;  
  
  public static string RawLocalization => _rawLocalizationCsv;  
  
  public static bool IsLoading = true;  
  
 void Awake()  
 {  
	DontDestroyOnLoad(this);  
	StartCoroutine(DownloadLocalization());  
 }  
 
 private IEnumerator DownloadLocalization()  
 {  
	using (UnityWebRequest client = UnityWebRequest.Get(CSV_URL))  
	{  
		UnityWebRequestAsyncOperation result = client.SendWebRequest();  
		SetLoad(true);  
		yield return new WaitUntil(() => result.isDone);  
		SetLoad(false);  
		_rawLocalizationCsv = result.webRequest.downloadHandler.text;  
	}
	   
	LocalizationController.Instance.Initialize();  
 }  
 
private void SetLoad(bool isLoading)  
{  
	_loadingAnimator.SetBool(Loading, isLoading);  
	IsLoading = isLoading;  
 }
}
```
So the game does not start without localization. A loading system was implemented, as seen in the previous script.

Now with the .csv string downloaded, we need to handle this data. For this, we will use a library called CSVHelper. It will allow us to work with csv in a more optimized way. The CSVHelper library can be downloaded following the following Microsoft tutorial (dlls can be found inside my project at [git](https://github.com/GabrielPrzybysz/liveops-localization-system)):
https://docs.microsoft.com/en-us/visualstudio/gamedev/unity/unity-scripting-upgrade

With the library installed, let's create a controller to merge this library with the previously downloaded data. For example purposes, the controller implementation used the singleton pattern. I believe it could be done in a "cleaner" way. The script can be found in the repository.

How to merge downloaded data with the CSVHelper library?

```csharp  
  
private void LoadLocalizationFromCSV()  
{  
	 CsvConfiguration csvConfiguration = new CsvConfiguration(CultureInfo.CurrentCulture)  
	 {  
	     HasHeaderRecord = false,  
		 Delimiter = ","  
	 };  
  
	 using (var csvParser =  new CsvParser(new StringReader(LocalizationDownloader.RawLocalization), csvConfiguration))  
	 {  
		 using (var csvReader = new CsvReader(csvParser))  
		 {  
			  try  
			  {  
				  var localizationSheet = csvReader.GetRecords<CSVContent>().ToList();  
  
				  foreach (var rawItem in localizationSheet)  
				  {  
					  _localizedItems.Add(rawItem.KEY, new LocalizationItem(rawItem.en, rawItem.es, rawItem.pt));  
				  } 
			  }  
		      catch (Exception e)  
			  {  
				  Console.WriteLine(e.Message);  
				  throw;  
			  } 
	    } 
	}
}  
```

It needs to be easily accessible by the UI system and simple to use, so now:

```csharp
public string Localize(string key)  
{  
	 switch (CurrentLanguage)  
	 {  
		 case Languages.EN:  
            return _localizedItems[key].En;  
		  case Languages.ES:  
            return _localizedItems[key].Es;  
		  case Languages.PT:  
            return _localizedItems[key].Pt;  
		  default:  
            throw new ArgumentOutOfRangeException();  
	 }
 }
```

Now, with all that ready, let's create the script that will be added to the text element to be located:
```csharp
	public class TextLocalize : MonoBehaviour  
	{  
		 [SerializeField] private string _keyToLocalize;  
  
		  private void OnEnable()  
		  {  
			  gameObject.GetComponent<Text>().text = LocalizationController.Instance.Localize(_keyToLocalize);  
		  }
	}
```

Example of use:

![](https://lh5.googleusercontent.com/eJ_Z9s20lGEWxYZ53U6w_FxLc7e7qgC6IAHwFirqnYass1Y8sTVz2OrYpUtFUupvX1Gc63dBj0V3uuySyfKBZ-2g43SKXQhSrJIm9pJlm15KqUCz4y3reUCjR14dbSbj_JtEOzvB)

# Thanks

With this, I hope to make it easier for game developers to create a localization system for their games. I'm a Computer Science student with experience in the gaming industry, and here I showed some techniques I learned. If you have any questions or improvements, send me an email, or access any of my other social networks (they can be found here in the lower-left corner).

<br><br>
**Author**: Gabriel Przybysz Gonçalves Júnior - Backend Programmer
