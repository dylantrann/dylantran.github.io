---
title: "Anagram Game"
excerpt: "Displays anagram, player guesses original word.<br/><img src='/images/anagram.png'>"
collection: projects
---

# ___agram
Joint project by Benny Xie and Dylan Tran for ACM DiamondHacks 2024 \
Link to GitHub Repository: [Anagram Game](https://github.com/dylantrann/anagram_game) \
Our submission video: [Team Dylan&Benny](https://www.youtube.com/watch?v=Hivj5OwqLeg)

## Game Concept
A word is displayed as an anagram and the user attempts to guess the original word in as little tries as possible.

The player can either choose for the word to come from a predetermined word bank associated with a category, or they can input their own word.

## Running
After compiling, use the following to run the program:

```
java AnagramGenerator.java <category>
```

The argument `<category>` isn't required, but will allow the player to access words from a determined category. Categories themselves are text files that can be created by the user. The text file's name determines the category and each line in the text file represents a word that can become the chosen anagram.

* If the player decides to pick a category, a random word will be chosen from the category's word base to become an anagram.
* If the player decides to input their own word to play (i.e. they choose not include any arguments), the program will automatically prompt the user to input their own word of choice. 

## Anagram Development
Uses Anagramica API to generate sensible anagrams. Anagramica doesn't just scramble letters, it looks for a valid word given it's dictionary. 

Anagramica has a terrible flaw where it will often return nothing due to it's simplistic query handling. To have a better success rate and allow for more query options, we developed a randomizer that is able to split the single query into two unique queries and scramble the letter orders. This allows for previously failed queries to now produce multiple unique anagrams while still using all the characters and keeping it sensible.

For example, the query `lewishamilton` would be split into two queries, `lewish` and  `amilton`, and each query would be processed through the API. If no valid result returns, we would scramble the letters and split the query once more, generating two more unique queries. This process would repeat until either an anagram is found or a set number of attempts is reached.

The implementation allows for more valid queries and produces an anagram at 50x the speed of the naive implementation.