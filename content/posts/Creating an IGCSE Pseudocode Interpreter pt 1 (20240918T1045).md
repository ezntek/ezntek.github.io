+++
title = "Creating an IGCSE Pseudocode Interpreter (part 1)"
date = 2024-09-18T10:45:48+08:00
author = "ezntek"
tags = ["programming", "school"]
description = ""
draft = false
+++

## Time for the compiler engineer to manifest...

For the longest time, I have loved compiler engineering.

<img alt="A compiler's structure, represented by a cow" src="/img/cowcompiler.png" width="100%" />

It just sounded so mesmerizing. Being able to take code and analyze it in a multitude of ways to generate something based off of logical reasoning...all done through an *automated program*.

The possibilities of a pseudocode runtime is endless, I can even self host a compiler on it, eventually!

## Origins of the idea + Rationale

I take 0478 CS at my school.

My teacher, Let's call him Magic man. Magic man is quite inexperienced for his profession, he is quite good at teaching all topics but actual programming. He is alright at pseudocode, which is why from day 1, he *absolutely obliterated* our innocent minds with this mess:

```
PROCEDURE CanDrinkBeer(Age : INTEGER)
  IF Age > 18
    THEN
      OUTPUT "you can drink beer!"
  ELSE
    OUTPUT "do not drink beer!
  ENDIF
ENDPROCEDURE

DECLARE Age : INTEGER
OUTPUT "what is your age? "
INPUT Age
Result <- CALL CanDrinkBeer(Age)
OUTPUT Result
```

(Without types of course. Let us say that he cannot comprehend those very well).

### Puzzle Piece 1

You can't run pseudocode. That is a given, pseudo is in the name.

Sketchy web interpreters and even desktop interpreters exist, such as [this](https://cs.coursemo.com/igpc/) one and another one called [pcse](https://github.com/virchau13/pcse). However there are some issues.

 1. Normal programming concenpts cannot even be implemented. Since not all data types are first class citizens, code like so won't even work:

```
PROCEDURE BubbleSort(Data : ARRAY[1:5] OF INTEGER)
  FOR i <- 1 TO LENGTH(Data) - 1
    FOR j <- 1 TO LENGTH(Data) - i - 1
      IF Data[j] > Data[j+1]
        THEN
          temp <- Data[j+1]
          Data[j+1] <- Data[j]
          Data[j] <- temp
      ENDIF
    NEXT j
  NEXT i
ENDPROCEDURE

DECLARE Data : ARRAY[1:5] OF INTEGER
Data[1] <- 5
Data[2] <- 1
Data[3] <- 6
Data[4] <- 9
Data[5] <- 4

CALL BubbleSort(Data)
OUTPUT Data
```

In the case of pcse, This code simply complains about a missing assignment operator. Strange.

 2. I am not even sure if the specification was implemented properly, as omitting the parameter list in a procedure or function doesn't even parse properly.
 3. There is a big lack of convenience features. Array initialization syntax, i.e. `Array <- [1, 2, 3, 4, 5]` Doesn't even exist, one must spell out each index and each value.

There are a few more I could come up with, but at the time of writing (I am in class), I can't think of any more reasons.

### Puzzle Piece 2

Cambridge actually has quite a [detailed syllabus](/doc/2023_2025_cs_syllabus.pdf), where in the 4th section of the document there exists a concretely defined but loosely phrased grammar for the language. This would actually be quite convenient, I would be able to have a guaranteed-to-be-correct and accurate reference to write a parser for.

## The end?

This is just the rationale, and the beginning steps. I really want to get an article out, and I have no clue how this project will grow and progress over time.

I have already commenced the creation of a more formal grammar for the language. Stay tuned.

<script src="https://utteranc.es/client.js"
        repo="ezntek/ezntek.github.io"
        issue-term="title"
        label="comments"
        theme="github-dark"
        crossorigin="anonymous"
        async>
</script>
