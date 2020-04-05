---
layout: post
title:  "D&D Coin Converter"
tags:
    - python
    - web scraping
    - Dungeons&Dragons
---
so this project starts like many others with a slight annoyance and enough free time to hack together a solution. todays problem arose whilst playing Dungeons & Dragons, I wanted to buy a shops worth of pastries; *being D&D these where of course the most evil and unholy of pastries*. I was asked to pay, *I know weird how even in fantasy we bring our problems with us* on the plus side in this game coin was easy to come by, but that poses other issues. you see the conversion rates of each coin while consistent never seem to stick with me, I struggle to convert copper, electrum, silver, gold, and platinum in my head. I also had allot of platinum, *the big bucks coin.* but no copper, which surprisingly was needed to buy simple goods like pastries. so there I am, in the middle of the game doing fast money conversion trying to wok out change. the online sheet I use kindly tells me the worth of each coin, but doesn't work it all out for me.
### the problem
I wanted to plug in the money I have then tell it what adjustment i want to make and it spits out the new coin total i should have. 
#### so to break it down further:

- entering the current amount of coin my character owns should be as easy as possible.
- the adjustment made should result in the optimal change given.
- it should be simple to continue adjusting adding further adjustments to the current total coin
- readable code

#### nice to have:

- pull current coin total from the online sheet
- reuseable code
    - accept alternate currency systems
    - encapsulate functionality in objects

### planed method
so after some thought I decided I could set a map of coin to its value, and asserting that one coin was the lowest value and therefore was worth **1** in relation to all other coins.
silver coin is worth 10 copper, copper being the smallest i can say copper is worth 1. 

this seems so obvious, Im probably confusing. but the point is by having this map, i can convert the coins given in to its worth in copper, doing the same with the adjustment value, then good old arithmetic takes over.

so converting a arbitrary pile of coins into copper, how do i tell a computer to do that? simple multiply the number of each coin with its mapped value, i can even simplify this to a loop as i included copper as 1!

    def denominations_to_base(self, purse):
        cp =0
        for k, v in purse.items():
                cp+=self.currency_index[k]*v
        return cp

is this compleat? well it will convert coins it knows about fine, but what if i typo, *or i get a coin from a strange and dark land.* its to brittle.

how can I fix this? well, for starters i care less for the coins i was handed and more for the types of coins i know about so lets start by only looking at those:

        
    def denominations_to_base(self, purse):
        cp =0
        for k, v in self.currency_index.items():
                cp+=purse[k]*v
        return cp

this is better but still has its issues, now if im not handed a coin i know about, the program will crash as it looks for a coin it wasn't given. a simple fix, the computer just has to ask if it has been given a coin type before it starts trying to do something with it.

        
    def denominations_to_base(self, purse):
        cp =0
        for k, v in self.currency_index.items():
            if k in purse.keys():
                cp+=purse[k]*v
        return cp

next, i need to get the coins back into separate parts again, to undo the process i just made. it should be some function **foo** such that:

    >>> x == foo(denominations_to_base(x))
    True
    >>> ▯

so a 

    def get_denominations(self, c):
        rval={}
        for k, v in sorted(self.currency_index.items(), key=lambda item: -item[1]):
            denom = int(c/self.currency_index[k])
            rval[k]=denom
            c-=denom*self.currency_index[k]
        return rval
