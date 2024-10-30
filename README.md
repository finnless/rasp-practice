# RASP Practice Problems

The following problems are taken from the [2024 CMC CS181 "Thinking Like a Transformer"](https://github.com/mikeizbicki/cmc-csci181-languages/blob/efd4e6754ac5e5cc1c409b3b8b57097ca1ce10cc/topic05_thinking_like_transformers/README.md#homework) section. They are intended to be solved in [RASP language](https://github.com/tech-srl/RASP).

You can install RASP by installing the submodule and running `./setup.sh`.
To install the RASP submodule, run `git submodule update --init`.

### Problem 1
Write an s-op that contains the indices in reverse order.
For example:
```
> reverse_indices("hello")
[4, 3, 2, 1, 0]
```

```
reverse_indices = length - indices - 1;
```


### Problem 2
Write a function that "rotates" the input text by the specified number of characters.
For example:
```
> rotate(tokens, 0)("hello")
"hello"
> rotate(tokens, 1)("hello")
"ohell"
> rotate(tokens, 2)("hello")
"lohel"
```

```
def rotate(s_op, k) {
    return aggregate(select(indices, (indices + k) % length, ==), s_op);
}
```


### Problem 3
Write a function that takes a sequence as input and "swaps every letter with its neighbor".
Specifically, for every even index $i$, positions $i$ and $i+1$ will be swapped;
if the length of the sequence is odd, then the last element should not move.
For example:
```
> swap(tokens)("hello")
"ehllo"
> swap(tokens)("ababab")
"bababa"
```

```
def swap(s_op) {
    adjustment_needed = (indices == length - 1) and (length % 2 == 1);
    target_index = indices if adjustment_needed else (indices + (1 - 2 * (indices % 2)));
    swap_selector = select(target_index, indices, ==);
    return aggregate(swap_selector, s_op);
}
```



### Problem 4
Write a function that returns the maximum value in the sequence repeated for every position.
For example:
```
> maxseq(tokens)("ababcabab")
"ccccccccc"
```

```
def maxseq(seq) {
    sorted_seq = sort(seq, seq);
    max_token = load_from_location(sorted_seq, length - 1);
    return max_token;
}
```


### Problem 5
Write a function that returns the maximum value in the sequnce only of the tokens seen so far.

For example:
```
> maxseq(tokens)("ababcabab")
"abbbccccc"
```

> *Hint:*
> The `mask_ag` selector is useful for any problem that is either "autogenerative" or using only tokens "seen so far".


SKIP. Is this possible?



### Problem 6
Write a function that performs sequence reversal "autogeneratively".
That is, it will take a sequence as input, and the sequence will contain a special token `$` that marks the "end of the prompt".
The text before the `$` should be unchanged, and the text after the `$` should be the text before the `$` reversed (this text represents the model's response to the prompt).
The code should be robuse to the case when the length of text after `$` is not the same as the length of text before `$`.
For example:
```
> reverse_ag(tokens)("hello$     ")
"hello$olleh"
> reverse_ag(tokens)("hello$ ")
"hello$o"
> reverse_ag(tokens)("hello$X")
"hello$o"
> reverse_ag(tokens)("hello$XXXXXXXXXX")
"hello$olleh     "
```

```
def reverse_ag(s_op) {
    dollar_index = aggregate(select(s_op, "$", ==), indices);
    before_selector = select(indices, dollar_index, <=) and select(indices, indices, ==);
    reverse_selector = select(indices, dollar_index, <=) and select(indices, 2 * dollar_index - indices, ==);
    extra_selector = select(indices, 2 * dollar_index, >) and select(indices, indices, ==);
    combined_selector = before_selector or reverse_selector or extra_selector;
    return aggregate(combined_selector, s_op);
}
```

### Problem 7
Write a function that counts the number of times a certain token appears in the input sequence.
For example:
```
> howmany(tokens, "a")("hello")
"00000"
> howmany(tokens, "h")("hello")
"11111"
> howmany(tokens, "l")("hello")
"22222"
```

```
def howmany(s_op, token) {
    return selector_width(select(s_op, token, ==));
}
```


### Problem 8
Write a function that counts the number of times a certain token has appeared in the input sequence so far.
For example:
```
> howmany(tokens, "a")("hello")
"00000"
> howmany(tokens, "h")("hello")
"11111"
> howmany(tokens, "e")("hello")
"01111"
> howmany(tokens, "l")("hello")
"00122"
```

```
def howmany(s_op, token) {
    return selector_width(select(s_op, token, ==) and select(indices, indices, <=));
}
```


### Problem 9
Create your own "interesting" problem statement.
Write a function/s-op that solves this problem.
Write a function that computes the parity of the number of times each character has occurred so far in the sequence.
For each position in the sequence, return '1' if the character's occurrence count up to that position is odd, and '0' if it is even.
For example:
```
> parity_of_occurrence(tokens)("ababab")
"1, 1, 0, 0, 1, 1"
> parity_of_occurrence(tokens)("hello")
"1, 1, 1, 0, 1"
```

```
def parity_of_occurrence(s_op) {
    count_so_far = selector_width(select(s_op, s_op, ==) and select(indices, indices, <=));
    parity = count_so_far % 2;
    return parity;
}
```



---


### Challenge
Write a "running average" function. That is, the function will return the average of every element in the sequence "seen so far". For example:
```
> running_average(indices)("hello")
[0, 0.5, 1.0, 1.5, 2.0]
> running_average(indices*2)("hello")
[0, 1.0, 2.0, 3.0, 4.0]
> running_average(indices^2)("hello")
[0, 0.5, 1.667, 3.5, 6.0]
> running_average(indices^2)("hello world")
[0, 0.5, 1.667, 3.5, 6.0, 9.167, 13.0, 17.5, 22.667, 28.5, 35.0]
```

```
def running_average(seq) {
    return aggregate(select(indices, indices, <=), seq);
}
```