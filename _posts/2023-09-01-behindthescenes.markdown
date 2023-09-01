---
layout: post
title: AskFlow Security - My Experience Conducting Penetration Testing for a Friend's App
date: 2023-09-01 20:32:20 +0300
img: askflow.jpg 
tags: [Python3, Cloudflare, Security, Pentest]
---

## *Special Thanks*
[Heitor](https://www.linkedin.com/in/heitor-melegate/)

**Important Notice: Consent and Purpose of Security Assessment**

Before we proceed, it's imperative to stress that all penetration testing activities described in this report were conducted with the explicit consent of the developer and creator of the "AskFlow" application. There is no intention to promote or endorse any illegal activities. Instead, the purpose is to analyze the mechanisms employed by malicious attackers in order to bolster the security posture of the application.

This report is not aimed at compromising the security of our digital lives; rather, it serves as a means of raising awareness and promoting prevention. I reiterate my opposition to anyone attempting to weaken the security of systems. The core objective of this report is to disseminate knowledge about information technology and cybersecurity. Please understand that the content presented here is intended solely for learning purposes and for safeguarding your digital environment.

It's important to highlight that all information shared herein has been disclosed only after the identified vulnerabilities were remedied. The intent is always to foster a safer and more secure online environment for all users of the "AskFlow" application.

Should you have any questions or concerns regarding this notice or the subsequent content, please do not hesitate to get in touch for further discussion.

## Introduction

Well, my buddy **Heitor** just pulled off something pretty cool – he built a fresh new application using the programming language I'm really fond of, Go! Now, because I can't resist getting involved with my friends' projects, I offered to take on the security testing for this one. I've set a challenge for myself: find as many vulnerabilities as I can within 12 hours. And as an added twist, I've decided to do it all manually, just with good old Python.

Oh, here's a neat thing: this is also the perfect opportunity to put my new Linux distro, BlackArch, through its paces! So, about these tests, they're kind of what you'd call "whitebox" – I've got access to all the code and can chat with Heitor whenever I need to, you know?

As my guiding star for this adventure, I'm following the OWASP (Open Web Application Security Project Top Ten) methodology, which fits perfectly since we're dealing with a fresh project and it's a web application. My focus is on the key security pillars:

- **Confidentiality**
- **Integrity**
- **Availability**
- **Data Authenticity**

Now, let's get this show on the road – it's game time! 🚀

## Greeting the Target

**Here's where the action unfolds:**

The API is providing data for the question-and-answer website he (Heitor) put together – it's kind of like a Stack Overflow. This gives us a solid attack surface; the application has a login system, posts, users, images, and so on. So, there's quite a bit to explore.

![askflowintro](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/6f5e9b36-e49e-424b-b339-ca6949db14bd)


## First Step

**Initial Approach:** 
When dealing with web applications, my first move is to begin by mapping out the endpoints and user inputs. Typically, my goal is to pinpoint the server's hosting location and the underlying operating system in use. However, in this scenario, everything is running within a Docker environment on my Linux machine.

## Exploring the Endpoints:

Let's delve into this pivotal phase. As we're conducting a "whitebox" penetration test, we have a clear view of the inner workings. When we navigate to the `resources.go` class, we encounter a comprehensive compilation of all the endpoints the application offers. Each of these access points represents a specific functionality within the application, be it actions, requests, or displays.

![Capturar](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/2087073c-c905-441c-9d22-2ea748133e4a)

This list of endpoints serves as a map guiding our testing journey. Each one of them is a potential entry point, and our goal is to meticulously explore and scrutinize each one. This in-depth analysis will enable us to understand how

---

## Exhaustion DoS (Denial of Service by Exhaustion):

**What is an Exhaustion DoS Attack?**

An Exhaustion DoS (Denial of Service) attack is a type of cyber attack designed to overwhelm a system, service, or resource, in order to render it inaccessible or non-operational for legitimate users. This attack type capitalizes on the target system's limited processing power, memory, or bandwidth, depleting these resources until no further capacity remains to fulfill genuine requests.

There are several forms of exhaustion that can be used in Exhaustion DoS attacks:

1. **Bandwidth Exhaustion:** In this method, the attacker floods the target with an excessive amount of traffic, consuming all available bandwidth. This results in the system's inability to process legitimate user requests.

2. **Computational Resource Exhaustion:** Attackers may send requests that demand intensive processing or memory from the target system. This causes the system to allocate most of its resources to process these requests, leaving little to none for legitimate users.

3. **Connection Exhaustion:** In this scenario, the attacker tries to occupy all available connections in the target system. For instance, in a web server context, this could involve opening a large number of simultaneous connections, depleting the server's capacity to accept new connections from legitimate users.

4. **Specific Resource Exhaustion:** Some systems possess specific resources that can be targeted in exhaustion attacks, such as IP address pools, routing tables, packet buffers, and more. By depleting these resources, the attacker can disrupt the normal operation of the system.

5. **Authentication Exhaustion:** Certain systems require authentication for granting access. Authentication exhaustion attacks involve repeated and massive attempts at authentication, which can overwhelm the authentication system and deny access to legitimate users.

### Initial Observations and Vulnerabilities:

One of the first things I noticed was the API's ability to accept strings of any length in fields like login, password, title, and others. Right off the bat, I realized that this feature could indicate an avenue for handling requests of varying sizes. This is how I pinpointed one of the initial vulnerabilities, which was related to a potential Denial of Service (DoS) that could impact the service's availability.

Now, our next step is to craft a Proof of Concept (PoC) to validate the existence of the flaw and assess whether it can indeed cause the server to crash. We'll be using the `/login` endpoint as an initial example (subsequently, we'll test the others).

Let's delve into what occurs when we send a POST request to this endpoint:

![login_dos_](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/0df12802-ae07-4458-966e-91e9acd66a68)

 We were able to send it in a very simple way, just requiring the used body, without any additional validation.

### Exploring the Model and Potential Weaknesses:

So, here's the deal: the API is using a default model for all the requests that involve users. Now, that might not make much sense and could cause some response time hiccups, not to mention spilling some interesting beans for the troublemakers out there.

Now, check this out, we can take a few swings to see if there's a shot at a Denial of Service (DoS) attack. But here's a more chill approach I'm putting on the table: let's whip up a massive, random string and chuck it into the email field. Then, when the system goes snooping around for that email in the database, it might get a bit tangled and start maxing out the API in all sorts of ways.


### Digging into Server Stats:

I've got access to the server's statistical data, so let's roll up our sleeves and dive into the metrics. But before we jump into that, let's fire up our Python script:

![python_dos](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/682d46e0-e7f6-4753-a393-e823ba4e5d1c)

During the conducted tests, one of the observed scenarios was the complete exhaustion of server resources, including RAM, CPU, and other essential resources. This occurred due to a resource exhaustion test, where a large number of requests were sent at high frequency to the target application.

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/a60f523e-cceb-429f-ab75-45b29f1308e8)

This was the last screenshot I managed to capture before the server went completely offline. It vividly illustrates the substantial resource consumption that was taking place at the moment of the server failure.


![down_home](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/b6150891-4b94-42fc-abe9-f4c29041a0d2)


And guess what? This goes down the same way for all the other endpoints, with a pretty similar impact.

---

## IDOR (Insecure Direct Object References)

Now, let's get serious and talk about the script kiddies' favorite: IDOR.

**What is IDOR:**
It's a security vulnerability where a clever attacker can mess around with the parameters in a request to access information or functionalities that shouldn't be available to them.

Taking a sweep through the application, we spot the JWT implementation, and that's when I got a bit less confident about the potential for an IDOR in the authentication. But taking a closer look, we realize that the parameter controlling the session is outside the JWT.

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/3e611543-13ae-4e22-9ca7-55aa0bdd6613)

So, simply by tweaking a value in the request and sending any valid JWT, you could play the role of any existing user in the database. 

I am logged in as the user (id: 22):

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/b5a2014a-ed87-4af7-ad07-90bf50952920)


I want to make a post as (id: 12):

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/eb8a400d-14a8-47c4-b1bb-df8d98dfdd8a)

This got us excited for the Proof of Concept (PoC) phase. We're going to cook up a Python script to show how to do this with some class <3 The script is simple, it's going to create a post as if it were user 12:

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/19b642b7-3ff2-44cd-8fac-f25ad8248135)

And voilà, we've made the post!

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/e55169e1-a7d2-4e95-a828-2c98564ef72b)

**Wiping Everything Out:**

Now, in the spirit of flexing our skills, we can also wipe out any post, even if we're not the ones who originally posted it. And guess what? It's super straightforward.

Just send a delete request to the endpoint /v1/post/{id} with the ID you want, no fuss.

**Taking a Peek at Everyone:**

The same concept repeats multiple times. Sending a request to /v1/user/{id}, you can get a load of information about the user, including their encrypted password.

Additionally, I can edit any user's email by sending a request to /v1/email/{id}.

And hey, I won't list everything here, but basically with the same trick, I can change the username, email, and even the avatar of all existing users. This is some serious stuff, my friend!

## Injecting and Collecting

Let's circle back to the action of crafting a post, shall we? Remember that image I showed you earlier? Well, take a gander at the field labeled `imgpost`. This field serves as a cozy little spot where you can slot in a URL. Now, here's the twist: this URL gets generated at some point down the line and then stashed away in the database. But guess what? You can play around with this setup and inject any URL you fancy. And voilà, anyone who steps onto the site will be greeted by that injected URL.

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/cb26d184-8438-4d76-a408-0bd9d5454ba1)

Can you already sense the excitement in the air?

Now, to explore this glitch in a rather neat manner, you can put together a Python server. This bad boy logs all the visits to the site, noting down the IP addresses and the browsers in use.

Check out this script:

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/eca46086-812d-4751-a930-c7e6d492778f)
![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/432d8d15-a44e-4e13-8c9a-af0dc1e920aa)

And just like that, you're keeping tabs on all the site's activity. Take a look: 

![image](https://github.com/GabrielPrzybysz/gabeblog/assets/45472156/ce10d338-3c2c-42fb-bb0b-e5a09b37a094)


## Open Email API Test

And to cap things off with a flourish, there's a test endpoint—/v1/send/email—just hanging out there without requiring any authentication. What's the catch, you ask? Well, it's like a wide-open gateway for some not-so-friendly endeavors, if you catch my drift.

## Limits to Prevent Request Overload

Check this out: the application isn't keeping tabs on how many requests each user is firing off. And you know what? Without putting a leash on this, it could lead to some major issues. Imagine someone deciding to flood the system with requests non-stop. That could create quite a mess. It'd be wise to set up some limits to keep things from spiraling out of control.

## Lack of Logging and Monitoring

Another thing that's kind of a bummer is the absence of a solid logging and monitoring setup. It's like driving without looking in the rearview mirror. If something goes south, we're left in the dark about what went down. Without detailed event logs and a watchful eye on suspicious activities, we're pretty much left hanging if things heat up. It's like embarking on a journey without GPS—no clue where we're headed and risking getting into a jam.

## Wrapping Up

Just a quick reminder, the application is still taking its baby steps, so it's all good if we're not stumbling upon those super complex bugs. But, with that said, it's always a smart move to address these issues. This ensures the application starts off on the right foot and saves us headaches down the road. 😉



Author: Gabriel Przybysz Gonçalves Júnior - Backend Programmer
GitHub: https://github.com/GabrielPrzybysz/
