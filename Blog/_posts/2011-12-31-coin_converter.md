---
layout: post
title:  "D&D Coin Converter"
comments: true
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

so the approach I took was to work out how many of each coin I could fit into the current total, save it, then adust the total by that much.
some quick code features:
- ill need to look at each type of coin and the value compared to the base.
- the current total will need to be remembered over the whole loop
- the coin totals i have worked out will need to be saved for after im done looking though coins

    i want to get_denominations. ill need to know about: currency_index (the value of each coin) and c (current total coin value)
        i will need to remember the coins to return, rval
        look at each coin_type and coin_value in currency_index, and for each one:
            i need to remember how many coins of the type im looking at i can get out of c, ill name this denom
            i need to put denom into rval
            i need to remove the value of denom from c for future iterations
        when thats done
        i can use rval as the final coin totals 

or in actual code

    def get_denominations(self, c):
        rval={}
        for k, v in self.currency_index.items():
            denom = int(c/self.currency_index[k])
            rval[k]=denom
            c-=denom*self.currency_index[k]
        return rval

this is a simple premise, however the stated implementation has a flaw, python dictionary's aren't sorted, even if they where they would be sorted by key not by value. but the value in this context is the worth of the coin, and for this to work i need to look at the biggest coins first, otherwise in worst case where the base coin is looked at first.... ill just get the current total... entirely in the base coin.
to fix this i add a sort function call that looks at the item value. in this case i made it negative as sorted by default makes it ascending and making the value negative flips that to descending  

    def get_denominations(self, c):
        rval={}
        for k, v in sorted(self.currency_index.items(), key=lambda item: -item[1]):
            denom = int(c/self.currency_index[k])
            rval[k]=denom
            c-=denom*self.currency_index[k]
        return rval

you may have noticed these are in a class from the use of self as a parameter, i did this so i could group the two functions with the currency index. this way i can pass around an object which has final say on how to convert a currency. which means the next we need to work on a place to keep the actual coin values. for this i created another class, this one taking a currency object, and a dictionary of coins. using the currency object i was just handed i convert to coins to their base value then save them. finally i created a function to alow changing the coin amount,  for this i created a function that took keyvalue arguments and used them as coin definitions, worked out their value and converted them to the base to adust my total with. 

    def change(self, *args, **kwargs):
        diff = c.denominations_to_base(kwargs)
        if self.value+diff>=0:
            self.value+=diff
            return True
        return False

i also added a boolean return to represent if the change could be made without negative coins. all thats left now is to create a way to print the coin totals to the screen. simply creating the __str__ python function that calls the currency function solved this. 

    def __str__(self):
        return json.dumps(c.get_denominations(self.value))

cool basic functionality sorted.
### Bonus Features
so this magical online sheet i use, has its flaws but one perk is the character sheets have a JSON representation you can request. it took me a while to work out how to get it though, mainly because the python request library kept being blocked till i swapped from url open to a base request. i used a application to send a get request (postmaster chrome app) which succeeded then just copied the auto gen-ed python code resulting in:

    if __name__ == "__main__":
        i =''
        c = currency()
        import http.client
        
        conn = http.client.HTTPSConnection("www.dndbeyond.com")
        
        headers = {
            'cache-control': "no-cache",
            'postman-token': "84ca471b-4d5d-9cea-2c32-3174eb89e7b4"
            }
        url = f"/character/{int(input('char id: '))}/json"

now i have all the url setup for a specific character and can get their data, this mean i can weave it into the currency system and manipulate it in a looped interpreter.
i figured if the input is empty the user wants to pull data and refresh the local values. otherwise they enter space separated amounts like `-5gp` to adjust the amounts. finally q is quit.

    conn.request("GET", url, headers=headers)
    with conn.getresponse() as w:
        char_data = w.read().decode("utf-8")
        j = json.loads(char_data)
        p = purse(j['currencies'],c)
        print(p)

refresh code breaks down into send request, with the response w get the text value and interprete it as json, navigate to the currencies sub object. which *almost like i planned it* is in the expected format for the currency c to read it, put them both in a purse object together for future use and finaly print the contents. next up the ajustments.

            try:
                m = [ re.match(r"\+?(?P<value>-?\d+)(?P<coin>\w+)", x).groupdict() for x in i.split(' ')]
                m = {i['coin']:float(i['value']) for i in m}
                p.change(**m)
                print(p)
            except Exception as e:
                print(e)

this is in a try *cause user input....* next i look for all parts of the input that look kind of like they could be talking about currency, in the format {-+}{some number}{type of coin} each found gets added to the list and used to make a dictionary of coins. this is then passed as kv arguments to the change function, and how we coded the change function it will ignore any coins it doest recognize. the final loop looks like this:

    while (i is not 'q'):
        if (i is ''):
            conn.request("GET", url, headers=headers)
            with conn.getresponse() as w:
                char_data = w.read().decode("utf-8")
                j = json.loads(char_data)
                p = purse(j['currencies'],c)
                print(p)
        else:
            try:
                m = [ re.match(r"\+?(?P<value>-?\d+)(?P<coin>\w+)", x).groupdict() for x in i.split(' ')]
                m = {i['coin']:float(i['value']) for i in m}
                p.change(**m)
                print(p)
            except Exception as e:
                print(e)
        i = input()
### final code
all together it looks like: 

    import json
    import math
    import urllib.request
    import re

    class currency:
        def __init__(self, currency_index={'cp':1, 'sp':10, 'gp':100, 'ep':50, 'pp':1000}):
            self.currency_index=currency_index
            
        def denominations_to_base(self, purse):
            cp =0
            for k, v in self.currency_index.items():
                if k in purse.keys():
                    cp+=purse[k]*v
            return cp

        def get_denominations(self, c):
            rval={}
            for k, v in sorted(self.currency_index.items(), key=lambda item: -item[1]):
                denom = int(c/self.currency_index[k])
                rval[k]=denom
                c-=denom*self.currency_index[k]
            return rval

        def get_empty(self):
            return {k:0 for k in self.currency_index.keys()}

    class purse:
        def __init__(self, coins={}, c= currency()):
            self.c=c
            self.value=c.denominations_to_base(coins)
            
        def change(self, *args, **kwargs):
            diff = c.denominations_to_base(kwargs)
            if self.value+diff>=0:
                self.value+=diff
                return True
            return False

        def __str__(self):
            return json.dumps(c.get_denominations(self.value))

    if __name__ == "__main__":
        i =''
        c = currency()
        import http.client
        
        conn = http.client.HTTPSConnection("www.dndbeyond.com")
        
        headers = {
            'cache-control': "no-cache",
            'postman-token': "84ca471b-4d5d-9cea-2c32-3174eb89e7b4"
            }
        url = f"/character/{int(input('char id: '))}/json"


        while (i is not 'q'):
            if (i is ''):
                conn.request("GET", url, headers=headers)
                with conn.getresponse() as w:
                    char_data = w.read().decode("utf-8")
                    j = json.loads(char_data)
                    p = purse(j['currencies'],c)
                    print(p)
            else:
                try:
                    m = [ re.match(r"\+?(?P<value>-?\d+)(?P<coin>\w+)", x).groupdict() for x in i.split(' ')]
                    m = {i['coin']:float(i['value']) for i in m}
                    p.change(**m)
                    print(p)
                except Exception as e:
                    print(e)
            i = input()

### final thoughts 
in a game with multiple currencies it might be cool to have a purse contain multiple currency objects and use that to store them on the fly, you'd have to think about how to tell the difference in the change function. also maybe having the a conversion from the base of one to the base of another you could have interesting interactions.