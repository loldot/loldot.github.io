---
Title: Euler Problem 59
Published: 28/9/2013
Tags:   
    - Code
    - Project Euler
---
This is my solution to Euler problem 59 in python. It is pretty straight forward and relies on the heuristics of the problem. Initially it filters out every key that will only produce printable characters for all its corresponding positions in the encrypted text. It then proceeds to combine these keys and check for actual, common English words in the text.


    from math import ceil
    import string

    def test():
        asciis = open('cipher1.txt', 'r').read()
        encCodes = [int(s) for s in asciis.split(',')]

        asciiSum = 0
        
        pKeys = plausibleKeys(encCodes, 3)
        for k0 in pKeys[0]:
            for k1 in pKeys[1]:
                for k2 in pKeys[2]:
                    text = "".join(applyKey([k0,k1,k2], encCodes))
                    if(properStringProbability(text)):
                        print(text)
                        asciiSum = sum([ord(c) for c in text])
        return asciiSum

    def plausibleKeys(encCodes, keyLen):
        pKeys = {
            0: [x for x in range(255)],
            1: [y for y in range(255)],
            2: [z for z in range(255)]
            }

        for i, c in enumerate(encCodes):
            for k in pKeys[i % keyLen]:
                if chr(c ^ k) not in string.printable:
                    pKeys[i % keyLen].remove(k)
        return pKeys

    def properStringProbability(string):
        cnt = 0
        for word in ["the", "and", "have", "that", "you"]:
            cnt += string.count(word)        
        return cnt > 5


    def applyKey(key, asciiText):
        return [chr(x ^ int(y)) for (x,y) in zip(key * int(ceil(len(asciiText) / 3)),asciiText)]