# tryhackmetyperacer
The project started after a wrote a hack for typing website http://play.typeracer.com.
While this is quite easy, I wondered what alternatives exists to block robots or scripts from automatic typing.

Generating a image from text is not hard, but does it solve the issue of real time feedback. You typically want to inform the typer is one way or the other, that his 
current typings is wrong and he should correct those before contining. 

To sum up, I want to achieve the following:
1. The text should be consealed and never be exposed to the client
2. The client should know if the input is right or wrong.

I believe I came up with a solution that fulfills both these requirements. It's a combination of hashing and visual feedback.

One way hashing is useful in a number of security related dillemas. For instance password stored in a database is hashed one way, to make the value almost completly useless if compromised. That is because you cannnot decrypt an one way hashed value.

Sampe funtion to validate a stordPassword
```js
function validate(input, storedPassword) {
    return hash(input) == storedPassword
}
```

So instead of validating passwords, I validate text. I could skip the hash of the whole text:

```js
var input = document.querySelector('input')
input.addEventListner('input', (e) => {
    
})
```

```js
// Exposed to client
var hashed = '2d0708903833afabdd232a20201176e8b58c5be8a6fe74265ac54db0';

var finished = function(value) {
    if (SHA3_224(value) == hashed) {
        console.log('Typing is 100%')
        return
    }
    console.log('There might be issue with your typing')
}

var input = document.querySelector('input')
input.addEventListner('input', (e) => {
    // Assume 10 is the length of current text
    if (input.value.length == 10) {
        finished(input.value)
    }    
})

```

But then client feedback doesn't exists. So instead of hashing the whole text I could hash words individually:
```js
// hashed words for 'The', 'quick', 'brown'
var hasedWords = [
    'f87c9fbeac7bed32e219a2874c3e34942c4d54fc5e37bf184b4793a3',
    'b8a5cf26796ac0402d8dd60e60b8f594d9547e8ab57311588fdcd166',
    'd553f2b663637b1806fdda7dfc944d19ea2a7a964ee65869963679f4'
]

function validateWord(word, index) {
    return hashedWords[index] == SHA3_224(word)
}

var input = document.querySelector('input')
input.addEventListner('input', (e) => {
    var words = input.value.split(' ')
    for (var i = 0; i < words.lenght; i++) {
        if (validateWord(words[i], i)) {
            console.log(word[i], ' is correct')
        } else {
            console.log(word[i], ' is not correct')
        }
    }
})
```

Bruteforcing seems difficult enough:
    -There are about: 72 different inputs. a-z + A-Z + 0+9 + !@&"'?.,-%
    -Available permutations given that the average english word is 5.1 characters:
    72^5.1 = 2 967 534 193.
    
But the number of english words is way less. There exists about 200 000 words in the enghlish dictionary. Add some puncutation and uppercase letters and you might get around 1-2 million permutation. And 1-2 million permutations is liable enough to bruteforcing.

So what about we have validation for every 5th or 6th character? Our permutation range will be much much larger as it may contain different parts of differnt words:

So for the sentence: "The quick brown fox jumps over the lazy dog." we need to validate:
-"The qu"
-"ick br"
-"own fo"
-"x jump"
-"s over"
-"the la"
-"zy dog"
-"."

Which wil be an improvement on difficulty for brute forcing. But still be need to have user feedback for each character. 

We don't need client script validation on each character. We only need visual feeback on wheter the character is right or wrong. And we simply draw the user's text above the image generated. We need to make sure the drawing of client side and of server side is perfeclty alligned. 

For each letter in layers:
1) Complete red block
2) Current input leter in green with transparent background.
3) Clipping of generated image. White with the correct letter transparent.

![layers](/docs/layers.png)
 
Left bottom is what a wrongly typed T will look like. There is enough red to show the user the current typing is wrong.




