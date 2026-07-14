June 30, 2026
AI-generated content may be incorrect

Malik Falana
started transcription

Cody Seher
0 minutes 3 seconds0:03
Cody Seher 0 minutes 3 seconds
for the reoccurring UI section as well. So I'm going to shut up now and watch what you got.

Kyle Lawson
0 minutes 13 seconds0:13
Kyle Lawson 0 minutes 13 seconds
Yeah, absolutely. So I'm going to go ahead and share my screen. I'm going to walk you through kind of how we built this. Malik and I just wanted to make sure we had every scenario covered. So what I did was, Patrick, that appendix you provided us continues to be super helpful. So I'll thank you for that again. I took all of the scenarios in the

Cody Seher
0 minutes 33 seconds0:33
Cody Seher 0 minutes 33 seconds
OK, you bet.

Kyle Lawson
0 minutes 38 seconds0:38
Kyle Lawson 0 minutes 38 seconds
And what we did was just make a test record for all of them to make sure that we can have a UI that accommodates for all of these different pieces. And Cody, to spoil it a little bit, we have an example for all of the situations, which includes recurring.

Cody Seher
0 minutes 55 seconds0:55
Cody Seher 0 minutes 55 seconds
Yay!

Malik Falana
0 minutes 57 seconds0:57
Malik Falana 0 minutes 57 seconds
The.

Kyle Lawson
0 minutes 57 seconds0:57
Kyle Lawson 0 minutes 57 seconds
So we have everything. We have the bonus points. So a lot of them are the same as far as the UI set up. There's just a flag that will be checked and stuff like that. So what I was planning on doing is just walking through the scenarios one by one, telling you guys any places where we're still confused. That's like how a variable will be set if we're grabbing it from the contract, that you'll have some context for those questions.
Kyle Lawson 1 minute 21 seconds
Here, when they come up, but okay, cool. Like to do a couple of disclaimers before I run through completely. I have modified the dealer goods and services form layout to kind of make it a lot less busy since we're moving away from it. This is just for me, so dealer goods and services for...
Kyle Lawson 1 minute 41 seconds
All other end users is exactly the same. I made a form behavior specific to not just the default role, but to me specifically. So your user ID has to be klawson at waterdest.com. So in case you're wondering, well, where's all the old stuff? Nothing's broken. It's just hidden in the background so we make things a little prettier.

Cody Seher
1 minute 59 seconds1:59
Cody Seher 1 minute 59 seconds
OK.
Cody Seher 29 minutes 38 seconds
through Aspire without the suspend invoicing turned on. So, you know, go live day one, suspend invoicing may still be a thing, and that's something that you guys will have to address in the post to Aspire initially.
Cody Seher 29 minutes 58 seconds
But then day seven, after we've gone through, you know, the first three that are all wrong, we're going to turn that off and let the spice flow, so to speak. Okay. So that's something that is going to kind of scare them to some degree, something we have to prepare for for
Cody Seher 30 minutes 17 seconds
care after the initial implementation.
Cody Seher 30 minutes 22 seconds
Also, any questions that you have for the end users, put on a slide and let's make sure every single one of them is addressed. And that goes for you too, Patrick. So this meeting is happening.

Kyle Lawson
30 minutes 31 seconds30:31
Kyle Lawson 30 minutes 31 seconds
Got it.
Kyle Lawson 30 minutes 41 seconds
Thursday, right?

Cody Seher
30 minutes 44 seconds30:44
Cody Seher 30 minutes 44 seconds
Thursday at 1230 Pacific. So make sure that we have those slides in play and that little bit of cleanup in play. If we get a relatively clean feedback, then that's what's going to start the actual testing.

Kyle Lawson
30 minutes 54 seconds30:54
Kyle Lawson 30 minutes 54 seconds
Yeah.

Cody Seher
31 minutes 6 seconds31:06
Cody Seher 31 minutes 6 seconds
So one thing that I will need to say, and if I don't say it, and you guys can, you know, make sure that I do, is that if something technically comes up, that one of these things is impossible. These are all untested through the process at the moment. And so we're going to have another discussion after testing is done.
Cody Seher 31 minutes 25 seconds
To make to confirm that all of these do work, OK?

Kyle Lawson
31 minutes 28 seconds31:28
Kyle Lawson 31 minutes 28 seconds
Gotcha.

Cody Seher
31 minutes 30 seconds31:30
Cody Seher 31 minutes 30 seconds
Okay.
Cody Seher 31 minutes 33 seconds
Any other questions for Patrick and I, or Patrick, do you have any questions for Kyle or Malik? No, I don't know. Okay.

Kyle Lawson
31 minutes 40 seconds31:40
Kyle Lawson 31 minutes 40 seconds
I'm Ogden.

Cody Seher
31 minutes 42 seconds31:42
Cody Seher 31 minutes 42 seconds
Okay, yeah, awesome. Good, good bonus points. Good. You know, that's what I was looking for, bonus points. How that actually works, don't know. You know. There's a...

Kyle Lawson
31 minutes 43 seconds31:43
Kyle Lawson 31 minutes 43 seconds
Cool.
Kyle Lawson 31 minutes 55 seconds
It, it does, it does. Don't worry, don't worry.

Cody Seher
32 minutes 1 second32:01
Cody Seher 32 minutes 1 second
There's an engineer that once told me something to the effect of that, you know, everyone in life, everyone is trying to win points, but what it goes to, no one really knows. Same thing for projects sometimes, but we'll figure it out. So yeah, that's what we need to have prepared for Thursday.

Kyle Lawson
32 minutes 13 seconds32:13
Kyle Lawson 32 minutes 13 seconds
This is true.
Kyle Lawson 32 minutes 15 seconds
Oh, yeah.

Cody Seher
32 minutes 22 seconds32:22
Cody Seher 32 minutes 22 seconds
Again, I just want to reiterate that.
Cody Seher 32 minutes 27 seconds
they're going to be lost for the first 5 to 10 minutes. Okay. And then they're going to be trying to catch up for the following 10, 20 minutes after that. So it may take a lot of repetition. They may ask the same question that was just answered. You know, all those sorts of things are in place. So you just have to be ready and patient and
Cody Seher 32 minutes 49 seconds
And also, they may come up with something completely that we didn't think of, nothing to worry about. We do want to know that now. Now we want to know that, right? We don't want to know that day one we've implemented and then they roll in and go, hey, what about this obvious thing that, you know, we deal with every single day?

Kyle Lawson
33 minutes33:00
Kyle Lawson 33 minutes
Yes.
Kyle Lawson 33 minutes 2 seconds
Yes.
Kyle Lawson 33 minutes 10 seconds
Yeah.
Kyle Lawson 33 minutes 11 seconds
Yeah.

Cody Seher
33 minutes 12 seconds33:12
Cody Seher 33 minutes 12 seconds
Yeah, that's the failure point. So yeah, just be open and ready for all of those, all of that type of feedback. Okay.

Kyle Lawson
33 minutes 23 seconds33:23
Kyle Lawson 33 minutes 23 seconds
Sounds good.

Cody Seher
33 minutes 24 seconds33:24
Cody Seher 33 minutes 24 seconds
And that's all I have for a sermon today. Okay. Amen. All right. All right. Bless you, child, and have a great day. Bye.

Kyle Lawson
33 minutes 26 seconds33:26
Kyle Lawson 33 minutes 26 seconds
Cook.

Malik Falana
33 minutes 27 seconds33:27
Malik Falana 33 minutes 27 seconds
Mhm.

Kyle Lawson
33 minutes 28 seconds33:28
Kyle Lawson 33 minutes 28 seconds
Amen.
Kyle Lawson 33 minutes 35 seconds
Talk to you guys soon. Bye guys.

Malik Falana
33 minutes 35 seconds33:35
Malik Falana 33 minutes 35 seconds
IT.
