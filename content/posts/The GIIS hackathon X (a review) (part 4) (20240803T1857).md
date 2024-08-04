+++
title = "The GIIS Hackathon X - My Experience (the finale)"
date = 2024-08-03T18:57:43+08:00
author = "ezntek"
tags = ["life", "programming"]
description = "Was everything all for nothing?"
draft = true 
+++

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

It was not far less the length of my _schlong_ when it is _active with blood_,
in layman's terms, around 15 centimeters. The diameter was also miniscule, 
coming in at around 3-5 centimeters. Overall, a very depressing lunch, but
the good thing would be that I was able to shove it in my mouth within ~3 bites,
which is great.


## The final coding session

This is where I really felt the energy in the room rising. Tensions were high
amongst _people who came here to chase excellence, not the complementary Mario
Kart stations_, myself included.

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

Not good. So I decided to abuse the API




See, the spoonacular API has this points limit. Not requests, but points.
To this day, I still do not know how these points are calculated, but basically,
if you try to make too many requests, the API would rate limit you and refuse
to give you more recipes, instead asking you to pay some cash to get the more
premium tiers.

![API rate limit](/img/hackathon/apiratelimit.png)



<script src="https://utteranc.es/client.js"
        repo="ezntek/ezntek.github.io"
        issue-term="title"
        label="comments"
        theme="github-dark"
        crossorigin="anonymous"
        async>
</script>
