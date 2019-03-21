---
title: "Open Source, 2 years retrospective"
published: true
---

I don't really blog much, but I wanted to post this as it's been almost two years since I began working on my favourite OSS software, [**Ombi**](https://www.ombi.io/) (Previously named *PlexRequests.Net*). I thought this would be an excellent time to write a post about what it's been like working in the OSS world for two years.

That's right, nearly two years have passed since the repository was created! It's gone by unbelievably fast and I did not expect to still be working on it at all, much less adding new features.

Let’s begin with a bit of background, I found the original [Plex Requests](http://plexrequests.8bits.ca/) and thought to myself "Wow! this is an excellent idea." I'd been using it for a few weeks, and since I'd wanted to start a new project anway, I figured I'd give it a go, but with C# and .Net rather than Javascript. With the first release, the creator of Plex Requests gave me some helpful feedback on areas to improve.

Originally it was all written in .Net 4.5, and ran in Mono on Linux systems. Now don't get me wrong - Mono is great - but what I found is that with each different version I always had a new issue, and it was difficult to control. With .Net Core, I am controlling the framework I want to use by deploying a “self-contained deployment” (SCD). This means that the end users do not have to install any dependencies, like Mono. That is a HUGE bonus for me.

Today, the tech stack is .Net Core 2.0 with a SQLite db (for multiplatform support) and an Angular 5 frontend. When I started rewriting in .Net Core I started on 1.1 and it was really difficult to get things done efficiently, so I'm glad that .Net Core has now matured and is continuing to mature. 

After a while of getting a bunch of base functionality, I then released version 1.4, now at this point I was amazed since people started actually using it! I remember that the download count hit 1k and I was jumping for joy! Now we have over 5 million downloads!

Since then there have been 30+ releases and more and more users are jumping onboard the Ombi train, and every time someone thanks me or the team for working on this I get that same feeling. 

Now, that was a bit of background information on what I have been doing in the OSS world. So I am going to start with some negatives I have encountered,
One thing I have learned since doing this is that a (semi)popular project could consume your life if you let it. I have already invested a lot of my spare time in the evenings and whole weekends to Ombi, not saying it's 100% a bad thing, but sometimes it's nice to have a small break away from it. For example last weekend I did not touch any Ombi code, but I was still assisting in the support aspect side of things on my phone when there is an issue I try and respond as fast as possible to get the user up and running or to answer any queries. Obviously, this is not a must if you are running with open source projects, but this is more of a personal thing, I take pride in what I develop and I want people using it without any issues.

> I'd like to mention a tweet from Nick Craver ([here](https://twitter.com/Nick_Craver/status/868833079753928704)).

Now that does put things into perspective and I wish others realized this. You get some people that take open source software for granted, now there are many bigger projects than Ombi that are providing a service for free, but in reality, it is not free for the maintainers, there are many costs involved, the biggest being time. Now not saying that everyone has this attitude, but some people do and I wish people would try and see a different perspective, from the maintainer's point of view, we do not get paid to do this.

But on the flip side I have worked with a lot of great people since I started working on Ombi, not just in my repositories but also contributing to other repositories, this way you will always be learning something new, working with different people, they have different styles/practises and it's an excellent experience.