The Problem
=========
1. There are two players.
2. Each player writes a number, hidden from the other player. It can be any integer 1 or greater.
3. The player reveal their numbers.
4. Whoever chose the lower number gets 1 point, unless the lower number is lower by only 1, then the player with higher number gets 2 points.
If they both chose the same number, neither player gets a point.
This repeats, and the game ends when one player has 5 points.

Is it possible to have a strong winning strategy?

My Intuitive Solution
=========
1. There is no such thing as random if there is a strategy.
2. If so, naturally each player is going to to pick 1. But if I know that you are going to pick 1, I will pick 2. Then you know I am going to pick 2, you will pick 3. And so forth.
3. Obviously I can't deduce that into a strategy based on I know that you know I know you know. So I can only take a guess as the best bet.
4. The way I guess your next number is taken the average of all the numbers you played in the same round. Plus 1. Unless your average number is greater than 2, then I will pick your mean mod by 5 (a simple way to counter skew the mean).

Which translates into:
```
def player_strategy(scores, player_answers, computer_answers, total_iterations, current_iteration):
    if not player_answers:
        return 1
    else:
        player_avg = int(sum(player_answers) / len(player_answers))
        if player_avg <= 2:
            return 2
        else:
            div = player_avg % 5
            if div == 0:
                return 1
            else:
                return div
                
```
Not Good Enough
=========
But this strategy didn't get me to the top 1 winning algorithm. So my naive answer is not optimal, which makes me put some mathematical rigor to it.

Which leads to this strategy:
```
def player_strategy(scores, player_answers, computer_answers, total_iterations, current_iteration):
    if scores['player'] == 4 and scores['computer'] == 4:
        return random.randint(1, 3)
    elif scores['player'] == 4 and scores['computer'] == 3:
        return random.randint(1, 5)
    elif scores['player'] == 3 and scores['computer'] == 3:
        return random.randint(2, 6)
    else:
        return random.randint(2, 4)
```
Winner Revealed His Algorithm
=========
Couple days ago, the winner revealed his algorithm. It's very hard to read and grasp what the code is doing without debugging.

```
def computer_strategy(scores, player_answers, computer_answers, total_iterations, current_iteration):
    v = [] # counter of values
    expected = [] # expected values of scores
    for i in range(10):
        v.append(1)
        expected.append(0)

    for i in player_answers:
        i = i - 1
        if i < 0:
            i = 0
        if i > 9:
            i = 9
        v[i] = v[i] + 1

    # compute expected value
    # computer chooses i
    # adversary chooses j
    for i in range(10):
        for j in range(10):
            score = 1
            if i + 1 == j:
                score = -2
            elif i - 1 == j:
                score = 2
            elif i == j:
                score = 0
            elif j < i:
                score = -1
            expected[i] = expected[i] + score * v[j]

    best = 1
    maxi = -10000000 # -INF
    for i in range(10):
        if expected[i] > maxi:
            maxi = expected[i]
            best = i + 1
    return best
```
Head to Head
=========
Naturally I want to know if my mathemaitcally formulated algorithm is optimal. Time to put it to the test!

Game Runner Code
---------
```
# https://realpython.com/blog/python/python-programming-contest-first-to-five/

import random


def player_strategy(scores, player_answers, computer_answers,
        total_iterations, current_iteration):
    if scores['player'] == 4 and scores['computer'] == 4:
        return random.randint(1, 3)
    elif scores['player'] == 4 and scores['computer'] == 3:
        return random.randint(1, 5)
    elif scores['player'] == 3 and scores['computer'] == 3:
        return random.randint(2, 6)
    else:
        return random.randint(2, 4)


def computer_strategy(scores, player_answers, computer_answers,
        total_iterations, current_iteration):
    v = [] # counter of values
    expected = [] # expected values of scores
    for i in range(10):
        v.append(1)
        expected.append(0)

    for i in player_answers:
        i = i - 1
        if i < 0:
            i = 0
        if i > 9:
            i = 9
        v[i] = v[i] + 1

    # compute expected value
    # computer chooses i
    # adversary chooses j
    for i in range(10):
        for j in range(10):
            score = 1
            if i + 1 == j:
                score = -2
            elif i - 1 == j:
                score = 2
            elif i == j:
                score = 0
            elif j < i:
                score = -1
            expected[i] = expected[i] + score * v[j]

    best = 1
    maxi = -10000000 # -INF
    for i in range(10):
        if expected[i] > maxi:
            maxi = expected[i]
            best = i + 1
    return best


def compute_score(player_answer, computer_answer, scores):
    if player_answer + 1 == computer_answer:
        scores['computer'] += 2
    elif computer_answer + 1 == player_answer:
        scores['player'] += 2
    elif player_answer < computer_answer:
        scores['player'] += 1
    elif player_answer > computer_answer:
        scores['computer'] += 1
    return scores


def calculate_winner(scores, outcomes):
    if scores['player'] > scores['computer']:
        outcomes['player'] += 1
    elif scores['computer'] > scores['player']:
        outcomes['computer'] += 1
    else:
        outcomes['ties'] += 1
    return outcomes


# run!

if __name__ == '__main__':
    iterations = 1000
    scores = {'player': 0, 'computer': 0}
    outcomes = {'player': 0, 'computer': 0, 'ties': 0}
    player_nums = []
    computer_nums = []
    rounds = []
    for x in range(0, iterations):
        # print x
        while scores['player'] < 5 and scores['computer'] < 5:
            player_answer = player_strategy(
                scores, player_nums, computer_nums, iterations, x)
            computer_answer = computer_strategy(
                scores, player_nums, computer_nums, iterations, x)
            player_nums.append(player_answer)
            computer_nums.append(computer_answer)
            compute_score(player_answer, computer_answer, scores)
        if len(computer_nums) == 3:
            print 'Computer: ', computer_nums
            print 'Player: ', player_nums
            print scores
        rounds.append(len(computer_nums))
        calculate_winner(scores, outcomes)
        scores = {'player': 0, 'computer': 0}
        player_nums = []
        computer_nums = []
    print('Results - Player {0}, Computer {1}, Tie {2}'.format(
        outcomes['player'], outcomes['computer'], outcomes['ties']))
    print 'average rounds', sum(rounds) * 1.0 / len(rounds)
```

Result
-------
**I win 56.4% (at minimum)**. He wins 43.6% (at maximum), which shows my algorithm (and easy to read too!) is **optimal**. 

Theoretical Stuff
=====
If you have a strategy that wins more than 50% of the time, your opponent can use the same strategy and also win more than 50% of the time, which is impossible. Therefore, the best possible strategy cannot guarantee winning more than 50% of the time.
