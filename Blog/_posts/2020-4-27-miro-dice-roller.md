 

---
layout: post
title:  "Rpg Dice Roller for Miro Whiteboard"
comments: true
tags:
    - javascript
    - webapp integration
    - dice
    - miro
    - regex
    - promise
    - random
    - Dungeons&Dragons
---

this story starts, like many others with d&d. Recently my group has been using Discord for calls, a bot for the dice, and Miro providing the Battle-map. 
Miro our interactive whiteboard of choice is fluid, and easy to use. 
In my game its become a solid replacement for Roll20. The one thing it doesn't have yet is dice rolling capabilities. In this blog, I document my path to fixing that.

## Formal Problem Spec

-   I want to create a way to roll dice on the board.
-   Dice should be simple, intuitive, and should do as much maths as possible

## Where I Started

Miro has integration support, this support boils down to:

1.  you make a website, which has the relevant javascript
2.  you tell the whiteboard about the integration
3.  the whiteboard loads your page as an iframe
4.  your loaded page interacts through the exposed Miro API

this means a static site is all that's needed for simple integrations. In this project I use GitHub pages, as its a free and fast way of publishing such a site; its even recommended by Miro.

## Research

Miro's Documentation is passable, it has its flaws like out of date magic incantations. The more infuriating of which was widgetContextMenu is now getWidgetMenuItems.

there is a public code repository of the Miro library, which is invaluable. But the examples provided felt skin deep. 
the button examples where all on the same type of button, which was a start but added overhead to the research.

## Smallest Viable Product
in the spirit of prototyping and iterative design, I started by designing the least I needed to do to make it work. After some head scratching I made a regex to pull stings that looked like notation from the board. I used a development board to play with the SDK (software development kit). This let me iterate through functional prototypes, not worrying about custom buttons. I realized that trying to find notation with a regex would get to complex to be useful. So as a simplification i chose to use some form of brackets. To begin with I chose an asterisk, I later changed this to actual brackets to fit with rpg content.
The next big break I had, I found a library for rolling dice from rpg notation. This made my life a lot easier as all I needed to do was plug it in. 
One aspect that may stumble, the SDK is asynchronous. Which in javascript boils down to making promises or marking functions as async. Promises are objects that will contain a value... At some point. They have handy functions that tell a promise what to do with a value it gets, `then(func())` is my most used function. You make a promise that the computer will get a value **then** once it gets a value it should run the function you give it. Then is also returns a promise meaning you can compose a chain of actions. A chain that should happen when the promise's value shows up to party. 

## I've proved its possible 
the next thing i figured was most value for least work, make it an actual app. To do this, Miro has a developer dashboard section. I created a new app, now armed with a client id I can make a full extension. Pulling the boiler code from Miro's site, i uploaded a static site to Github Pages. Pointing my integration at it and hitting the board install button I verified it worked. 
so the next feature the buttons. Miro lets you define extension points to add buttons to the relevant menus Miro has. Miro's context menu invoked with the spell "getWidgetMenuItems" to work. The spell appears to have changed recently. The visual documentation of all the extension points used the wrong incantation. This threw me off for a while. 
the next snag I hit, was that the context menu. The event handler only hands me partial representations of the objects selected. These partial objects contain the id of the object and a few other things but, it does't have the text of the object. This means i have to also request the object after I'm handed the partial object. This has an interesting unintended benefit, the rest of the code gets executed in a promise. This has a quirk in the Miro stack, that makes the errors more clear. Before they where obscured by the iframe container, now errors point to the problem line. Now I hook the prototype code up to each object found I get handed. `wiget.plainText` I used to start with as `widget.text` often contains html. That confused the prototype, later I switched to `.text` and escaped the text myself. This was to preserve the formatting of the text i was reading.

## what we have now
a button that will get the text of a Miro object, pull dice notation between brackets, attempt to use it to roll. Then insert the result where the notation was, keeping the notation for future rolls. 

code can be found [here] (https://github.com/megonemad1/MiroDicePostIt)