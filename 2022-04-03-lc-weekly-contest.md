## April 2nd, 2022 Weekly Leetcode Contest


### Problem 1: [2224. Minimum Number of Operations to Convert Time](https://leetcode.com/problems/minimum-number-of-operations-to-convert-time/)

The question is basically asking us to find the time difference between two strings. Compute the time difference in minutes and use a greedy approach to find how much `1, 5, 15, or 60` minutes go into that time difference.

```python
def convertTime(self, current: str, correct: str) -> int:
    def get_minutes(t):
        hours, mins = [int(x) for x in t.split(':')]
        return hours * 60 + mins

    cm, crm = get_minutes(current), get_minutes(correct)
    diff = crm - cm
    op = 0

    while diff != 0:
        for m in [60, 15, 5, 1]:
            op += (diff // m)
            diff %= m
    return op
```
