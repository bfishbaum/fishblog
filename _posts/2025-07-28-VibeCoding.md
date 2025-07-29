---
layout: post
title: Vibe Coding and Hydra Slaying
---

So I finally "launched" (it's technically up) my first project post Google, [Cryptic Clues](https://bfishbaum.github.io/cryptics), [Github here](https://github.com/bfishbaum/cryptics)
and it's a pretty simple [cryptic crossword](https://en.wikipedia.org/wiki/Cryptic_crossword) puzzle game, but I mostly did it to play around with Claude Code and some infrastructure services.
My design was pretty simple and mostly a copy of a certain cryptic crossword website that I won't list here.

** Claude Code and Vibe Coding **
I was very surprised by the funcitonality of claude code, we had absolutely nothing like it at Google, but I also doubt that with it's context window it would have produced good code that also would have been correct. Using it here is one thing because all of this code must have been written ten thousand times before (or very similar code), but in a more complex code base I'd be curious to see what it does. I think a lot of work I did involved writing helper functions, and being able to just define the function signature and a quick line, but I suspect that it might take longer to guide it correctly to the best answer than it would actually do by itself. I also find the code to be very verbose, which is a form of tech debt I guess? I think I might try and redo the front end because it's not pretty enough for my standards, but I also kinda just want to leave it on the shelf and be done with it.

** Infra work and Hydra Slaying **

I have a working website on Azure, but that's just a simple app with frontend, backend, but no actual database that is persistent, so I wanted to do something a little more complicated infrastructurally. 
I decided to go to AWS just because it seems like the corporate standard, and that began my fight with the Hydra.
In greek mythology, a Hydra was a monster that would grow two heads whenever you cut one off.
AWS is a cloud service provider that requires you to use two new services to fix an error with one.
It took me about a week to actually the whole system working and I figured I'd put my stack here for posterity as well as a tutorial. (at the bottom)

















** Tutorial **

I was using [this tutorial](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-private-integration.html#http-api-private-integration-vpc-link) as a guide, but I didn't want to use their cloud formation yaml they provided because I figured that was cheating and not conducive to learning.

So I started with a Node.js express server in a docker container, and I uploaded that to ECR.
Next I set up a postgres RDS and used environment variables to connect to it. (This caused me a lot of headaches, but was ultimately not that difficult, I just forgot to set the ssl setting to true.)

I had assumed I was able to just directly hit the endpoint of my ECS service, but alas that was not to be.

Instead, AWS requires you to set up an application load balancer (ALB) in EC2 to receive traffic.

* First make sure you have a VPC group and security group that you will be using for the entire project
* Make a target group (this is in EC2 at the bottom)
	* Set the VPC to 
	* Make sure the port specified here is the one exposed by your ECS docker container.
* Set up a Application Load Balancer 
	* Add a listener rule that listens to the exposed port and redirects HTTP(s) traffic to the target group you just made
* Set up a new task definition in ECS with your container settings as follows
```
	"name": "YOUR_CONTAINER_NAME",
	"image": "YOUR_CONTAINER_IMAGE",
	"cpu": 0,
	"portMappings": [
	{
		"name": "CONTAINER_NAME-PORT", # cryptic-container-3001
		"containerPort": EXPOSED_DOCKER_PORT,
		"hostPort": EXPOSED_DOCKER_PORT,
		"protocol": "tcp",
		"appProtocol": "http"
	}
	],
	"essential": true,
	"environment": [ # put your DB environment variables here
	{
		"name": "DB_NAME",
		"value": "postgres"
	},
	]
```
Then start a new service that uses your application load balancer and the new task definition.

Finally you need to make an API gateway that redirects input to your application load balancer, and that API gateway will have a public endpoint that you can hit, and you should finally reach your ECS task. I suggest keeping Cloudwatch logs until you can confirm that you are hitting your service.

Finally you have to make the RDS postgres database, and this is pretty straightforward, just make sure that it has a public IP address and a strong password. (I haven't figured out how to do it without that.)
Then you just put your settings into the environment variabels 
name = postgres (check configuration, its DB name, not instance ID)
user = Master Username
password = master password
host = endpoint
ssl should be true in your express server

That should let your service communicate with your database.

One last tip is that if you want to use postgres admin to manage the database, you need to modify the security group of the database to add a new inbound rule that allows "My IP". That should let you connect and run queries, make new tables etc.


