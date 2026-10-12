---

layout: post

title:  "Claude code and the vibes"

date:   2026-10-12 12:00:00 -0700

---

I've been fairly down on all things LLM since they were forced on me a few jobs ago. I formed this opinion back when these tools were very much `code completion on steroids` and I never updated that opinion. Time has marched on, I've been using claude code for a few months now and I was wrong to write these tools off so completely. They are not magic, but I now see why so many people think they are.

# Claude is the best IDE I've ever used

I am not a fan of IDEs and when I was learning to code I found the conveniences of the integrated development environment (IDE) to be hinderance rather than a help. In my first comp sci course I was introduced to the Java language and to the netbeans IDE. I got to the point where I was able to build a calculator and a tic-tac-toe game and maybe other stuff (it's been a minute), but at the time I did not understand object oriented design. In fact, the final for the course asked me to write a collection of [getters and setters](https://stackoverflow.com/questions/2649096/explain-to-me-what-is-a-setter-and-getter) for some simple classes which were provided to me and I was unable to do so. I didn't even REALLY understand what getters or setters were. I didn't understand OOP. I was taught to use and IDE and I knew there was a menu item in there which would generate getters and setters for me.  I failed that class along with all but two of my other classmates and while the instructor was awful (and later let go) I came to the conclusion that my choice to use the IDE was a mistake. I had bypassed a friction point which would have forced me to understand what I was doing in exchange for a short term benefit (getting my homework done). Failing that class closed off computer science to me for a number of years until I came back to it in grad school.

Claude and copilot and codex and all the rest are a new generation of IDE and one that I find myself actually enjoying... mostly. The trade off is the same but more extreme. I can have a conversation, ask it to consider a set of possibilities, edge cases, example data, etc... and then when I've filled its context with enough relevant data I then ask it to materialize an artifact and it does and it works... mostly. It's magical. I can ask it to ingest a codebase, have a conversation about how its structured, how to finish it and then have it point out a design flaw which I had suspected had been there. It's sharp and it begins to feel like one of those quasi-sentient computers you see in movies. Except when it's not of course, but it's intoxicating when it works and like other intoxicants; they can be used responsibly or... not.

## An argument for the vibes

If we accept that the models will mostly do what we want most of the time then it can make sense to say `go build the thing` and then later `go fix the problem in the thing`. You're up and running in almost no time and you can iterate on a design as you go. The cost of ideation and prototype development drops to the cost of tokens and that's like $20. I've spent more than that on a single beer! Please ignore what happens to pricing when a market becomes captured. Iteration too is great. Speak and the code change you want manifests in front of you. This all makes great sense if your primary concern is the observable behavior of a block of code. I can see it also making sense to someone who understands their code as output delivered by one branch of the org chart. If you're one of these types then you're probably already familiar with the idea that the code doesn't do what you want because you under specified your design to the developer team, so `hallucinations` are nothing new. If we're moving to a world where natural language is the new abstraction for most development then maybe this period will eventually feel like when python or ruby showed up and made web development possible at a much larger scale than would have otherwise been practical.

# What I like
## Existence proofs

I really appreciate that LLMs are able to provide quick proofs of existence. If you think something is possible then you can ask for a rudimentary version of `the thing`, get something back and then you can test it to see if it works. They're kinda like 3d printers in that if you can imagine something with a reasonable degree of precision then you can make it. The print might not be great. It might snap when you put a little load on it, but it can be enough to prove a point (or counter point) and that can be valuable. Conversations that hinge on something being possible can be resolved quickly and that's nice. Of course you can't do the negative.

## Talking to code

Being able to ask questions of a codebase is neat. Asking for data flow diagrams, references to function calls, test case coverage and so forth are all amazingly useful. So long as you ask for references (and check them) these tools allow for rapid deobfuscation of a codebase that you've never seen before. Being able to ask questions of a filesystem or a document or a set of logs or a binary or any other ordered collection of bits is neat too. Being able to ask natural language questions of existing artifacts is going to open up new methods of testing and experimentation that is probably going to allow for much more reliable code in the long run.

# What I don't like

## Untestable

Claude as an IDE is in all practical terms untestable. It's a cloud saas product that updates constantly, pre-filters my input to the backing LLM, A/B tests me, has downtime, and so forth. I like the product, but the it's inherently fluid and uninspectable. Once we can run models locally this will largely be solved as they can be held constant, denied access to the internet etc... For the moment we're at the whims of product managers who are trying to make their products sticky, and that means constant refinement and constant churn.

## It lies to you

"Hallucinate" might be the preferred nomenclature, but it lies. It speaks untruths for a variety of reasons and even so it still provides value. You ask it to make some code and it does, but every time you ask it to review the code it finds new bugs. So long as there's a bias toward correctness, as there seems to be, this issue can be addressed by just asking it to review itself over and over. It's unsatisfying from an engineering perspective, but it does work. Maybe that's just the moment we live in now.

## It's watching you

I have a lot of tabs going with different conversations in them and when one gets blocked I'll move to another. I can lose track of the time and forget how long some async process may have been running for so, I thought it would be nice to have timestamps. So, I googled around for how to enable timestamps. Found that [no such feature is planned](https://github.com/anthropics/claude-code/issues/2441) and then noticed a day or two later that my claude had timestamps on it's output. Kinda creepy.


# We throwing dice now

We've mapping english language to bundles of code which are in essence the code shaped answer to the prompt and while that is truly amazing it is also truly frightening. These answers are semi-random and they can only be as precise as the imprecise language we use. We're solving problems by throwing dice and praying that they've been weighted to our benefit. 

## Context control

I'm starting to form an intuition of how to use these tools and I've found the most useful control point to be the context that it handed to the LLM. If there is an answer, or relationship, or pattern in the context that makes the objective of a task "easier" then make sure it's in the context window. If there is some answer, or relationship, or pattern that is related, but not useful to the task at hand then it should be omitted. I view this as weighting the dice that are being thrown.

# Equivalence

I want to spend a little time digging into the concept of equivalence. Consider a simple python vowel counter program 

```python
def count_vowels(input_string):
	count=0
	vowels=["a","e","i","o","u"]
	for char in input_string:
		if char in vowels:
			count+=1
	return count
```

There are some potential issues there (looking at you `w` and `y`), but lets consider it correct for the purposes of this blog post. Now consider the function

```python
def cooler_count_vowels(input_string):
	vowels=["a","e","i","o","u"]
	return len([char for char in input_string if char in vowels])
```

which can be tested as equivalent to the former function.
```python
import random, string
def random_word(length):
	return ''.join(random.choice(string.ascii_lowercase) for c in range(length))

for i in range(1000000):
    word = random_word(i)
    if count_vowels(word) != cooler_count_vowels(word):
        print(f'mismatch on {word}')
```

Change that range or rerun it as much as you like until you're satisfied. There are [better ways to assert equivalence](https://en.wikipedia.org/wiki/Formal_equivalence_checking), but these better methods also require use to more formally define what we mean by `equivalence`. Lets come back to that in a minute. Using [python's disassembly module](https://docs.python.org/3/library/dis.html#dis.dis) we can inspect the bytecode that gets executed for each of these functions
```python
>>> dis.dis(count_vowels)
  2           0 BUILD_LIST               0
              2 LOAD_CONST               1 (('a', 'e', 'i', 'o', 'u'))
              4 LIST_EXTEND              1
              6 STORE_FAST               1 (vowels)

  3           8 LOAD_CONST               2 (0)
             10 STORE_FAST               2 (count)

  4          12 LOAD_FAST                0 (input_string)
             14 GET_ITER
        >>   16 FOR_ITER                20 (to 38)
             18 STORE_FAST               3 (char)

  5          20 LOAD_FAST                3 (char)
             22 LOAD_FAST                1 (vowels)
             24 CONTAINS_OP              0
             26 POP_JUMP_IF_FALSE       16

  6          28 LOAD_FAST                2 (count)
             30 LOAD_CONST               3 (1)
             32 INPLACE_ADD
             34 STORE_FAST               2 (count)
             36 JUMP_ABSOLUTE           16

  7     >>   38 LOAD_FAST                2 (count)
             40 RETURN_VALUE
```
and 
```python
>>> dis.dis(cooler_count_vowels)
  2           0 BUILD_LIST               0
              2 LOAD_CONST               1 (('a', 'e', 'i', 'o', 'u'))
              4 LIST_EXTEND              1
              6 STORE_DEREF              0 (vowels)

  3           8 LOAD_GLOBAL              0 (len)
             10 LOAD_CLOSURE             0 (vowels)
             12 BUILD_TUPLE              1
             14 LOAD_CONST               2 (<code object <listcomp> at 0x1023019d0, file "<stdin>", line 3>)
             16 LOAD_CONST               3 ('cooler_count_vowels.<locals>.<listcomp>')
             18 MAKE_FUNCTION            8 (closure)
             20 LOAD_FAST                0 (input_string)
             22 GET_ITER
             24 CALL_FUNCTION            1
             26 CALL_FUNCTION            1
             28 RETURN_VALUE

Disassembly of <code object <listcomp> at 0x1023019d0, file "<stdin>", line 3>:
  3           0 BUILD_LIST               0
              2 LOAD_FAST                0 (.0)
        >>    4 FOR_ITER                16 (to 22)
              6 STORE_FAST               1 (c)
              8 LOAD_FAST                1 (c)
             10 LOAD_DEREF               0 (vowels)
             12 CONTAINS_OP              0
             14 POP_JUMP_IF_FALSE        4
             16 LOAD_FAST                1 (c)
             18 LIST_APPEND              2
             20 JUMP_ABSOLUTE            4
        >>   22 RETURN_VALUE
```
The difference is clear, but does the difference matter? We can go further and get yet more unique functions by changing the list construction or the comparison logic or by adding extra, irrelevant, logic.
Ex.
```python
def less_cool_count_vowels(input_string):
	vowels=["a","e","i","o","u"]
	x=1+1
	return len([char for char in input_string if char in vowels])
```
which disassembles to similar but yet distinct bytecode. Throw that into the disassembler yourself if you don't believe me. Now, some readers may object to this example and claim that it is not equivalent to the others, but this begs the question `what do we mean by "equivalent"?`. We need to be more precise about what we mean by `equivalent` if we want to reject this new function and precision is simply not the current vibe. It turns out that we can construct infinitely many "equivalent" implementations for a given program. For this simple example the variation is fairly well constrained, but as the size of the program grows the variability is bound to grow as well. Flipping that view; given that there are infinitely many implementations of any given program, the proportion of those implementations that conform with our expectations drops as the complexity of the application rises.

I do wonder if this vibe era is going to end with a whole new cohort discovering Haskell and functional programming more generally. We'll see I guess.

For what it's worth I cross checked against chatgpt with the prompt `write me a python function to count and return the number of vowels in a string passed to the function` 

and got 

```python
def count_vowels(s):
    return sum(1 for char in s.lower() if char in "aeiou")

```
then I asked it again in a new private window and got
```python
def count_vowels(string):
    vowels = "aeiouAEIOU"
    return sum(1 for char in string if char in vowels)
```

Does any of this matter? We'll find out.

# Deferred maintenance my beloved

Deploying at the speed of light means also taking on tech debt at the speed of light. Container sets grow. Database tables grow. Cloud infrastructure simply grows with the life of the `Agentic`-ness. Code complexity grows with time as well and that's a bigger issue to me because the time is takes anyone to onboard to a new codebase is a function of the complexity of the codebase (or the bit they care about). Super charging the complexity growth of a codebase is going to mean that you'll need tools to manage that complexity and that is exactly what an LLM IDE is good at. I think it's telling that open source projects are limiting their exposure to these tools.

There is an `Agentic` approach to addressing that tech debt and it is not yet clear if that makes economic sense. It's gonna be interesting to see if those economics work out. 

`Thanks for reading`