Title: Euler Problem 92
Published: 28/9/2013
Tags:   
    - Code
    - Project Euler
---
Here's my solution to Euler problem 92. The code is pretty simple, it tries to shortcut the sequence by storing every number in a chain along with the number that recurs.

    class Euler92:
        def __init__(self):
            self.knownSeq = {}

        def ChainNumber(self,n):
            currentSeq = []
            while not n in [1,89]:
                currentSeq.append(n)
                if(n in self.knownSeq):
                    n = self.knownSeq[n]
                else:
                    n = SquareDigits(n)
            for x in currentSeq:
                self.knownSeq[x] = n
            return n

        def Solve(self, n):
            return len([x for x in range(2,n) if self.ChainNumber(x) == 89])
            
    def SquareDigits(n):
            return sum(map(lambda x: int(x)**2, str(n)))
