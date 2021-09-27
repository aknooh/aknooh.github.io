---
layout: single
classes: wide
title:  Rabin-Karp Algorithm - Pattern Matching Substring Search 
categories: posts
excerpt: "First Occurrence of Substring"
tags: [interview, strings, algorithm, hashing ]
comments: true
share: true
published: true
---

Recently, I was interviewing for a software engineering position, and during one of the interviews, one of the interviewers asked me the following question (paraphrasing):

> Given a substring "s" and a string "str", write an algorithm to find the first occurrence of "s" in "str".


At first glance, this looks like an easy question. In fact it's so easy that this can be done in [one line](#one-liner) using Python's [string find function](https://docs.python.org/3/library/stdtypes.html#str.find). However, once you clarify the implicit expectations with the interviewer, you find that, 1) You are not allowed to use any library functions such as Python's string find, 2) the algorithm must be efficient, and 3) the function returns the index of the first occurrence of `s`
in `str` or -1 otherwise. These are all simple and fair requirements.

##### What is a substring anyways?
A substring of a string is another string that occurs in that string. For example, the string "together", "to" is a substring of "together". "get" is another substring of "together". Another example is "foot" and "football". "foot" is a substring of "football".

Feeling pressured by the interviewer watching over me and the pressure of limited time, having not seen this problem before, I quickly devised and implemented a brute-force algorithm that uses two nested loops. The first loop would iterate through `str` and the second loop checks if `s` is in `str` starting from that position of `str`. I tested the function and it worked as expected. The time complexity for this algorithm is O(n^2). The
interviewer asked for a more efficient algorithm. I could not think of any other algorithms at the top of my head, and needless to say, I did not move to the next round. 

After the interview, Googling this question, I found that there a few algorithms that solve exact problem. I assume the interviewer was expecting an answer using one of those algorithms, and I did not provide one, so I did not move to the next round.
Anyways, one of those string searching algorithms is called the Rabin-Karp algorithm. This is an elegant algorithm that is very easy to understand.

#### What is Rabin-Karp Algorithm
Simply stated, the Rabin-Karp algorithm generates a hash of the substring(pattern) and checks the rolling hash of the string for a match.

- It is a string matching algorithm based on hashing
- It calculates **hash value** for the substring, and for each M-characters of the string.
- If the hash values are equal, then the algorithm compares the substring and M-characters of the string.
- If the hash values are not equal, then it calculates the hash value (i.e. the rolling hash) of the **next M-characters**.

At the end, there's only one comparison per text subsequence, and character matching is only needed when the hash values match.

{% highlight c++ %}
int RabinKarpSearch(const string &sub, const string &str)
{
    /*
     * Returns the index of the first occurrence of @sub in str
     * or -1 if @sub is not found in @str
     */

    // Error checking: size of substring should be <= size of str
    if (sub.size() > str.size())
        return -1;

    int str_hash = 0;
    int sub_hash = 0;
    int base = 26;
    int multiplyer = 1;

    // Find the hash for sub string and M characters of str
    // Where M is the length of the sub string
    for (unsigned int i = 0; i < sub.length(); ++i)
    {
        multiplyer = i ? multiplyer * base : 1;
        sub_hash = sub_hash * base + sub[i];
        str_hash = str_hash * base + str[i];
    }

    for (unsigned int i = sub.length(); i < str.length(); ++i) {
        // if there's a hash match, check if all characters match
        if (sub_hash == str_hash) {
            if (str.compare(i - sub.length(), sub.length(), sub) == 0) {
                // Found a match, return index of match
                return i - sub.length();
            }
        }

        // Compute the new hash by removing the current first letter and
        // adding the next letter from string
        str_hash -= str[i - sub.length()] * multiplyer;
        str_hash = str_hash * base + str[i];
    }

    // Edge case, check sub string at end of string
    if (sub_hash == str_hash) {
        if (str.compare(str.length() - sub.length(), sub.length(), sub) == 0)
            return str.length() - sub.length();
    }

    // no match found
    return -1;
}

{% endhighlight %}

### Hash Function
{% highlight c++ %}
for (unsigned int i = 0; i < sub.length(); ++i)
{
    multiplyer = i ? multiplyer * base : 1;
    sub_hash = sub_hash * base + sub[i];
    str_hash = str_hash * base + str[i];
}
{% endhighlight %}

This is a simple additive hash function that adds the positional values of each character. Additive hash function makes it easy to remove the hash of the first character.
The multiplyer keeps track of the size of the substring. It's equivalent to `pow(base, (sub.length() -1))`. 


`str_hash -= str[i - sub.length()] * multiplyer;` removes the first character from the string hash.


`str_hash = str_hash * base + str[i];` Adds the next character to the hash

### What's the Time Complexity?
With a good hash function, this algorithm runs in linear time. Since each character of the string is checked at most once, the "big-O" complexity is **O(m + n)** where m is the size of the substring and n is the size of the string.


<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

**Footnotes**

<a name="one-liner"></a> Python One-liner
{% highlight python %}
def find_substr(substr, string):
    '''
    Given a substring @substr and a string @string
    returns the index of the first occurrence of @substr
    in @string, otherwise returns -1
    '''
    return string.find(substr)

print(find_substr('hello', 'hello world'))  # 0
print(find_substr('quick', 'the quick brown the fox the'))  # 4
print(find_substr('yellow', 'hello world')) # -1

{% endhighlight %}
