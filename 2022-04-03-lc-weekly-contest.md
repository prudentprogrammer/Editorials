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

Time Complexity: `O(1)` (some constant number of operations)
Space Complexity: `O(1)`

### Problem 2: [2225. Find Players With Zero or One Losses](https://leetcode.com/problems/find-players-with-zero-or-one-losses/)

Approach 1: Use Set Intersection and a Hashmap to record the counts for the loser.

```python
def findWinners(self, matches: List[List[int]]) -> List[List[int]]:
    winner_set = set()
    loser_set = set()
    loss_map = defaultdict(int)

    for winner, loser in matches:
        winner_set.add(winner)
        loser_set.add(loser)
        loss_map[loser] += 1

    return [sorted(list(winner_set - loser_set)), sorted([k for k,v in loss_map.items() if v == 1])]
```

Time Complexity: `O(nlogn)` (sorting)
Space Complexity: `O(3n) = O(n)`

**Approach 2:** Use one map and sort in the very end. First initialize the winner with count 0, if we encounter the winner again in the loser's position for another match, the count will be >= 1.

```python
def findWinners(self, matches: List[List[int]]) -> List[List[int]]:
    loss_map = defaultdict(int)

    for winner, loser in matches:
        if winner not in loss_map:
            loss_map[winner] = 0
        loss_map[loser] += 1

    res = [[], []]
    for k,v in loss_map.items():
        if v <= 1:
            res[v].append(k)
    return [sorted(x) for x in res]
```

***Time Complexity: `O(nlogn)` (sorting)
Space Complexity: `O(3n) = O(n)`***

Approach 3: Bucket Sort

```python
def findWinners(self, matches: List[List[int]]) -> List[List[int]]:
        max_number = float('-inf')
        for winner, loser in matches:
            max_number = max(max_number, winner, loser)

        buckets = [-1] * (max_number+1)
        for winner, loser in matches:
            if buckets[winner] == -1:
                buckets[winner] = 0
            if buckets[loser] == -1:
                buckets[loser] = 0
            buckets[loser] += 1

        res = [[], []]
        for i in range(len(buckets)):
            if buckets[i] == 0 or buckets[i] == 1:
                res[buckets[i]].append(i)
        return res
```

Time Complexity: `O(max(player_index))` (sorting)
Space Complexity: `O(max(player_index))`

### Problem 3: [2226. Maximum Candies Allocated to K Children](https://leetcode.com/problems/maximum-candies-allocated-to-k-children/)

```python
def maximumCandies(self, candies: List[int], k: int) -> int:
      def can_take(target):
          i = 0
          for candy in candies:
              if candy >= target:
                  i += (candy // target)
          return i >= k

      l, r = 1, max(candies)
      res = 0
      while l <= r:
          m = (l + r) // 2
          if can_take(m):
              res = m
              l = m + 1
          else:
              r = m - 1
      return res
```

Time Complexity: `O(nlog(max(candies)))` (sorting)
Space Complexity: `O(1)`


### Problem 4: [2227. Encrypt and Decrypt Strings](https://leetcode.com/problems/encrypt-and-decrypt-strings/)

* Eager Way
For encrypt, just map each letter to the corresponding encrypted letter.
For decryption, just generate all the possible permutations and store it in another map for quicker lookup.

```
abcd --> eizfeiam
acbd --> eieizfam
adbc --> eiamzfei
badc --> zfeiamei
dacb --> ameieizf
cadb --> eieiamzf
cbda --> eizfamei
abad --> eizfeiam
```


```python
def __init__(self, keys: List[str], values: List[str], dictionary: List[str]):
    self.encrypt_map = {k: v for k, v in zip(keys, values)}
    self.decrypt_map = Counter()

    for word in dictionary:
        res = self.encrypt(word)
        self.decrypt_map[res] += 1

def encrypt(self, word1: str) -> str:
    return ''.join([self.encrypt_map[letter] for letter in word1])

def decrypt(self, word2: str) -> int:
    return self.decrypt_map[word2]
```

* Lazy Way

Create a trie and traverse for decryption.

```python
class TrieNode:
    # Initialize your data structure here.
    def __init__(self):
        self.children = collections.defaultdict(TrieNode)
        self.is_word = False

class Trie:
    def __init__(self, node):
        self.root = node

    def insert(self, word):
        current = self.root
        for letter in word:
            current = current.children[letter]
        current.is_word = True


class Encrypter:

    def __init__(self, keys: List[str], values: List[str], dictionary: List[str]):
        self.encrypt_map = {}
        self.decrypt_map = defaultdict(list)
        self.root_node = TrieNode()
        self.trie = Trie(self.root_node)

        for k, v in zip(keys, values):
            self.encrypt_map[k] = v
            self.decrypt_map[v].append(k)

        for word in dictionary:
            self.trie.insert(word)

    def encrypt(self, word1: str) -> str:
        return ''.join([self.encrypt_map[letter] for letter in word1])

    def __dfs(self, node, word, i=0):
        if i == len(word):
            return node.is_word

        ans = 0
        curr_slice = word[i:i+2]
        for mp in self.decrypt_map[curr_slice]:
            current = node.children.get(mp)
            if not current is None:
                ans += self.__dfs(current, word, i + 2)
        return ans


    def decrypt(self, word2: str) -> int:
        return self.__dfs(self.root_node, word2)
```
