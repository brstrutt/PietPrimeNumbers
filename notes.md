So my website needs an icon. There's nothing there right now and I want to change that. But what to put there?

[Piet](https://www.dangermouse.net/esoteric/piet.html)! Pixel art! A match made in heaven!

### Goals
- To create a [Piet](https://www.dangermouse.net/esoteric/piet.html) program that calculates and displays the first 100 prime numbers. Generating prime numbers was one of the first things I did when I started programming so it seems fitting.

- To create a good looking pixel art sprite. I've been dabbling in pixel art so I hope I can come up with something that actually looks GOOD.

- Optional THIRD goal, to create a way to specify the colour palette that Piet programs use. This would make it MUCH easier to create good looking pixel art piet programs. The default piet palette is very clear, but it is way too saturated. I feel like custom palettes would make it a bit easier to create good looking pixel art source code.

### The Piet program
So I want to generate prime numbers. Hmmm. This will be a challenge. I've never worked with a stack based programming language before (except for very basic hello world programs).

#### How to create the source code?
I COULD use some kind of integrated piet development environment. But fuck that. I'd rather use pixel art software. Something like aseprite? Yeah lets use asprite. I already know how it works from pixel art shenanigans.
#### How to run it?
Well first of all I need to find a damn interpreter I can install on Ubuntu. Or maybe I just use a browser based one? Ok yeah I will use [piet.bubbler.one](https://piet.bubbler.one). It has an import feature so I assume I'll be ok with it. If not I should be able to use [gabriellesc's interpreter](https://gabriellesc.github.io/piet/) or as a last resort [npiet](https://www.bertnase.de/npiet/)(the installation of npiet looks scary).

Lets just confirm it works with a little hello world.
"Why hello there" in ascii is "87, 104, 121, 32, 104, 101, 108, 108, 111, 32, 116, 104, 101, 114, 101". So i need to get those numbers on the stack...backwards, then loop through echoing them until there is nothing left on the stack.

Start out by testing just "101", aka "e". Should be a simple `push 2, push 5, multiply, duplicate, multiply, push 1, add, echo`. This worked!
Now...I could do the same thing for each character...OR I could just duplicate the 101, then add 13. And continue in the same vein for everything except the capital and the spaces.

Ok I wrote a version which should push the numbers to the stack. I'll print them later, but for now I can use the web interpreter to see the stack values live. Ok yup the stack has the right values! Now to do a printing loop!

How to loop? We'll need to change the pointer direction based on the top value of the stack.
So something like `duplicate, not, push direction change, multiply, change direction`.
Ok `switch` is easier than `pointer` because 0 means no change and 1 means one change. 

FUN FACT! Operations on an empty stack do NOT just treat the stack as being full of zeros. Good to know!
With this in mind, also it turns out I misunderstood how the CC toggles, I'm thinking the `pointer` command probably IS the right one to use here.
Oh wait I need to run the NOT twice. Else a value becomes 0 and no value also is zero because it's a no-op.
Yup, that print loop worked!

#### Now how to calculate prime numbers?

So, prime numbers are just numbers >2 that don't divide by any prime numbers less than them. There are other potentially more accurate definitions out there, but this one is good for efficient implementation.

The basic plan is simple:
1. Start at 2
2. Keep a list of all prime numbers we have found. It starts empty
3. divide our current number by every number in the list
4. if no divisions had remainder 0, then push our number to the list and add 1 to it
5. repeat until we have however many primes as we wanted in our list

Now...how the fuck do we do this in piet?
The biggest issue is everything lives on the stack, so I can't just have a list safely tucked away, I gotta constantly be tip-toeing around it. Also remember I need to hold onto some kind of LIVE value too.

I'm going to need to make LIBERAL use of the ROLL operator. And duplicate. I can't examine a value without popping it so I'll need to duplicate every time I want to check something.

##### The Stack
So to deal with the fact everything lives on the stack I think it's probably best if I defined some kind of memory map. For example, the live value lives at stack index 1, the list of found primes lives at 2->102.
Then I need to conceptualise blocks of pixels as "functions", where each function's only characteristic is that it starts and ends with the stack values in the right positions. I should prototype this somehow.

##### Fibonnaci
###### Prepare the stack
Lets practice by pushing the first 100 fibbonaci numbers to the stack! This is easier than prime numbers but still needs the stack and active number registers to be reserved/in use.

First, lets push a shittonne of zeroes to the stack. I can work with these later.
Wait lets not do 100. These numbers grow exponentially fast so maybe don't go that high.

But anyway, how the fuck do I push 100 zeroes to the stack? I COULD just waste 100 pixels on this, but a loop would be nicer. How do I LEAVE the loop? I'm thinking try to roll a 0 to the back. If this was a noop then there'll be a 0 at the front, if not there'll be a 1? Oh fuck this is undefined behaviour. WHY DO I ALREADY NEED TO RELY ON UNDEFINED BEHAVIOUR? I'M WRITING THE FIBONNACI SEQUENCE?!

Ok new plan, push the number of repetitions I want to the stack, push 1, roll1, sub 1, check, loop

SO: `push 100, LOOP START, push 0, roll 1x1, sub 1, duplicate, not, LOOP`
Of course a bunch of these "operations" are composite. We can't just PUSH 0, we gotta push 1, duplicate, subtract. Similar nonsense applies to the others.

OK so `roll 1,1` doesn't do it. I need roll `2,1`? Yup. That worked. but slow, but it worked.

###### Calculate the sequence
So it should just be a matter of:
1. start at 1
2. roll by 2, duplicate, roll by 3 twice, duplicate, roll by 2 (this should duplicate the top two items on the stack and put them next to each other.)
3. add
4. roll by 100, -1 to check if we've hit the max. If it gave us a zero then undo the roll and loop

Can I duplicate the top two faster than this?
```
1,2,3,4 -> 1,2,3,4,3,4
DUP
1,2,3,4,4
ROLL 3,2
1,2,4,4,3
DUP
1,2,4,4,3,3
ROLL 4,1

VS
ROLL 2,1
1,2,4,3
DUP
1,2,4,3,3
ROLL 3,2
1,2,3,3,4
DUP
1,2,3,3,4,4
ROLL 3,1
1,2,3,4,3,4
```

Ok the first one is fastest and I don't think there's a faster way. There might be one that uses fewer pixels but eh, no need to optimise THAT much right now.

Ok Rolling by 100 is tricky because it takes up a lot of space pushing a 100 to the stack. I mean it's not TRICKY it's just a tad awkward.

HAHA IT WORKED! only last little issue is the number being checked doesn't get sent back unharmed, it gets eaten when I check if it's the end. That's fine. I can live with that.

Wait no I can't. I need to fix that. I need to NOT consume every number I check.
Wait no I SHOULD consume it...but ONLY if it is a zero. Else I'm just PUSHING fibonnaci numbers to the stack, I'm not cycling them back into the allocated array.

Ok it works. I have two more numbers than I technically wanted, but I think it just means my stack has 2 extra numbers in it. Also I needed one of them anyway to pretend to be the leading zero of the sequence.

Anyway. That's enough fibonnaci. This proves the array system works. Up next is prime numbers...

##### Prime Number Time!

First prep the stack the same as I did in Fibonnaci. Maybe make it look slightly nicer.
`push 100, LOOP START, push 0, roll 2x1, duplicate, not, LOOP`
Then move the `sub 1` to occur when the loop restarts

Now, how the fuck do I do this?
I need to push 2 into the list. The set the active number to 3

So whats the loop?
`check if the active number is divisible by any number in the list`
`if it isn't, we add the number to the list`
`if the list is full we exit`
`else increment the active number by 2 and repeat`

These aren't going to be simple, so lets tackle them one at a time:
How do we check if the active number is divisible by any number in the list?
1. We need to loop through the list of numbers, bailing when we hit a zero.
2. When we hit a zero we need to know how many items we looped through the list so we can unloop it. I'm thinking store this counter at listend + 1? But then we need to roll list length, -1 every time in order to see it? I mean we'll have to do that a lot ANYWAY. Whenever we operate on the list everything else needs to be pushed behind it.

Ok yeah just psuedocode it first. Make life easier.

```psuedocode
MEMORY:
1: current value
2->22: list of found primes (list length 20 for now)
23: distance travelled through the list

LOOP1: divide by every number in the list
	MODULUS
		duplicate top two values (DUP, ROLL3,2, DUP, ROLL 4,1)
		mod
		if 0
			EXIT FAIL (DUP, NOT, POINTER)
	CYCLE TO NEXT NUMBER
		Push the list to the front (ROLL 23, 1)
		Rotate the list ( ROLL 21, 1)
		Update the list count ( ROLL 22, -1, PUSH 1, ADD, ROLL 22, 1)
		if 0 
			EXIT SUCCESS (DUP, NOT, POINTER)
		ELSE
			Pull the active number back to the front (ROLL 23, -1)
			start LOOP1 again

EXIT
	Pull the active number back to the front (ROLL 23, -1)
	
	SUCCESS:
		Push it to the list (DUP, ROLL 3,1, ADD, ROLL 2,-1)
	FAIL:
		Do not push it to the list (NOOP)

	Push the active number away (ROLL 23, 1)
	LOOP2: Return the list to normal
		Grab the list count (ROLL 22, -1)
		IF !0 (DUP, NOT, POINTER)
			subtract 1 (PUSH 1, SUB)
			push the count back (ROLL 22, 1)
			rotate the list 1 (ROLL 21, -1)
		ELSE
			Grab the active number again and RETURN (ROLL 23, -1)

LOOP0: outer loop. Run check division on every active number until we have 20 prime numbers
	check if we have 20 primes yet (ROLL 20,-1, DUP, NOT, POINTER)
	WE DO:
		ROLL 20,1 then FINAL EXIT
	WE DON'T:
		ROLL 20,1 then LOOP1
```


### General notes

I also learned that passing through white allows you to change colour without triggering a command. Might be useful for making actual pixel art. I also learned that the fundamental way the programming language works is very NOT conducive to pretty pixel art. You have to vary both hue AND value in very specific ways to get the functionality you want. This means every operation no matter how insignificant is likely to be made up of two colours that contrast quite strongly. Oooh but for creating letters I can reduce complexity by just using more pixels to represent the numbers. I think the custom palette plan might be a necessity.

Oh here's another consideration. The website symbols tend to be 32x32 or 64x64. This is a definite limitation I'll need to consider. Not sure I can get good looking AND functional into something that small but we'll have to see.

Editing this is fucking difficult. If you just do a big chain of colours, ANY modification to an earlier part of the chain will require rewriting the everything that follows. Using lots of white to break it up? Very recommended.

Hmmmmm. I'm thinking a custom palette which allows for MULTIPLE colours to map to white/black may be a solid plan. That way you can hide the functionality colours and surround them with variety of different white/black colours.

I'm also leaning towards some styling conventions to make reading this easier. For the hello world program I have started using blocks of vibrant red whenever I need to push a value to the stack, arranges into 3x3 squares connected by single pixel blocks in the centre of an edge. I feel this will make reading the numbers easier. It also makes it easier to see where they pushed, and what mathematical functions get applied to them.

Oh shit the favicon is supposed to exist as 16x16, 32x32, AND 48x48 pixel ico files? I didn't plan for this. I'm going to ignore it. We'll see how small I can make this piet program before I start worrying about sizes.

Well it turns out this interpreter DOESN'T TREAT WHITE AS A SINGLE CONTIGUOUS BLOCK. Coool. Don't put turns in the white path, it won't follow them properly if they go anticlockwise.

It should be noted, this image is going to be compressed to shit when it gets displayed in the browser tab. Maybe I can use that to my advantage? It looks like a noisy mess when crisp and clear but when compressed the noise might blend together to make an actually decent image?