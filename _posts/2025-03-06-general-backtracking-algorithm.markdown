---
layout: post
title: "General Backtracking Algorithm"
date: 2025-03-06 00:00:00 -0800
tags: python
---

I'll describe a general format for solving backtracking problems on Leetcode,
or in interviews. Not all of the parts are needed all of the time, and 
sometimes it's challenging to figure out where to put a piece of logic, but so
far, this has been working well for me.


## The General Format

This is the general format of the backtracking solution. The `state` variable
is a particular solution in the search space that is being built up one unit at
a time with each call to the `solve(state)` function. The
`get_candidates(state)` function generates these new additional units of a
particular solution with each call to the `solve(state)` function. The
`is_solution(state)` function is our escape hatch out of this recursive 
process. The `process_solution(state)` function does any transformations 
required to change the state object into the expected format of a solution.
This usually means collapsing a list of characters into a string or something.
The `is_valid(state)` function is an additional filter to skip any invalid 
solution candidates. (This could be wrapped into the `get_candidates(state)` 
function, but having it separate reminds you about this possibility.)

{% highlight python %}
from typing import List

class Solution:
    def problem(self, nums: List[int]) -> List[List[int]]:

        def is_solution(state) -> bool:
            pass

        def process_solution(state) -> None:
            pass
        
        def get_candidates(state) -> List:
            pass

        def is_valid(state) -> bool:
            pass

        def solve(state):
            if is_solution(state):
                process_solution(state)
                return
        
            candidates = get_candidates(state)
        
            for candidate in candidates:
                if is_valid(state + [candidate]):
                    solve(state + [candidate])

        solutions = []
        solve([])
        return solutions

nums = [1,2,3] # whatever input
Solution().problem(nums)
{% endhighlight %}


## Example: Permute

Problem statement: Given an array nums of distinct integers, return all the 
possible permutations. You can return the answer in any order.

In this case, the `is_valid(state)` function always returns `True`. The
`get_candidates(state)` function provides a list of integers not already
present in the current state, since repeats are not allowed in permutations.
The `is_solution(state)` function pops out of the recursion if we have a state
that is the same length of the original list; there's no need for any further
checks.

{% highlight python %}
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:

        def is_solution(state):
            if len(state) == len(nums):
                return True
            return False

        def process_solution(state):
            solutions.append(state)
        
        def get_candidates(state):
            candidates = [i for i in nums if i not in state]
            return candidates

        def is_valid(state):
            return True

        def solve(state):
            if is_solution(state):
                process_solution(state)
                return
        
            candidates = get_candidates(state)
        
            for candidate in candidates:
                if is_valid(state + [candidate]):
                    solve(state + [candidate])

        solutions = []
        solve([])
        return solutions

Solution().permute([1, 2, 3])
# >>> [[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]
{% endhighlight %}


## Example: Combination Sum

Problem statement: Combination Sum — Given an array of distinct integers 
candidates and a target integer target, return a list of all unique 
combinations of candidates where the chosen numbers sum to target.
You may return the combinations in any order. The same number may be chosen 
from candidates an unlimited number of times. Two combinations are unique if 
the frequency of at least one of the chosen numbers is different.

In this case, `is_valid(state)` is checking to make sure we're not "re-trying"
any solutions we've already attempted.

{% highlight python %}
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        def is_solution(state):
            if sum(state) == target:
                return True
            return False

        def process_solution(state):
            solutions.append(state)
        
        def get_candidates(state):
            items = [i for i in candidates if sum(state) + i <= target]
            return items

        def is_valid(state):
            state = sorted(state)
            if state in attempts:
                return False
            attempts.append(state)
            return True

        def solve(state):
            """
            General backtracking algorithm.
            """
            if is_solution(state):
                process_solution(state)
                return
        
            candidates = get_candidates(state)
        
            for candidate in candidates:
                if is_valid(state + [candidate]):
                    solve(state + [candidate])

        attempts = []
        solutions = []
        solve([])
        return solutions

Solution().combinationSum([2,3,6,7], 7)
# >>> [[2, 2, 3], [7]]
{% endhighlight %}


## Example: Letter Combinations of a Phone Number

Problem statement: Letter Combinations of a Phone Number — Given a string 
containing digits from 2-9 inclusive, return all possible letter combinations 
that the number could represent. Return the answer in any order.

{% highlight python %}
class Solution:
    def letterCombinations(self, digits: str):
        if not digits:
            return []
    
        def is_solution(state) -> bool:
            return len(state) == len(digits)
    
        def process_solution(state):
            solutions.append("".join(state))

        def get_candidates(state):
            if len(state) == len(digits):
                return []
            digit_to_letters = {
                '2': 'abc',
                '3': 'def',
                '4': 'ghi',
                '5': 'jkl',
                '6': 'mno',
                '7': 'pqrs',
                '8': 'tuv',
                '9': 'wxyz'
            }
            digit = digits[len(state)]
            return list(digit_to_letters[digit])
    
        def is_valid(state):
            return True
    
        def solve(state):
            if is_solution(state):
                process_solution(state)
                return
        
            candidates = get_candidates(state)
        
            for candidate in candidates:
                if is_valid(state + [candidate]):
                    solve(state + [candidate])

        solutions = []
        solve([])
        return solutions

Solution().letterCombinations("23")
# >>> ['ad', 'ae', 'af', 'bd', 'be', 'bf', 'cd', 'ce', 'cf']
{% endhighlight %}