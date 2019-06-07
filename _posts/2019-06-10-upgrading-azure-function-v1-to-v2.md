---
title: "Converting a V1 Azure Function to V2"
published: false
---

Recently I converted a V1 Azure Function to V2 and there was a few things that I couldn't find any documentation on, hence reason for this post. Hopefully it can also make someone else's life easier.

The way I did this was in Visual Studio 2019 I created a new Azure Function (V2 .Net Core 2.2) with the inbuilt template along side my V1 function and then ported over the code; this should be quite easy as hopegully your function is pretty small and SOILD :).

