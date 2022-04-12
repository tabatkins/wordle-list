# wordle-list
A randomly-ordered list of all possible words that are potentially valid guesses in wordle, taken straight from the game's source code. Use it like:

```bash
curl -s https://raw.githubusercontent.com/tabatkins/wordle-list/main/words | grep ...
```

Filtering Guesses With `grep`
-----------------------------

If you're not familiar with using `grep`, it's very easy for use-cases like this; you only need the tiniest amount of regex knowledge.

Say you make the following two guesses:

```
 D R U N K
⬛🟩⬛🟨⬛

 F I G H T
⬛⬛🟨⬛⬛
```

Then you can whittle down the wordlist with the following chain of greps:

```bash
curl -s ... | grep -v [dukfiht] | grep .r... | grep n | grep -v ...n. | grep g | grep -v ..g..
```

which'll return the following list of potentially valid words:

```
groan
green
grown
```

The general structure of the chain of `grep`s is always the same:

1. Remove all the letters that returned a black tile with `grep -v [abcd]`, putting all the rejected letters between the square brackets. `grep -v` means "reject anything that matches this pattern", and the pattern will match any word containing one of those letters.
2. Force any letters that returned a green tile with `grep .a..b`, putting the letters in their appropriate spot and using `.` for anything you haven't gotten a green on yet. Make sure to pass all five characters, or else it'll match incorrectly.
3. For any tile with a yellow, first filter for words containing the letter with `grep a`, then filter out words with the letter in that position with `grep -v ..a..`; repeating more pairs if you have multiple yellow letters. Again, make sure to include `.`s to fill out a full five characters, or else it'll match incorrectly.

If you used any doubled letters in your guess, it's potentially more complicated:
* if both came up black, or both came up green, then it's fine. Just follow the above instructions like normal.
* if one came up yellow and other came up yellow or green, it means the word definitely has two of that letter. Just follow the instructions above, but manually ignore any word that *doesn't* have that letter twice.
* if one came up yellow/green and the other came up black, it means the word definitely has only one of that letter. Ignore the black tile but otherwise follow the instructions above, and manually ignore any words that *do* have that letter twice.

(These situations can be handled with regexes, but it's more complicated and not really worth explaining.)
