# Regular Expressions

Adapted from course materials I created for BIOL 5005/5006: Research Methods at ULM from material written by [Jeremy Brown](https://github.com/jembrown?tab=repositories) at LSU.

## File setup for today

Download these files from the [data](https://github.com/cathynewman/SC_useful_code/tree/main/data) folder on GitHub. Make sure you save them with the exact filename and extension as shown here.
- words.txt
- names.txt
- LSUMZ18817.fasta
- swedish-lake-data.txt

## What are regular expressions?
Editing text files is one of the most common tasks in computational biology. Often, these files contain datasets in a format that needs to be changed, or from which some information needs to be extracted. Accomplishing these tasks by hand is tedious at best, and impossible in many cases.

**Regular expressions** (or **regex**) are an advanced form of "find" or "find-and-replace" that can be used to manipulate complex patterns of text.

Regular expression syntax can be used in a variety of different programs and contexts, including BBEdit (Mac), Notepad++ (Win), `grep`, and `sed`.

## grep

`grep` is a Unix program whose name stands for "Globally search a Regular Expression and Print".

`grep` searches input file(s) for a search string and prints the lines that match it. Beginning at the first line in the file, `grep` copies the line into a buffer, compares it against the search string, and if the comparison matches, prints the line to the screen. `grep` repeats the process until it runs out of lines.

Let's use the **words.txt** file to explore `grep` and some of its options (**flags**).

> [!WARNING]
> You can use double quotes (`" "`) around the search string instead of single quotes (`' '`) in most cases, but because double quotes won't work in some special cases, single quotes is easier.

```bash
grep 'boo' words.txt
```

It prints every line that includes 'boo':

```
## boot
## book
## booze
## boots
## boo
## boots?
## boot$trap
```

**Some useful flags:**
- `-n` = shows the line numbers of lines that match search string

    ```bash
    grep -n 'boo' words.txt
    ```

    ```
    #  1:boot
    #  2:book
    #  3:booze
    #  5:boots
    #  16:boo
    #  17:boots?
    #  18:boot$trap
    ```

- `-v` = prints the lines that _don't_ match search string

    ```bash
    grep -v 'boo' words.txt

    # What do you think this will do?
    grep -n -v 'boo' words.txt

    # Note that you can combine flags! Equivalent:
    grep -nv 'boo' words.txt
    ```

- `-c` = shows only the total number of lines that match search string

    ```bash
    grep -c 'boo' words.txt
    ```

    ```
    ## 7
    ```

- `-i` = ignores case

    ```bash
    grep -i 'boo' words.txt
    ```

    ```
    ## boot
    ## book
    ## booze
    ## boots
    ## BOOTS
    ## BOoks
    ## boo
    ## boots?
    ## boot$trap
    ```

- `-w` = matches entire word only

    ```bash
    grep -w 'boo' words.txt
    ```

    ```
    ## boo
    ```

## grep wildcards

Sometimes we need even more flexible searches. This is where wildcards come in.

**Wildcards** are symbols or strings with special meaning; they're used to replace or represent one or more characters.

- `-E` = allows use of regex wildcards in string search

    ```bash
    # What does grep find here?
    grep 'boots?' words.txt

    # What about if we use the -E flag:
    grep -E 'boots?' words.txt

    # What do you think the "?" wildcard means?
    ```

`?` indicates that the immediately preceding character might or might not be present.

### Some wildcards:
- `^` beginning of line (location indicator)
- `$` end of line (location indicator)
- `\n` end of line (character) - We'll come back to this later today.

    ```bash
    grep 'b' words.txt

    # Recall: flag -E allows wildcards
    # What does grep find here?
    grep -E '^b' words.txt

    # What about here?
    grep -E 'b$' words.txt
    ```

Now try searching these other special character combinations. What do they match?
- `\s`
- `\w`
- `\d`
- `.`

Did you guess above after trying them? Some of them are more obvious than others, with the particular word list we're using. Here are the answers:
- `\s` = white space (spacebar or tab)
- `\w` = word character (alphanumeric, not just letters!)
- `\d` = digit (0-9)
- `.` = any character

### Escaping special characters
What if you wanted to search for a literal `$`? How would you do it?

The backslash `\` is used to "escape" these characters that normally carry special meanings. So, if you search for `\$`, it will match a literal dollar sign instead of end of line.

Notice that this is essentially the opposite of the way `\` is used with regular letters (like `\d`). In that case, it _gives_ the letter meaning. But escaping _removes_ the special meaning.

> [!CAUTION]
> Here is where you'll have problems if you use double quotes.

```bash
# This finds every end-of-line
grep -E '$' words.txt

# Now try this. How is this different?
grep -E '\$' words.txt
```

### Repeating characters

Often, we want to search for more than one instance of a particular type of character in a row. To search for **one or more** instances of that character, add a `+`

```bash
grep -E 'bo+t' words.txt
```

Notice that it now finds *robots*:

```
## boot
## boots
## robots
## boots?
## boot$trap
```

Sometimes you're not sure if the character will be present at all but want to match it either way. To look for **zero or more** instances, use `*`

```bash
grep -E 'bo*t' words.txt

# How does that differ from this?
grep -E 'bo?t' words.txt
```

`*` looks for zero or more instances of a character, whereas `?` looks for either 0 or 1 instance exactly.

#### Try it!

Let's say you want to find any cases where a `.` is followed by a **word** or **number**, but there may or may not be **white space** (`\s`) in-between.

How would you write this search as a regular expression?

Remember to escape `.` since it's normally a symbol with special meaning.

### Ranges and "or"

`grep` searches for the exact string of characters you put inside `' '`. But what if, for example, you don't know if a text was written in American or British English, so you want grep to return matches for both _gray_ and _grey_?

This is when we use a **character class**. Character choices are put inside square brackets, and `grep` will match any one of the single characters in the brackets.

```bash
grep -E 'gr[ae]y' words.txt
```

```
## gray
## grey
```

You see that grep returned all lines that included matches for _gr_, followed by _a_ OR _e_, followed by _y_.

Another example: character classes are also useful if you want to search for only specific letters or digits and not all of them:

- `[ABCD]` matches only capital A, B, C, D, whereas `\w` would match all alphanumerical characters.
- `[12345]` matches only digits 1, 2, 3, 4, 5, whereas `\d` would match all digits.

> [!IMPORTANT]
> Regex in the command line is always case sensitive unless you use a flag to disable this behavior. So `[AB]` and `[ab]` will return different results unless you use `-i` to ignore case. Alternatively, search for `[ABab]`.

**Ranges** of letters or numbers can be included in custom sets to avoid having to type each one individually:
- `[0-9]` matches all individual digits
- `[A-Z]` matches all capital letters
- `[d-h]` matches lowercase letters between d and h, inclusive

> [!CAUTION]
> Be careful about specifying a range that includes both uppercase and lowercase letters! `[A-z]` will often either give you an error or include some non-letter characters, depending on your OS. To include both upper and lowercase, specify separate ranges: `[A-Za-z]`

You can also create wildcards that match anything **except** what you specify, by adding `^` to the beginning of your wildcard. Note that this is different from the use of `^` to specify beginning of line when wildcard is ***not*** inside square brackets. So, to find all characters that are _not_ lower-case letters, you could use `[^a-z]`. (Note that this finds all CHARACTERS that aren't lowercase letters; it doesn't only return uppercase letters!)

Special characters behave weirdly in character classes depending on the OS (sometimes they need to be escaped, other times not). So we won't mess with that right now.

## Useful Example
Let's practice what we just learned by exploring a large data file. **LSUMZ18817.fasta** contains thousands of short DNA sequences for a salamander tissue sample, catalog number LSUMZ 18817, a _Plethodon_ specimen in the LSU Museum of Natural Science Herpetology Collection. This is part of a real data file. The actual file is 3 times as large though, so I trimmed it down to have a more manageable file size.

This file is in FASTA format:

```
>Seq1_name description
ACATAAACATTGTTGCATGAATGTCTATATATTTTTTCCCTTTCTAATACAGCTTGTGGG
ATATGCAACAGAACGTGCGACACCTCACCTGGGTTCAAATCCAGAGCTAGCAGCAGAAAC
ATTGGATCCCTTG
>Seq2_name description
ACATAAACATTGTTGCATGAATGTCTATATATTTTTTCCCTTTCTAATACAGCTTGTGGG
ATATGCAACAGAACGTGCGACACCTCACCTGGGTTCAAATCCAGAGCTAGCAGC
```

The DNA sequences are in lines of up to 60 characters, so sequences >60 bases in length are printed in multiple lines, with a hard line break (enter key, not word wrapped) at the end of each line.

Because this file is large, we can't just open the file, skim through it, and count things by hand. We need to use some of the tools we've learned so far. Let's explore the file.

1. First, let's see **how many lines** there are in the file, using `wc` for _word count_.

    ```bash
    wc LSUMZ18817.fasta
    ```

    This prints the output as: `lines words characters filename`

    ```
    ## 101287  450895 8058257 LSUMZ18817.fasta
    ```

2. Now, let's count the **number of DNA sequences** in the file. This one is trickier. _Why can't we simply count the number of DNA sequence lines in the file?_ Take a look at the file. What do you notice about the first few sequences? Are they all the same length (same number of lines)? But in a FASTA file, each DNA sequence is preceded by ONE line containing the sequence name. That line with the name ALWAYS starts with the character `>`, and `>` never appears anywhere else in the file. So the number of DNA sequences will equal the number of lines containing `>` and the sequence name! ***This is a great simple example of how bioinformatics requires the development and use of problem solving skills!***

    ```bash
    # Use flag to only print the count, not each instance!!!
    grep -c '>' LSUMZ18817.fasta
    ```

    ```
    ## 16000
    ```

3. We want all of our sequences to be at least 100 bases in length. So let's check the file for any sequences **<100 bases long**. The sequence name format in this particular file gives us an easy way to do that that doesn't involve actually counting bases. The string **len=xxxx** in the seq names is the length of that sequence. So we just need to look for any seqs where the number following **len=** is exactly 1 or 2 digits. Recall the wildcard for matching 0 or 1 instance (`?`).

    ```bash
    # Since we're using regex wildcards, use -E flag
    grep -Ec 'len=\d\d?' LSUMZ18817.fasta

    # What happened?
    # Remember, grep finds all words _containing_ that string.
    # But we want grep to find that string as a whole word, i.e. followed by whitespace.
    grep -Ecw 'len=\d\d?' LSUMZ18817.fasta
    ```

4. Ok, so all of our sequences are at least 100 bases in length. Good. Now, here's the challenge. We need to know **how many** ***loci*** ("genes," sort of) we sequenced. If you scroll down through the file enough, you'll eventually see some sequences with repeated "TR" names - Some loci are so long that they were targeted with multiple overlapping sets of primers, so the full sequence for that locus is split in this file into multiple shorter sequences, each with the same "TR" identifier. So that means we can't just take our answer from step 2 above (the number of _sequences_) as the number of loci. Instead, we need to count the number of **UNIQUE "TR" identifiers**. Here's how we think through this problem:
- Each seq starts with **>TR** followed by 1 or more digits followed by a space, which means **>TRdigit+** is a "word" according to grep. So let's first use `grep` to find each instance of that word:

    ```bash
    # Don't run this yet, just type it in (don't hit enter)
    grep -Ew '>TR\d+' LSUMZ18817.fasta
    ```

- But wait. If we run the command like this, grep will return the _entire line_ of each line containing that word. We actually only want the word itself, not anything else on that line, so that each line with the same TR number will be counted together.

    ```bash
    # Add the -o flag to the command you typed above
    grep -Ewo '>TR\d+' LSUMZ18817.fasta
    ```

- Ok, now it gives us a list of each >TR#+ instance. Now, we need to find all the **unique lines**. To do that, we can use a new command: `uniq`. The output is a list of unique lines in the input, each only listed once. So we need to feed `uniq` the output from `grep` to show only the unique lines in the `grep` output list of TR identifiers:

    ```bash
    # Reenter the previous command, then pipe it to uniq
    grep -Ewo '>TR\d+' LSUMZ18817.fasta | uniq
    ```

- Now we have a list of all the unique TR identifiers (meaning, all the loci). The last step is to count up that list. Since each TR identifier is on a separate line, all we need is the total number of lines. We can use `wc` for that:

    ```bash
    # Reenter the previous commands, then pipe it to wc
    grep -Ewo '>TR\d+' LSUMZ18817.fasta | uniq | wc
    ```

- Now we have our answer!! Remember, the first number in the `wc` output is the number of lines: 9,598 loci sequenced.

### Answer to _Try It!_
You're searching for a **period** (escaped) (`\.`), followed by **0 or more spaces** (`\s*`), followed by an **alphanumeric character** (word or number) (`\w`).

```
grep -E '\.\s*\w' words.txt
```


## Find & replace with **sed**

`sed` = "Stream EDitor" - an extremely powerful Unix program. Here, we're only going to use its find & replace function.

To use `sed` for find & replace, give it instructions in the following format, replacing <TEXT> with your text and removing the brackets `< >`. Note that we use FORWARD slashes here.

```bash
's/<FIND_TEXT>/<REPLACE_TEXT>/g'
```

To start, try giving `sed` input using a pipe:

```bash
# Can you guess what will happen here?
echo "Here is some text" | sed 's/some/a little/g'
```

`sed` can also use the text of a file as input. Here's an example:

```bash
# Create new file
touch shells.txt

# Add text to file
echo "She sells seashells down by the seashore." >> shells.txt

# View file
cat shells.txt

# Run find-and-replace
sed 's/seashells/dinosaur bones/g' shells.txt

# View file
cat shells.txt

# What happened?
```

Perhaps most often you want to just do a find & replace on the file directly, and not have the output sent to the screen. To do that, add the flag `-i.bak`. This will run the replacement and create a backup file with the extension `.bak`. The backup file contains the original text. If you are satisfied with your changes, simply delete the backup file.

```bash
sed -i.bak 's/seashells/dinosaur bones/g' shells.txt

cat shells.txt
```

### Using regex with sed

One of the most powerful uses of regex is to reformat files. This requires finding text _patterns_ and replacing them (or adding to them).

Some regex characters aren't included in basic `sed`, so just like we used `grep -E` for grep regex, well use `sed -E` here.

**Recall:** `\n` is the wildcard for the end of line character (EOL). A "hard line break" is where you hit enter/return to go to the next line. If you look at a text file in a plain-text editor and show invisible characters (spaces, tabs, line breaks), you'll see that every hard line break has a special EOL character. That's what `\n` represents.

<img src="https://github.com/cathynewman/SC_useful_code/blob/main/images/EOL.jpg" width="70%">

To start, let's just have `sed` print the find & replace results to the terminal and not modify the file. Try to guess what the result will be for each case.

```bash
sed -E 's/^/NNN/g' shells.txt

sed -E 's/$/NNN/g' shells.txt

# Careful with the slashes! What is this command doing?
# Note that \s doesn't usually work anymore, as it is now a GNU-sed character.
sed -E 's/ /\n/g' shells.txt

# What is this command doing? What will be the result?
sed -E 's/.//g' shells.txt
```

You can also use **character classes** with `sed`. Let's replace all the uppercase letters with _N_:

```bash
echo "here is some text ABCDEFG" | sed -E 's/[A-Z]/N/g'
```

### Capturing text in search fields
It's very useful to capture some portion of the text matched by a search in order to use it as part of the replacement. To capture part of the search text, surround it with parentheses `( )`.

In the examples above, we added NNN to the beginning of each line and then, separately, to the end of each line. What if we wanted to add NNN to _both_ the beginning and end of each line with a single command, and leave the rest of the text unmodified?

We tell `sed` to find beginning of line, some text, end of line. Then tell `sed` to replace with NNN, some text, NNN. But if the text on each line isn't the same, we have to use regex to represent "some text" with wildcard(s), and then tell `sed` to use that same text in the replacement.
- **Current text:** She sells dinosaur bones by the seashore.
- **Goal text:** NNNShe sells dinosaur bones by the seashore.NNN
- First, put `( )` around the text to capture: (She sells dinosaur bones by the seashore.)

Now, think about what we want to capture. We don't want to change ANY existing text. So in essence, we want to capture ANY existing character in a line. So what wildcard do we use?

```bash
# Putting together the search string

# Find beginning of line
^

# Find ANY text and capture it to use in replacement
(.+)

# Find end of line
$

# Put it all together
^(.+)$
```

Now the **replacement string** - To use the captured text in the replacement, you'll need to indicate which captured field you'd like to use. Regex numbers them based on the **order of the ( ) sets in the search**. To indicate the first captured field, use `\1`, to indicate the second captured field, use `\2`, etc. Note these are _back slashes_. In our example, we only have one set of ( ) in the search anyway, so it's indicated with `\1`.

```bash
# Putting together the replacement string

# Replace at beginning of line
NNN

# Replace captured text
\1

# Replace at end of line
NNN

# Put it all together
NNN\1NNN
```

Now let's put the search and replace strings into the `sed` syntax and run it!

```bash
# sed syntax
sed -E 's/<SEARCH>/<REPLACE>/g' shells.txt

# Let's add our strings and run it!
sed -E 's/^(.+)$/NNN\1NNN/g' shells.txt
```

## Useful examples

Here are a couple of more examples of how `sed` can be useful when we need to efficiently reformat data files.

The file **names.txt** is a list of 50 randomly generated names in the format [LAST, FIRST] (with a space after the comma). All of the names contain only letters. Suppose we need to reformat this list to [FIRST LAST]. Let's run a search & replace.

> [!NOTE]
> **Update 2024:** It seems this won't work anymore with regular sed. To use `\w+` you'll have to install GNU-sed with homebrew (`brew install gnu-sed`) and call `gsed` instead of `sed`.

```bash
# Search string
# We want to keep first name and last name but not the comma or space
(\w+), (\w+)

# Replace string
# Remember, we want to flip the order of the names and put a space in-between
\2 \1

# Put it all together and run it (only screen output)
sed -E 's/(\w+), (\w+)/\2 \1/g' names.txt
```

Let's do another example. The file **swedish-lake-data.txt** contains temperature data from a Swedish sub-arctic lake ([data here](https://bolin.su.se/data/Crill-2015)). The file lists water temperatures in degrees Celsius taken at various times and at various depths. It's a **semicolon delimited file**, meaning the data cells are separated (delimited) by semicolons, and semicolons are not used for any other purpose in the file. To visualize what a semicolon delimited file looks like in table layout format, you can import the file into Excel and select "semicolon" as the delimiter. Make sure you understand that in this type of text file, two delimiters immediately together (no text or white space between them) indicates an empty cell, or _no data_. When you visualize the file in Excel and compare it to what it looks like in a plain-text editor, you can clearly see this.

It's often confusing at first when you are working with a delimited text file in a text editor because we are so accustomed to seeing the fields neatly lined up visually in Excel. The text editor doesn't work like that; it shows the text "as-is." So you need to know to count fields to determine (for example) which number on line 8 corresponds to which variable in the header row.

Many programs are very specific about the format of input files they accept. One common format requirement is specific notation for missing data. In our file currently, missing data is denoted by an empty cell (no text or white space at all). Let's say the program does not handle blank fields, so empty fields should be replaced with **NaN** ("Not a Number").

In a semicolon-delimited file, empty fields are denoted by `;;`

> [!IMPORTANT]
> WE MUST KEEP THE DELIMITERS. Deleting the delimiters is equivalent to deleting actual cells in Excel (not just the text inside the cell, but the cell itself). We are NOT replacing the `;;`. We're replacing what's inside `;;`. We are only replacing the "nothing" text _inside_ those fields with `NaN`. In other words, instead of `;<nothing>;`, we want `;NaN;`

This time we want to actually modify the file, so we'll use the `-i.bak` flag

```bash
sed -i.bak 's/;;/;NaN;/g' swedish-lake-data.txt
```

Another common format requirement is type of delimiter. Some programs will only accept comma-delimited files (CSV); others will only accept tab-delimited files. Our file is currently delimited by semicolons. Suppose we need to use a program that requires a tab-delimited file. Using `sed`, it's easy to find all semicolons and replace them with tabs.

**Recall:** The wildcard specific to a **tab** whitespace is `\t`.

```bash
sed -E -i.bak 's/;/\t/g' swedish-lake-data.txt
```
