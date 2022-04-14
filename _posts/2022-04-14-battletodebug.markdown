---
layout: post
title: A Battle to Debug
date: 2022-04-12 20:32:20 +0300
img: a_battle_to_debug.jpg 
tags: [Python3, Cloudflare, Unity, C#]
---

# **Intro**
On a Monday afternoon, we received a report from a player. He reported a problem when purchasing a DLC. After that, we started a saga to solve the bug. First, the tester would have to be able to reproduce on our device. Following the player's steps, the bug was quickly reproduced. But and now? Few clues as to what could be happening. There was no way for us to see the logs (the device didn't allow this to be possible in a simple way), which was essential to find a solution!

The first idea we had was to use a ready-made technology like Datadog, Datatrace, or even the Unity solution, "Cloud Diagnostics". But this was all too robust for what we needed, and with an implementation that would take longer than we had. So, together with the team, we came up with a practical and quick solution, which I intend to comment on here in this post.

## Designing the Solution

The idea was that every time the game used Unity's Debug.Log function, it would do a web request to a simple server. When the request arrives on the server. It would create a file with the logs sent in the body of the POST.

We need to design our idea. So we use the perfect tool for that MIRO!

![](https://lh5.googleusercontent.com/iq4NNrqwwm03-nAZ5KFdxYWgGisfWlUWkJ20ZV9W3V2l588PheaSd9G18z0u_hFb7tyQTcb-eOO1wAvSkXNx9iflhlfDZeP03s65Bod7THsCl24XdhF0LHR6QVK97zuMpIyN6o8L)

As we can see in the image, the idea was to be agile, to do everything locally. With that, we would use Cloudflare's tunnel technology, which makes an "internet" connection with our local machine, in a SUPER SIMPLE way.

## *Tunnels*

"In 2018, Cloudflare introduced Argo Tunnel, a  **private, secure connection**  between your origin and Cloudflare. Traditionally, from the moment an Internet property is deployed, developers spend an exhaustive amount of time and energy locking it down through access control lists, rotating ip addresses, or clunky solutions like GRE tunnels.

We built Tunnel to help alleviate that burden.

With Tunnel, users can create a private link from their origin server directly to Cloudflare without a publicly routable IP address. Instead, this private connection is established by running a lightweight daemon, cloudflared, on your origin, which creates a secure, outbound-only connection. This means that only traffic that routes through Cloudflare can reach your origin."

https://blog.cloudflare.com/tunnel-for-everyone/

 - **Creating the tunnel**
 
Entering Cloudflare's Zero Trust area, we can see the Tunnels tab:

![](https://lh3.googleusercontent.com/NRTfoxJMzis28ZtRZgr9tgCAyWnj07JTbjeLQEos3ygvQOAbZC_D_N1X3AF4Q8PILvdQyBHclos6aIViFk3hjMFRQfaEV-vHeUeGZo1K5pcNzkw7pt9ncSY4FGNYyeYGNheWC40D)

Thus accessing the tunnel creation area.

After that, we give our tunnel a name. We install the .msi file provided by Cloudflare to run on our machine that will be the python server. Our "connector" as it is called by Cloudflare. We run the command with the tunnel token to make the connection:

![](https://lh6.googleusercontent.com/WbDY8Dj4GhDWe-3wuDZb4i04S9h3mPIjEvVF4dXufCO9PlyQ1aLXI2kPGNSoEFg_X26n_A6UoVIElO3XfcHekSvUR4GbJQdbArV7n0njU5LM9SFiapnlf7TIdICsijoNJfj3e1AK)

Configure the hostname (Using the same port that we will set on the HTTP server. So that is mirrored) that automatically changes our DNS. And that's it, tunnel created and running.

We used this to make the request reach our python HTTP server, running locally.

## *Python HTTP Server*

We use the HTTP module that exists in python for requests and the JSON module to work with the received body:

![](https://lh6.googleusercontent.com/5JFOs2uL0gV-uWKhXudZaj7PnqQTu1YQgVOYI5dMcHmkk5ZDUWs-JfDR23sauH-oiznBOur38vhFten7UvKhNjTNR_N_FlFRSxm7E4ytt0vfAuqOBQtfqErXaeLx81_-8oEB8AVh)

We implement the do_POST function:

![](https://lh3.googleusercontent.com/1mvjalEOJfEny0GNuX66XT_-mkz5cIEIRIwFJ0B-VNsYbHe1-FWAFGdl36NTi2HCiCBdVoTFBytFHLg3Fg0PyFMbKLCiLeigI2kRMPaskABOpaCsUXAHBrFKN7rHxUbqaI-90nEB)

And we serve using a socket server.TCPServer on the indicated port:

![](https://lh5.googleusercontent.com/0zKAxmRRFdFGZns4OFmVGduJdSoT1U82M-QCA2h6BvfqOyjsL2yYnzja7WDZbZucIasau-kKIa0xSdzEm_mmUBSYyCrd1rzKsvvB0TpFfFTWWXXaicGZZXZZHVF9piO_5xTxwS_H)

So with the tunnel and the server running. Wait for the Unity request.


## *Unity Side*

As seen earlier, each debug session creates a new file in the server folder. We then need to prepare the object that will be in the request body:

![](https://lh6.googleusercontent.com/3eo7knOfmyvL3zFEzD6-Pz83zhPyHwy285jhn_cSe0hXA_0Em6IDYReHZT_k6jAMkswJ8ZsC8guvTEa71JH7GlAy8PWMli4cHGMnasN_taRtHvGk-4kxG40txi9VHr3jElCEtsb0)

That was the object, serialized as a .json

Now let's implement the function that was signed and called every time a Debug.Log() happens:

![](https://lh5.googleusercontent.com/yynx0XDU8gnxbbN_q1iU-Pe-mxxEf1EUtfhJBxJs6S4a_YL-hcy5sEWEdJebzow48Ex9_TDHQv34tNpeBTpeiqLQPUohRDaCzidzDT3WLkZmZoSNv8NYfvZl60XpcIo6zYBhZg0w)

Now in OnEnable let's sign it:

![](https://lh4.googleusercontent.com/ImPjguY7LwMQBuciQEakDB2c_pQozSiueIVoM8-07bwdKp0W31MjpdNfXcIIRJC3yFGwmKr6xs3JxO51m_FwjwFi8eeOKlM37SZLX80fWHGMOOworfIwK3Rp-ikf3WOndK0vtCeN)

And as a good practice let's remove the signature:

![](https://lh5.googleusercontent.com/_qhU48tnje6kn_Ro_GR-cMnO2onBexomZw_rtswE7AP-yyYg5pvhbqlPB-clS45GXOQ3ecWPUZaUBiLzuR8zRBWOSHJSLHLh3J1hAHgN5eV36a6wrVn3G50BHFegDobzEWRAA5is)

We add the script to a GameObject. In Awake we put a "DontDestroyOnLoad(this);"

## That's it

Now we have a link between the device and the developer. We added logs where we thought the error was, did a build, and the good news came, so we saw explicitly what was happening and the reason for the problem. The solution for capturing logs was done in less than two hours. We quickly had something valuable in our hands, something simple and effective that can be reused in any project and situation.

Then I saw a great post for my blog, sharing this idea so that other devs can debug when it isn't possible to get logs more directly.

Author: Gabriel Przybysz Gonçalves Júnior - Backend Programmer
GitHub: https://github.com/GabrielPrzybysz/battle-logs

