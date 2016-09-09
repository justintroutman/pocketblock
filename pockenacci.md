# Pockenacci

Pockenacci is a very simple block cipher, tested amongst 8-14 year-old students; it's built to emulate an SPN, or Substitution-Permutation Network, which is basically a series of substitutions and permutations that act on plaintext, controlled by an algorithm that an adversary can see, but configured with a key that they can't see. For example, the AES (Advanced Encryption Standard) is an SPN.

Pockenacci consists of the following components:

* Fibonacci-addition-style key schedule to expand our key,
* two P-boxes for permuting columns and rows, and
* two S-boxes for substituting individual characters

Pockenacci is an authenticated encryption scheme, which means it also includes a:

* a MAC, or Message Authentication Code, for preventing adversarial changes to ciphertext.

## Key Scheduling (We need keys!)

First, we need a key; let's start with a keyword: "SECRET"

Now, we need to convert this keyword into numbers; we do that by numbering each letter based on where it appears in the alphabet. If the keyword has two or more of the same letters, we number them from left to right.

````

S E C R E T
5 2 1 4 3 6

````

In alphabetical order, "C" appears in the alphabet before the other letters, so it gets the number "1". There are two instances of the letter "E", which is the next letter in alphabetical order; since we go from left to right, the first "E" gets the number "2", while the second "E" gets the number "3". When we finish this process, we'll get 5 2 1 4 3 6.

However, this cipher is a 6x6 grid, which means it can fit 36 alphanumeric characters; right now, our key is only six digits long. This means we need to expand it; this process of key expansion is done through a special algorithm known as "key scheduling".

We'll expand 5 2 1 4 3 6 through simple chain addition, which means we add, but do not carry; to do this, we take the first and second numbers in a row, add them together, and the result then becomes the first number of the second row. Let's begin:

````

5 2 1 4 3 6 (5+2=7; move 7 to next row)
7
````

Now we take the second and third numbers and do the same:

````

5 2 1 4 3 6 (2+1=3; move 3 to the next row)
7 3
````

And again:

````

5 2 1 4 3 6 (1+4=5; move 5 to the next row)
7 3 5
````

Keep this up for each row until you get six new rows, like this:

![image](https://www.justintroutman.com/images/keyschedule-1.png)


We'll discard the initial six digits we started with; we'll end up with our new expanded key, which is exactly 36 digits, to match the length of our plaintext. This is a basic example of *key expansion*, where a smaller key is expanded into multiple keys. In our case, we now have six keys of six digits, and we'll be using each one for a different operation.


## Loading Plaintext (A message to encrypt!)

Now that we have a key, we need something to encrypt. Let's say we want to encrypt the message, "This is a secret message that we need to hide."

![image](https://www.justintroutman.com/images/plaintext-1.png)

Now we're ready to set up our plaintext message; we're using a 6x6 grid, because it very nicely allows us to encrypt a-z and 0-9, which add up to 36 different characters. This means we can encrypt 36 characters at a time, in a 6x6 "block". This type of cipher is called a *block cipher*. We'll write our message from left to right, starting with the upper left most square. Let's say that our message is: "THIS IS A SECRET MESSAGE THAT WE NEED TO HIDE".

*This just happens to be 36 characters for our example; if your message is shorter, you can pad it with random characters; if it's longer, you'll need to split it into multiple blocks.*

![image](https://www.justintroutman.com/images/plaintext-1.png)

## Permutations (Let's move things around.)

The first operation we'll perform on our plaintext is a permutation; this doesn't change the value of any characters, but simply moves their position around in the grid. This is often called a *P-box* (P for permutation) and it helps diffuse the structure of the plaintext.

The first row of our key is `7 3 5 7 9 1`, so we'll use this to determine how and where the characters move. In this cipher, we'll choose to move the characters column by column. There are six columns and we have six key digits to work with. So, always starting with the top-most letter in the column, the first column will shift down by 7, the second column by 3, the third column by 5, and so on. When you've shifted all of the columns, you'll get this:

![image](https://www.justintroutman.com/images/pbox1-1.png)


You'll notice that the purple squares have moved; this shows the effect of the column-down permutation.

### Let's move them around again.

Now that we've moved our columns down, let's move our rows right. Here's the same block as above, but with the leftmost blocks highlighted in purple now that we're ready to show the effect of permuting the rows.

![image](https://www.justintroutman.com/images/pbox2-1.png)

### Okay, that's enough moving around.

Now, just like we moved the columns down, we will do the same for the rows, by moving them to the right with the key `0 8 2 6 0 8`, starting with the left-most letter in each row. Note here that our key includes "0" and "6"; the "0" means you don't do anything, and leave the row as-is; because our grid is 6x6, "6" has the same effect as "0": it means you do nothing, because moving six spaces up, down, left, or right will bring you right back where you started. After moving all of the rows, you'll get this:

![image](https://www.justintroutman.com/images/pbox3-1.png)

## Substitution (Let's change things completely.)

Now that we've shifted our plaintext message by column and by row, we want to actually change their value; this is done through what's called a substitution box, or *S-box*. This cipher uses two kinds -- one kind that simply changes the value of each letter, which is what we'll use now, and then what's called a "look-up table", where values are mapped out using coordinates. We'll talk about that later.

Our next key is `8 0 8 6 8 8`. Starting with the first letter in the 6x6 grid (upper left corner "T"), we want to shift its value forward in the alphabet by 8 spaces. The next letter, "E", will move forward 0 spaces, because the second number in our key is "0". We re-use the `8 0 8 6 8 8` key for each row. Also, keep in mind that our alphabet is alphanumeric, so it doesn't end with "Z". It's actually a-z, followed by 0-9. This makes up 36 characters, which is why we've chosen a 6x6 grid. When doing your value shifts, if your letter is Y, and you need to shift it three places, you would count like this: "Z, 0, 1", so the new value would be "1".

## Congrats! You've encrypted!

Once you've completed this, you'll get:

![image](https://www.justintroutman.com/images/ciphertext-1.png)

This is your final ciphertext. However, this only protects confidentiality, or keeps your secret safe; it doesn't protect your message from someone trying to change it when you send it to a friend, though. This is because there's no built-in mechanism to detect changes, which is why we need a MAC, or *message authentication code*.

## Message Authentication Codes (Check yourself!)

A MAC is a special code that you will check when you receive a message from a friend, because your friend attaches the MAC to the ciphertext. The MAC is based on the final ciphertext and their shared key. When you reconstruct it, you'll compare it to the MAC your friend attached. If they match, it means the ciphertext hasn't been altered by an adversary in any way. If the attacker attempts to change the ciphertext, the MAC won't be the same, because in order to create a new MAC for altered ciphertext, the attacker would have to know the key.

This is where we'll use the other type of S-box, which is a *look-up table*, and it does exactly what you think it might do. Using coordinates, you map their intersection to a particular value.

![image](https://www.justintroutman.com/images/abc-1.png)

You'll notice a new 6x6 grid, with a-z and 0-9, with the values `A E I O U Y` along the left side and top of the grid; there's nothing significant about `A E I O U Y`; they simply fit our need for a means to do look-ups, and do not affect the output in any way. Let's call this the "abc look-up table".

We'll use this table to get the coordinates that we'll use to look up that actual MAC values in another table; this other table, below, contains our full key, but also the `A E I O U Y` coordinates. Let's call this the "key look-up table".

![image](https://www.justintroutman.com/images/fullkey-1.png)


Basically, what we'll do is this. The first character of the ciphertext is the number "1". Let's go to the abc look-up table and find the number "1" in the grid, then determine its coordinates.

Once you locate the "1" in the abc look-up table, you'll notice that it's in the row labeled with a "U" on the left and the column labeled with an "O" up top. It's coordinates are "UO".

Now, look at the key look-up table. Use the coordinates "UO" to look up the first number that will become the first character in our MAC. Find the row labeled with "U" and the column labeled with "O", and you'll intersect at the number "0". If you repeat this look-up process for each character, you'll get the initial values of our MAC:

![image](https://www.justintroutman.com/images/mac_first_phase-1.png)

Okay, so now we have this first phase out of the way , but leaving it like this would allow an attacker to correlate which ciphertext letters correspond to which MAC numbers, making forgery easy. This is because our MAC value is pulling directly from our full key, so we need to mix things up a bit.

Remember that we've used the first three rows of our full key. The first to shift the columns down, the second to shift the rows right, and the third to shift the values of each letter forward. We still have three rows of key left.

Using this new grid of numbers above, and the same steps we used to encrypt our plaintext, use the fourth key (`8 8 4 4 6 6`) to shift the rows right, the fifth key (`6 2 8 0 2 4`) to shift the columns down, and the sixth and final key (`8 0 8 2 6 0`) to shift the values forward.

NOTE: The rule this time is that we'll only use numbers for shifting values; for example, if your number is 8, and you need to shift forward four spaces, you'll count like this: "9, 0, 1, 2", where your new value is "2". So that MACs look distinct from ciphertext, we won't wrap around and use a-z. This way, ciphertext can be alphanumeric, while the MAC is numeric; this is just an aesthetic choice.

When you shift the rows right by key `8 8 4 4 6 6`, you'll get this:
![image](https://www.justintroutman.com/images/mac_second_phase-1.png)'

When you shift the columns down by key `6 2 8 0 2 4`, you'll get this:
![image](https://www.justintroutman.com/images/mac_third_phase-1.png)

When you shift values forward by key `8 0 8 2 6 0`, you'll get this:
![image](https://www.justintroutman.com/images/mac_fourth_phase-1.png)

This last block represents your final MAC value.

## Finally, the end.

When you're ready to share your message, you'll append the MAC to the ciphertext:

(ciphertext on top / MAC below)

````

1 E M O I M   Ciphertext
M S 1 K M 0
L E I U 1 K
1 H V Y Q I
O S P N Z 1
0 D 4 S Q M
-----------
0 8 4 1 8 9   MAC
6 7 6 0 4 8
0 8 8 2 4 0
2 8 6 4 6 8
6 4 8 2 4 8
6 6 6 0 6 0

````
### Decryption is just a reverse process.

Decryption works in reverse, so if you shifted values forward you'll shift them backwards, if you shifted rows right, you'll shift them left, and if you shifted columns down, you'll shift them up.

### Check the integrity of your ciphertext before you decrypt!

However, before going through the trouble of encryption, the easiest way to verify that your ciphertext isn't corrupted or tainted (i.e., changed by an adversary), is to check the MAC. You'll use the steps above to rebuild the MAC and compare it to the one that's the attached to the ciphertext. If what you rebuild matches the one you received, the ciphertext is valid. Then you can decrypt. If it's not valid, you know it's corrupt, so you can discard the message.
