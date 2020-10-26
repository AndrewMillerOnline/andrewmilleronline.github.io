---
title: "SinoCrypt v1.0 Frequency Analysis"
tags: [Python, Matplotlib, Cryptography, SinoCrypt]
style: fill
color: success
description: Establishing a baseline of SinoCrypt v1.0 to benchmark future improvements against
---

## Frequency Analysis

Simple substitution ciphers are traditionally susceptible to frequency analysis.  In frequency analysis, a cryptanalyst will look at the frequency with which each letter occurs in the ciphertext.  In English, certain letters (and groups of letters) occur more often than others.  They can then compare the frequency of the ciphertext to the overall frequency in English to start breaking the code.  It's exactly the basis for starting with RSTLNE in *Wheel of Fortune*.

## Overcoming Frequency

Part of my goal with SinoCrypt was to allow for one-to-many substitutions.  The letter B can be encoded as 了 or 人 or 力 (or any of the 77 Hanzi characters comprised of two strokes), each of which can only be decoded as a B.  By giving the algorithm such a large pool of choices for each substitution, defeating frequency analysis should be easy.  That said, I initially limited most substitutions to a pool of 3 choices, with some letters getting 10 choices.  This was due to A) time constraints, as I found copy and pasting from HanziDB rather tedious, and B) the fact that while the Chinese language has some 77,000 unique characters, not all are in common use today.  Creating a ciphertext that is a garbled mess of meaningless characters will stand out, which I'm trying to avoid.  Ultimately, I'd like SinoCrypt to be sort of a linguistic steganography, with the algorithm capable of choosing character substitutions that make sense semantically.  So long story short, version 1 had fewer substitution choices that were limited to only the most commonly used characters.

## Assessing Frequency

To figure out how version 1 stacked up, I created a Python script to first calculate the frequency of each character in a string.
```python
def countCharacters(text):
    blankDict = dict()
    freqdict = dict()
    textLength = len(text)

    # Count the number of unique characters
    for i in text:
        if i in blankDict.keys():
            blankDict[i] += 1
        else:
            blankDict[i] = 1

    # Determine the frequency of each character
    for k, v in blankDict.items():
        freqdict[k] = round((v / textLength) * 100, 2)

    # Return a sorted list in descending frequency
    return(dict(sorted(freqdict.items(), key=lambda item: item[1], reverse=True)))
```

I then used Matplotlib to generate some pretty graphs demonstrating the frequencies.  The image is below, the [full code is on GitHub](https://github.com/AndrewMillerOnline/sinocrypt/blob/main/freqAnalysis.py).

![](/assets/sinocrypt-v1-frequency-analysis.png)

## Results

Analyzing the frequency data shows that v1 was moderately successful in its goal of overcoming frequency analysis.  The one-to-many substitution significantly increased the spread of the plot: while the original plaintext only had 18 unique characters, the encoded text had 46 unique characters.  Three characters repeated 4 times, two repeated 3 times, ten repeated 2 times, and the rest had only a single occurrence.