+++
title = "The GIIS Hackathon X - My Experience (the finale)"
date = 2024-08-03T18:57:43+08:00
author = "ezntek"
tags = ["life", "programming"]
description = "Was everything all for nothing?"
draft = false
+++

## Foreword

Yes, this will take you very long to read because this really should 
have been 5 parts. These reading time calculations are also quite
messed up. Not my fault, click away if you don't have patience :p

## Recap

As of the last post, I was on day 2, right before the lunch break. I had just
(almost) finished writing my article display, and I am now ready to start
connecting the API. Everything else works just fine.

## Lunch

Lunch first before work. However this time, I didnt realyl have too much time
on me to properly enjoy lunch sat down sipping my drink, etc. I really had to
cook.

...well the lunch suited this perfectly. A depressing chicken wrap!

![The food](/img/hackathon/dickwrap.jpg)

It was not far less the length of my _schlong_ when it is _active_;
in layman's terms, around 15 centimeters. The diameter was also miniscule, 
coming in at around 3-5 centimeters. Overall, a very depressing lunch, but
the good thing would be that I was able to shove it in my mouth within ~3 bites,
which is great.


## The final coding session

This is where I really felt the energy in the room rising. Tensions were high
amongst _people who came here to chase excellence, not the complementary Mario
Kart stations_, myself included.

We were to submit our work at 2:15 PM, it was around 1:20 when I returned,
Keep that in mind.

I didn't really have time to deal with any sort of API errors, which majorly
backfired later on. I just worked on my recipe cards just a little further.

As mentioned in the previous article, the paragraph would cut off outside
the recipe bubble, like so:

![The card overflowing](/img/hackathon/cardoverflow.png)

Therefore I had to figure out how to chop it off. `overflow: hidden;` didn't
really work that well, so I had to set the `max-height`, meaning that the page
resizing would break, too.

![Page resizing](/img/hackathon/pageresizing.png)

After that, API hell commenced.

### API hell

I started by making a basic request to

`https://api.spoonacular.com/recipes/complexSearch?apiKey=<API Key Goes Here>&ranking=2&number=50&addRecipeInstructions=true&addRecipeInformation=true`

to get a bunch of recipes in JSON format. They displaed just fine on the homepage.

Then, I got to compiling the API options. List arguments were separated by `,`,
and each option would be in the format `&key=value`.

I started with an empty string:

```ts
let options = '';
```

I began with the dietary restrictions, which was simple:

```ts
let diet = window.localStorage.getItem('diet')
if (diet === null) {
    diet = 'Non-Vegetarian'
}

options += `&diet=${diet.replace('-', ' ').toLowerCase()}`
```

Note that I have to replace `-` with ` `, which is what the API requires.
I set the enum members to enumerate strings in the second post, with dashes
for display purposes. But for querying, I had to do the above. I do also
set a default just in case the user did not set anything prior, but a settings
class would streamline this process of having a default, along with other
null-safety things. I didn't have time to mess with TypeScript (a completely
foreign language to me) at the time, which is why I just stuck with it.

I then did the allergies:

```ts
let intolerances = window.localStorage.getItem('allergies')
if (intolerances !== null && intolerances !== '') {
    let newOption = `&intolerances=${intolerances.replace(new RegExp(';', 'g'), ',').toLowerCase()}`
    options += newOption
}
```

Note the `RegExp` constructor. I had to construct a RegExp class to
properly replace every occurence of `;` with `g` which stood for global,
meaning to replace every occurence in a line, not just the first one.
_(vim users would know)._

I also only do the find and replace only if the `intolerances` variable's
value is set, which honestly could only happen with JavaScript (and therefore
TypeScript). Oh well.

I did the same with the pantry:

```ts
// get pantry from local storage
let includeIngredients = window.localStorage.getItem('pantry')
if (includeIngredients !== null) {
    let newOption = `&includeIngredients=${includeIngredients.replace(new RegExp(';', 'g'), ',').toLowerCase()}`
    options += newOption
}
```

I was then ready to construct the URL:

```ts
let url = `https://api.spoonacular.com/recipes/complexSearch?apiKey=${apikey}&ranking=2&number=200&addRecipeInstructions=true&addRecipeInformation=true${options}`
```

Simple enough. I then wrote some basic code with the fetch API I just learned
(XMLHttpRequest was too old and complicated for me):

```ts
try {
    let response = await fetch(url, { method: 'GET' })
    let result = await response.text()
} catch (error) {
    console.error(error)
}
```

which would save the text to the `result`. Response would return a promise
(a future because I'm a rust dev), and I'd then parse the text and append
every new `Recipe` class to a reactive list (in the try block).

```ts
let results: Array<AnyObject> = JSON.parse(result)['results']
results.forEach((recipeObj: AnyObject) => {
    recipes.value.push(new Recipe(recipeObj))
})
```

All of this would be in an async closure, keep that in mind.

Now, the spoonacular API doesn't actually randomize the recipes that it gets,
only by relevance. I didn't know that, so I set the load more button on the
bottom of the page to just run this callback again, which would result in
the recipes duplicating.

Not good. So I decided to actually clear the array of recipes when the user
hit the load more button, and load, say, 20 more recipes than the previous
number of recipes.

The code should have looked like this:

```ts
let results: Array<AnyObject> = JSON.parse(result)['results']
recipes.value = []
results.forEach((recipeObj: AnyObject) => {
    recipes.value.push(new Recipe(recipeObj))
})
```

but instead, on my initial run, I got ratelimited. I scrambled to make another
account to get a fresh API key, but I got insta-rate-limited once again. One
more key? Ratelimited again.

See, the spoonacular API has this points limit. Not requests, but points.
To this day, I still do not know how these points are calculated, but basically,
if you try to make too many requests, the API would rate limit you and refuse
to give you more recipes, instead asking you to pay some cash to get the more
premium tiers.

![API rate limit](/img/hackathon/apiratelimit.png)

At this point, it was too much for me to handle. It was around 2:13 PM when I
began testing and I really couldn't hit the submission deadline of 2:15.

I approached a volunteer for help. Turns out his name was Tagore, keep that name
in mind. He suggested using tempoary e-mails which would be fair, if only if 
spoonacular allowed you to spam keys with tempoary e-mails. Things were
looking grim, meaning that I had to revert the commit that I worked on back to
the version with duplicating entries.

It was only after the event that I realized that there was some sort of recursion
without a predicate happening, but I scanned through the code not to find anything useful.

My hands were stone cold, I was shaking, jittering and thinking aobut everything
that could go wrong in the next 3 hours, wondering if I had done all this for
nothing. I could barely even type on my keyboard while rushing my submission
into the Google Form because I was so stressed yet pissed that spoonacular really
rate-limited me at this time. Oh well.

## The aftermath (of the coding sessions)

So what have I made? A recipe scroller. You can tailor search results to your
dietary needs and the ingredients you have at home to get a curated list of
food that you could make.

<video controls width="100%">
    <source src="/img/hackathon/fulldemo.mp4" type="video/mp4">
</video>

Cool enough for a small side project, but genuinely, in a professional and
more mature setting in front of more experienced programmers, this would not be
anything. If anything, I genuinely thought I was going to go dead last.

But I kept thinking; maybe all those small children weren't secretly senior developers.

Either way, there was nothing to lose, nothing to give up on the table. I am competing
against small children and a few professional developers, what could they do to me?

These two conflicting thoughts flooded my mind as I went back into ICT lab S3-06,
the very first room I entered in this competition; a day ago I had nothing but now
I have a semi-broken app, and my fate were to be decided.

It was like 2:30, I was ready for the worst.

## The room

The not cold enough air coming out of the air conditioner settled on me as I slowly
inched my way into the room, to pull a red chair with an armrest and a headrest,
the only one of its kind in the lab.

<video controls width="50%">
    <source src="/img/hackathon/pitchroom.mp4" type="video/mp4">
</video>

As I sat there and waited for the judges to discuss something (which I still do not
know), I decided to put on some Wii Shop Channel music on through my right earbud
to try to make the situation less tense.

**You may even join me in listening as I describe the rest of the event.**

<audio controls width="100%" loop>
    <source src="/img/hackathon/shopchannel.mp3" type="audio/mp3">
</audio>

Oh, and before presentations actually started, of course there had to be a false
fire alarm. Only in the Global International Indian School :)

<video controls width="50%">
    <source src="/img/hackathon/firealarm.mp4" type="video/mp4">
</video>

But wait. Take a look at the first video.

You may see a man, a figure wearing a tan shirt with an IdeaPad (ew why not a ThinkPad)
with a slightly goofy face.

Let's zoom in.

<img src="/img/hackathon/judgetagore.png" width="30%" />

This guy would play a pivotal part in my journey, nothing would have ever happened
without the presence of Judge Tagore. More on him later.

## The Judges and the Pitches

There were 2 judges. One had a clean shave and one still had facial hair. Let as call the
one with facial hair "facial hair". The clean shaved one I know the name of, which is why
he will be referred to as Judge Tagore.

Revisiting part 1 and the judging system, this is what it was:

|criteria|marks|description|
|-|-|-|
|Creativity|25%|How innovative it is, and|how well the theme was adapted|
|Technical complexity|25%|The complexity, with comments and documentation|
|Functionality and usability|25%|User-friedliness and real-life applications|
|Pitching|15%|Being able to answer questions regading the code|
|Software-dependent marks|10%|AI would be data representation, Game development would be game design, app development would be the UI, and web dev would be the layout.|

all I had to do is make my product as presentable as possible to the judges and just rely on
trusting myself with the quality of my app in order to not lose that boldness.

I was quite far down the list, but here we go~

### Pitches

Before every pitch, the judges would always tell the room to shut their mouths,
else they would deduct points from the team for being too noisy and disrespectful.

Huge respect to those guys, they tried their best to keep the kids in check and
they kinda did it.

Pitches were done via Apple's AirPlay (proprietary wifi screen casting between
iDevices, basically) to an Apple TV mounted on the back of a massive television,
connected by HDMI. 

![Diagram](/img/hackathon/airplay.png)

They didn't have an HDMI cable available for other devices, so we kind of had
to use the school computer at the desk to pitch to the whole room, else we had
to just present to the judges and two supervising teachers. As I'd soon find
out, everybody who pitched to the audience got to use their public speaking
skills more and draw attention from the whole room. This, along with other
usually leads to people who can AirPlay to get higher scores.

Basically, we _had_ to have some sort of iDevice in order to pitch like a
champ. I genuinely thought the iCult of iDevices didn't extend to my fate
in a hackathon, but it did. I love you Apple, and your overly loyal
customers! You should suck my d-

### Not-so-honorable mention #1

***a scratch game***. I almost laughed out loud when I saw a scratch game. At myself,
I stressed so much about this competition but this scratch game completely got
rid of all my stress. It was a Junior team and they were literally just getting
started, but my mood immediately rose as I came to see a glorious green flag on
the screen with the 3 team members, who basically just dragged blocks around
and relied on how simple scratch is to make a game. Wonderful, I wish I could do that ;(

I posted this on my story:


<img src="/img/hackathon/juniors.jpg" width="50%">

as I waited my turn.

It was some kind of game, I forgot what it was. I just kept listening to Wii
Shop Channel music while waiting my turn, waiting for all the teams' submissions
to fly by.

As I both eagerly and scarily stared at the presenting order, slowly seeing the
contestants present their scratch and thunkable apps, I only became slightly 
more hopeful. 

### Honorable Mention #1

First honorable mention. I forgot this guy's team name (I think Ubistrong? not sure)
but he made a completely bespoke website in HTML, with a little bit of JS
and a lot of CSS. It was not custom and there was no name prefixing (LMAO IMAGINE),
he just copied a lot from past projects, but it was still cool.

It was about water conservation and it was meant to raise awareness on
a core element of life (life on all elements), and he had quite a lot
of information jam-packed inside. The layout and style was very pretty to
look at, and the programmer art was respectable on the background.

Unfortunately I don't have any photos (I have this terrible habit of not
having any) but this one was still very cool.

### Not-so-honorable mention #2

I forgot how many pitches went by after the first one, but this one was also
from a Junior team. I really do not want to bash them, but this one was a
very cursed scratch flappy bird clone. The sprites were (of course) random
PNGs and Emojis (not even programmer art, what a shame), and all the obstacles
were on the floor.

This one barely related to the theme at all, the pitch was terrible, and I 
still will not ever understand the appeal of playing flappy bird when all the
obstacles are on the floor. Oh well, not my fault-

Everything up to this point were Junior teams if I remember correctly, nothing
was cool and my confidence was quite high. A multitude of teams were missing
for no reason whatsoever; the order went absolute bonkers as teams randomly 
spawned in.

Eventually it was my turn.

### My Pitch

I got up from my seat and approached the school computer. It had failed a 
little while ago, it lost power during a pitch and it had been really unstable.

They did warn me that this could very much happen again, but I persisted.

After a few reboots, it appeared to be stable enough for a presentation.

My plan was to run `vite --host` to host my app on the local network, and then
make the iMac visit my server on the local network to get the site. Simple enough.
After a simple WiFi Network change (WHY NOT ETHERNET) it worked like a charm.

My heart was pounding, but I was ready. As Tagore and Facial Hair Man proceeded
to tell everyone to shut their mouths up, I quickly changed the iMac to dark mode
to ensure the color scheme would load properly, and when they were done, I began.

With a bold voice,

> See, there are many elements required for any living organism including humans
> to survive. They include air, (pause) water (pause) and food.
> However, we are not cows or sheep. We cannot simply lie down on the floor and
> munch on grass and call it a day, we require something more than that. We must
> prepare our food in a way that it will taste good to us and also provide
> adequate nutrition to meet our dietary needs.

I opened the minimized browser, and I begain, while worrying about me getting
ratelimited.

I gave them a demo similar to this (I didn't rehearse this one, but I did on
pitch day):

<video controls width="100%">
    <source src="/img/hackathon/fulldemo.mp4">
</video>

Of course I had a commentary going on, with large hand gestures and pointing
to the display to highlight important features, and EVERYBODY seemed to love it.

And about Judge Tagore. His face lit up in awe after I showed him the dynamism
of my program. His eyes broadened line a blooming flower, he attempted to standd up
while in awe. He froze mid-stand with his hand covered and decided to sit back down.

Turns out, he went to call Aditya Bairy, the head of the tech club. While the commotion
in the room grew, my legs started becoming soft due to shock and confusion- was all this
custom CSS that cool? I started making some of the weirdest screams that sounded more
like whines as I stood there shook, Magic Mouse in hand.

Despite how I listed out all the app's shortcomings, like the lack of a search and filter
feature, everybody was still in utter shock. 

A loud round of applause filled the air to the brim, as Aditya approached. Tagore
demanded him give me a demo. Everybody wanted me to play with the app more. I felt
like all the knots tied up in my heart were undoing themselves as I let out
a huge, massive sigh of relief; even. I couldn't believe my eyes.

This had to be one of the highest energy moments in my programming life. Everybody was
convinced I was gonna win; a Junior team of all girls had even started a conversation with me.
They complimented me and said that my app was so cool, and they really were rooting for me,
and they insisted that I would win first place. They even recognized me as the person
who had his computer out coding in the opening ceremony; IT WAS ME writing my blog post,
the first one, the very one that I published a couple of days ago. 

The teachers even asked me which school I went to after I sat back down.

***HAHAHAHAHA.***

I go to OFS, I said, and they were convinced that the teachers would teach us all this at 
grade 9. All the OFSers should know about the tragedies that we encounter. I really should
write about Mr. Hubbard's atrocities. A man who doesn't even know what a function is
who plaigarizes his teaching material from random books who taught us SQL a year early due
to lack of skill was our course leader, one who I had to constantly correct because of
***idiotic*** mistakes like saying that you can **cast** a function to run the code inside,
***not call***.

I really couldn't let Mr. Hubbard's lack of skill ruin this moment. I just told the teachers
that our school does not teach this at all and I had to learn all this myself from the start
of the hackathon. I said that I did programming since 6th grade.

They asked which curriculum we did, I said we did IGCSE and MYP together. They then asked
if I took computer science, I said I did just for the academic credit because I already knew
most of the syllabus. They asked if this was my first hackathon, I said yes, because I
did more niche things and that I prefer to stay in my own cave and code instead of coding
outside. 

My energy level was high, but I couldn't be too hopeful. There were still pitches left.

I actually think that they were much cooler than mine.

### Honorable Mention #2

Immediately after my pitch, came core.

This team is ***cracked***, to say the least.

A group of two quite dark skinned people who go to Singapore Polytechnic. This was their
52nd hackathon. ***FIFTY-SECOND***.

They took gardening, which is the act of maintaining living plants' health (theme) to
a whole new level, They used an esp32 as their µController, loaded **6** AI models on it
and hooked up a camera to detect if a plant were healthy or not. Depending on that, along
with a humidity and temperature sensor all mounted below the plant pot that they 3D printed,
it will water the plant and also send the plant's state to the user via Telegram and
a Telegram bot, all running on the esp32 with network access.

![The product](/img/hackathon/core.jpg)

This project was crazy; Judge Tagore had already been fazed and exposed to skill, so
he wasn't as shocked, but this one was even crazier than mine because Core incorporated
so many fields of Computer Science and even some biology into this hackathon project.

Almost all of this was written in C++, also very impressive.

### Honorable Mention #3

This one is from the Destiute Dingbats, a team actually from our school, those very kids
from places far and wide. They decided to not AirPlay for whatever reason despite having a
MacBook Air on hand, but I digress.

They took the theme and adapted it to a mood tracker/calendar thingy. They interpreted
elements as in elements of life, and made it very simple and minimalistic, stating
life on its core elements. A very neat concept for sure.

![Blurry picture](/img/hackathon/fromplacesfarandwide.jpg)

Again, I forgot what they did exactly, but it definitely had elements of dynamism,
it was very clean and minimal and it actualy functioned quite well. There were no CSS
animations, which was understandable because it isn't easy anyways.

Coming from students who were only taught IGCSE ICT at school (HTML4 and CSS2) and
without knowledge of component based design or any more advanced frontend design concepts,
this was very impressive. They should have done programming outside of ICT before though,
if I remember correctly.

### Honorable Mention #4

This team of 4 made a whole role-playing game, simulating the contsruction of a civilization,
including the process of taking control of the native people, along with other
concepts such as the construction of housing and sustainable fishing and farming
all in this small package. They have had experience doing game jams before as I asked,
but it was still quite cool.

The animations were wonky. The characters just rotated around a pivot situated around
the middle-ish of their body at a smooth rate of 10° ish each side, which did not look
very good. The programmer art was nice though.

![The game](/img/hackathon/godotgame.jpg)

The controls were quite interesting as I saw. The judges interacted with the game a bit
(which was written in Godot unfortunately, I'm using SDL and OpenGL with C next year haha.),
and the demo was quite cool although I did forget many aspects of it. They did also preach
Fish and Chips as a very sustainable food source though.

I dont want to eat bland food despite how sustainable it could be. _Let me eat my
red braised pork and samosas._

### Honorable Mention #5

This team of pure Asian talent cooked together an extremely complicated algorithm to
figure out sleep timings.

![The people](/img/hackathon/asiantalent.jpg)

Given a schedule, the algorithm would find out which sleep pattern (monophasic, biphasic
or polyphasic sleep) the user is best suited to, and a new schedule would be denegrated 
with optimal sleep times and with regard to their activities.

They had an insanely long presentation, which toured each part of the codebase. Unfortunately,
the judges did not really want to look at it for some reason despite all the effort
they put into the presentation, incuding all the math needed to figure out the time complexity.
The C++ they wrote was not the best at all (not expecting anything crazy, no formatter
and no C++ features were even used, it could have easily been done in C), but its still very cool
what they pulled off.

They also wrote a schedule parser which would let you write a custom syntax to describe
your schedule (which would be very annoying to test and implement being a compiler engineer
wannabe), and they even trained a custom GPT-4 to optimize the results of their algorithm
further. Unfortunately, it did not translate too well to their website, as it was mostly
informative, the UI did not look very good and it was quite shotty overall.

## The End?

I nervously waited for the closing ceremony to commence while eating some mystery
potato and chaat in yogurt sauce with ketchup.

![Funny tasting food](/img/hackathon/funnyfood.jpg)

I really didn't think I would win first place because Core did such a good job with
their plant, but maybe I'd win second, or third, or maybe nothing. I didn't know.

The closing ceremony began with speech after speech as I nervously watched as the Juniors
claimed their awards. 

They did have 2 honorable mentions which I both also shouted out, but I forgot which
teams exactly. I waited anxiously as they announced at third place was Code Lovers
(i forgor what they made)

![Third Place](/img/hackathon/thirdplace.jpg)

hats off to them.

And after a little more anxious waiting, came

> And at second place, we have...ezntek!

***oh yes***.

***I won***.

![i won](/img/hackathon/iwon.jpg)

Usually I look like a scarecrow. Dead serious face with perpetually tired looking
eyes and zero smile, but here I couldn't hide it. I won second place while at
my FIRST hackathon! Very nice.

Second place was fair, which meant at first place there had to be...

***CORE!***

Those guys were actually cracked, and they definitely deserved it. Considering
how I rawdogged TypeScript and Vue as a low level systems enthusiast/dev who uses
C and Rust more than anything, and how Core had attended over 50 hackathons
prior to this event, I really did not expect anybody but them to win.

![first place](/img/hackathon/firstplace.jpg)

This is the end of a journey; of random ass roadblocks, 2 rides on the PGLRT that
I am looking forward to not riding again, an API rate limit and Judge Tagore completely
saving me and flabbergasting me with that face. God, you did NOT have to open your mouth
that wide, you could have gotten lockjaw ;)

I thank everyone who made this event possible, including the president Aditya Bairy,
designer Ashwath who demanded an appearance (make sure your friends dont bully
you for demanding for an appearance on this site who noone visits), the VP of the club
whose name I do not know, Judge Tagore and Mr. Facial hair who yelled at the kids,
Akshat and all the other people.

![Ashwath](/img/hackathon/ashwath.png)


Not to mention the MC's which I believe were Twisha Metha and some other guy. Too many
names, SORRY IF I GET ANY WRONG (please correct me in the comments)

(You really should have put your names in some credits sheet so people can shout you out smh)

## Random ass other shit

To the photographer who took this photo, please take a look at this:


<div style="display: flex;">
    <img src="/img/hackathon/secondplacewalking.jpg" alt="Me" width="50%" />
    <img src="/img/hackathon/communistbunny.jpg" alt="Not Me" width="50%" />
</div>

(do not meme me)

## MY USERNAME

ITS EZNTEK. yeah right.

E as in Feed.
ZN as in Zin, with an I. Similar to the Sin in Singapore.
TEK as in Tech.

No capitalization required. Pronouncing it 

\/ɛznt̪ɛk\/ is a bit of a crime. sorry-

<script src="https://utteranc.es/client.js"
        repo="ezntek/ezntek.github.io"
        issue-term="title"
        label="comments"
        theme="github-dark"
        crossorigin="anonymous"
        async>
</script>
