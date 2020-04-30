---
layout: post
title:  "D&D Coin Converter"
comments: true
tags:
    - python
    - web scraping
    - Dungeons&Dragons
---
so this project starts like many others. With a slight annoyance and enough free time to hack together a solution. todays problem arose whilst playing Dungeons & Dragons. I wanted to buy a shops worth of pastries; being D&D these where of course the most evil and unholy of pastries. The Shopkeeper asked me to pay, I know weird how even in fantasy we bring our problems with us. on the plus side in this game coin was easy to come by, but that poses other issues. the conversion rates of each coin while consistent, never seem to stick with me. I struggle to convert copper, electrum, silver, gold, and platinum in my head. I also had a lot of platinum, the big bucks coin. but no copper, which I needed to buy simple goods like pastries. so there I am, in the middle of the game doing fast money conversion trying to wok out change. the online sheet I use kindly tells me the worth of each coin, but doesn't work it all out for me.

### the problem

I wanted to plug in the money I have, then tell it what change I want to make. It should spit out the new coin total I should have.

#### so to break it down further:

-   entering the current amount of coin my character owns should be as easy as possible.
-   the coin total I get should result in the optimal change given.
-   it should be simple to continue using the coin total for future calculations
-   readable code

#### nice to have:

-   pull current coin total from the online sheet
-   reuseable code
-   accept alternate currency systems
-   encapsulate functionality in objects

### planed method

so after some thought I decided I could set a map of coins to their value. by asserting that one coin was the lowest value and so was worth 1 in relation to all other coins. convert all coins to this base for use in calculations. so silver coin is worth 10 copper, copper being the smallest I can say copper is worth 1.

this seems so obvious, I'm probably making it more confusing.

so converting a arbitrary pile of coins into copper, how do I tell a computer to do that? simple multiply the number of coins you have with its mapped value. I can even simplify this to a loop as I included copper as 1!

    def denominations_to_base(self, purse):
        cp =0
        for k, v in purse.items():
                cp+=self.currency_index[k]*v
        return cp

is this complete? well it will convert coins it knows about fine, but what if I typo, or I get a coin from a strange and dark land. its to brittle.

how can I fix this? well, for starters the coins I'm handed are less important than the types of coins i know about. so lets start by only looking at those:


    def denominations_to_base(self, purse):
        cp =0
        for k, v in self.currency_index.items():
                cp+=purse[k]*v
        return cp

this is better but still has its issues. now if I'm not handed a coin I know about, the program will crash as it looks for a coin it wasn't given. a simple fix, the computer has to ask itself "do i have a coin type" before it starts trying to do something with it.


    def denominations_to_base(self, purse):
        cp =0
        for k, v in self.currency_index.items():
            if k in purse.keys():
                cp+=purse[k]*v
        return cp

next, I need to get the coins back into separate parts again, to undo the process I have just made. it should be some function foo such that:

    >>> x == foo(denominations_to_base(x))
    True
    >>> ▯

so the approach I took was to work out how many of each coin I could fit into the current total. save it, then adjust the total by that much. some quick code features:
-   I'll need to look at each type of coin and the value compared to the base.
-   i will need to remember current total over the whole loop, so i can reduce the pile as I go.
-   i need to save the coin totals I have worked out for after I'm done looking though coins
-   I want to get_denominations. I'll need to know about: currency_index (the value of each coin) and c (current total coin value).
-   i need to know how many of the coins I'm currently looking at I can convert the current running total to.
-   i don't want this to be a decimal number, half a gold coin shouldn't be 0.5 g

or in code

    def get_denominations(self, c):
        rval={}
        for k, v in self.currency_index.items():
            denom = int(c/self.currency_index[k])
            rval[k]=denom
            c-=denom*self.currency_index[k]
        return rval

this is a simple premise, yet the stated implementation has a flaw. python dictionary's aren't sorted, even if they where it would by key not by value. but the value in this context is the worth of the coin, and for this to work I need to look at the biggest coins first. otherwise in worst case where the base coin is first.... I'll get the current total... entirely in the base coin. who wants £1000 in pennies. to fix this I add a sort function call that looks at the coins' value. in this case I made it negative, as sorted by default makes it ascending. making the value negative flips that to descending. there is probably a fancy python spell I could cast to reverse the order, but *-1 is less googling.

    def get_denominations(self, c):
        rval={}
        for k, v in sorted(self.currency_index.items(), key=lambda item: -item[1]):
            denom = int(c/self.currency_index[k])
            rval[k]=denom
            c-=denom*self.currency_index[k]
        return rval

you may have noticed these are in a class methods. the use of self as a parameter. I did this so I could group the two functions with the currency index. this way I can pass around an object which has final say on how to convert a currency. which means the next we need to work on a place to keep the actual coin values. for this I created another class, this one taking a currency object, and a dictionary of coins. using the currency object I was just handed I convert to coins to their base value then save them. finally I created a function to allow changing the coin amount. for this I created a function that took key value arguments and used them as coin definitions. worked out their value and converted them to the base to adjust my total with.

    def change(self, *args, **kwargs):
        diff = c.denominations_to_base(kwargs)
        if self.value+diff>=0:
            self.value+=diff
            return True
        return False

I also added a boolean return to represent if the change could happen without negative coins. all that's left now is to create a way to print the coin totals to the screen. creating the __str__ python function that calls the currency function solved this.

    def __str__(self):
        return json.dumps(c.get_denominations(self.value))

cool basic functionality sorted.
### Bonus Features
so this magical online sheet I use, has its flaws. but one perk is the character sheets have a JSON representation you can request. it took me a while to work out how to get it though, because the python request library kept getting blocked. I swapped from url open to a base request. I used a application to send a get request (postmaster chrome app) which succeeded then. copied the auto gen-ed python code resulting in:

    if __name__ == "__main__":
        I =''
        c = currency()
        import http.client

        conn = http.client.HTTPSConnection("www.dndbeyond.com")

        headers = {
            'cache-control': "no-cache",
            'postman-token': "84ca471b-4d5d-9cea-2c32-3174eb89e7b4"
            }
        url = f"/character/{int(input('char id: '))}/json"

now I have all the url setup for a specific character and can get their data. this mean I can weave it into the currency system and manipulate it in a looped interpreter. i figured if the input is empty the user wants to pull data and refresh the local values. otherwise they enter space separated amounts like -5gp to adjust the amounts. finally q is quit.

    conn.request("GET", url, headers=headers)
    with conn.getresponse() as w:
        char_data = w.read().decode("utf-8")
        j = json.loads(char_data)
        p = purse(j['currencies'],c)
        print(p)

refresh code breaks down into: send request, then with the response w get the text value and interpret it as json. navigate to the currencies sub object. which almost like I planned it is in the expected format for the currency c to read it. put them both in a purse object together for future use and finally print the contents. next up the adjustments.

        try:
            m = [ re.match(r"\+?(?P<value>-?\d+)(?P<coin>\w+)", x).groupdict() for x in i.split(' ')]
            m = {i['coin']:float(i['value']) for I in m}
            p.change(**m)
            print(p)
        except Exception as e:
            print(e)

this is in a try cause user input.... next I look for all parts of the input that look kind of like they could be talking about currency. in the format {-+}{some number}{type of coin} each found gets added to the list and used to make a dictionary of coins. this is then passed as key value arguments to the change function. and how we coded the change function it will ignore any coins it doesn't recognize. the final loop looks like this:

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
                m = {i['coin']:float(i['value']) for I in m}
                p.change(**m)
                print(p)
            except Exception as e:
                print(e)
        I = input() ### final code all together it looks like:

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
        I =''
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
                    m = {i['coin']:float(i['value']) for I in m}
                    p.change(**m)
                    print(p)
                except Exception as e:
                    print(e)
            I = input()

### final thoughts
in a game with many currencies, it might be cool to have a purse contain many currency objects. then use that to store them on the fly. you'd have to think about how to tell the difference in the change function. having the a conversion from one currency to another could make interesting interactions.