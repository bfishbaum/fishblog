---
layout: post
title: Vibe Coding and Hydra Slaying
---

So I finally "launched" (it's technically up) my first project post Google, [Cryptic Clues](https://bfishbaum.github.io/cryptics), [Github here](https://github.com/bfishbaum/cryptics)
and it's a pretty simple [cryptic crossword](https://en.wikipedia.org/wiki/Cryptic_crossword) puzzle game, but I mostly did it to play around with Claude Code and some infrastructure services.
I have a working website on Azure, but that's just a simple app with frontend, backend, but no actual database that is persistent, so I wanted to do something a little more complicated infrastructurally. 
I decided to go to AWS just because it seems like the corporate standard, and that began my fight with the Hydra.
In greek mythology, a Hydra was a monster that would grow two heads whenever you cut one off.
AWS is a cloud service provider that requires you to use two new services to fix an error with one.
It took me about a week to actually the whole system working and I figured I'd put my stack here for posterity as well as a tutorial.

So I started with a Node.js express server in a docker container, and I uploaded that to ECR.
Next I set up a postgres RDS and used environment variables to connect to it. (This caused me a lot of headaches, but was ultimately not that difficult, I just forgot to set the ssl setting to true.)


