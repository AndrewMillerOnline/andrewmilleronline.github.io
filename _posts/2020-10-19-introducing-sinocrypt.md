---
title: "Introducing SinoCrypt v1.0"
tags: [Python, Cryptography, SinoCrypt]
style: fill
color: success
description: First draft at a simple cipher, written in Python.
---

## Background

I've always had a thing for secret codes. Maybe I was inspired by watching the 1998 classic *Mercury Rising* as a kid. I'm not sure.  All I know is, codes are fun.  So years ago, while I was working as a language analyst for one of those three-letter agencies, I dreamed up my own code.  Recently I took it upon myself to employ my burgeoning Python skills in bringing this code to life.  Thus I present SinoCrypt, a deceptively simple single substitution cipher (try saying *that* three times fast).

## A Code and Some Hints

Before I reveal how it works, why not try to break it yourself?  Here's an encoded message for you.  See if you can determine the original message (which is an English phrase).
```
簿要算题大戳囔整魔是攀察德譬题囊只蹦然馕么翻鬣籍题来翻乙激经说个乙然就馕藩是进说从鬣去嚼器戳题时藤乚新乚灌是下乚道道馕嚼翻是霸新整所乚模魔







```

<details>
    <summary>Hint #1</summary>
        <figure>
            <pre>
                Both the plaintext and ciphertext contain 69 characters.  Whitespaces, punctuation, numbers, etc are dropped.
            </pre>
        </figure>
</details>

<details>
    <summary>Hint #2</summary>
        <figure>
            <pre>
                While I can encode and decode programatically, I could also do both manually.
            </pre>
        </figure>
</details>

## How It Works

The quickest way to break SinoCrypt is with an understanding of Hanzi, or Chinese characters.  While there are over 81,000 unique characters in the Chinese language, each are composed of indivdual pieces called radicals; of which, there are only 214.  Each character is then comprised of one or more radicals.

Each of these radicals can then be broken down further into strokes (think brush strokes in calligraphy), with each radical having a precise number of strokes which are written in a specific order.

![](http://bishun.strokeorder.info/characters/162852.gif)

Sinocrypt uses this concept of strokes to substitute plaintext English to Chinese, based on the number of strokes.  A, being the first letter of the alphabet, is assigned a value of 1 and is rendered as a character with 1 stroke.

| Plaintext | Number of Strokes | Ciphertext |
| --------- |-------------------|------------|
| A | 1 | 一 |
| B | 2 | 十 |
| C | 3 | 子 |

## Increasing Randomness

However, a simple one-for-one substitution is vulnerable to frequency analysis.  In order to add randomness, and because there are a huge number of characters with 1-26 strokes, I used a dictionary to randomly choose a character with the appropriate number of strokes.

## Enough Talk, Let's See Some Code

```python
import random

codex = {
        'a': ["一", "乙", "乚", "乚", "亅"],
        'b': ["了", "人", "力", "十", "又", "二", "九", "八", "七", "厂"],
        'c': ["个", "上", "大", "也", "子", "下", "之", "么", "小", "已"],
        'd': ["不", "中", "为", "以", "方", "天", "分", "心", "开", "从"],
        'e': ["他", "们", "出", "可", "对", "生", "发", "用", "去", "只"],
        'f': ["在", "有", "地", "会", "而", "那", "自", "年", "过", "后"],
        'g': ["我", "这", "来", "时", "你", "作", "里", "没", "还", "进"],
        'h': ["的", "和", "国", "到", "所", "事", "经", "法", "学", "现"],
        'i': ["是", "说", "要"],
        'j': ["能", "家", "都"],
        'k': ["得", "着", "理"],
        'l': ["就", "道", "然"],
        'm': ["想", "意", "新"],
        'n': ["管", "算", "需", "精", "察", "境", "愿", "模", "疑", "演"],
        'o': ["题", "德", "影"],
        'p': ["整", "器", "激"],
        'q': ["藏", "戴", "翼"],
        'r': ["翻", "覆", "鹰", "藤", "鞭", "藩", "鳍", "蹦", "襟", "戳"],
        's': ["警", "爆", "颤", "疆", "攀", "蹲", "藻", "簿", "瓣", "孽"],
        't': ["魔", "籍", "耀", "灌", "嚷", "壤", "譬", "躁", "馨", "嚼"],
        'u': ["露", "霸", "蠢"],
        'v': ["囊", "懿", "镶"],
        'w': ["罐", "麟", "攫"],
        'x': ["矗", "鑫", "衢"],
        'y': ["囔", "馕", "鬣"],
        'z': ["蠼", "鱲", "鱵"]}

# Function that will replace a single english character with the appropriately coded chinese character
def encrypt(letter):
    # Choose a random character replacement from the correct list
    rand = random.randint(0, len(codex[letter]) - 1)

    # Return the encrypted character
    return(codex[letter][rand])
```
To encode, we then loop through the input
```python
        plaintext = input("Enter message to encrypt: ")
        encryptedMessage = ""

        # Loop through the plaintext string to build the encrypted message
        for i in plaintext:
            #drop punctuation, spaces, special characters
            if i.isalpha():
                encryptedMessage += encrypt(i)
    
        # Print the encrypted message to the screen
        print(encryptedMessage)
```

[See Source on GitHub](https://github.com/AndrewMillerOnline/sinocrypt)

## Roadmap
This made for a decent first version, but there are improvements that can be made.
* I need to add more character choices for most of the letters.  It was very time consuming to page through Hanzidb's website to find the most commonly used characters for each stroke count, so I didn't.  I'm working on a webscraper to gather the rest for me.
* `random.randint()` is not cryptographically secure, and resulted in a number of repeated characters in the v1 example.  Specifically, the portion 乚道道馕 is a chunk that most codebreakers would recognize as 'ally' in a heartbeat.
* Add NLP.  As it stands, the generated cipher is complete gibberish linguistically.  However, given the number of options for each English letter substitute, I feel like if I scraped enough Mandarin I should be able to generate syntactically correct ciphertext.  However, I don't know really anything about NLP or ML yet, so this might not actually be possible.