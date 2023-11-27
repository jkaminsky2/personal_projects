# [Link to 4-Number Guessing Game](https://www.goobix.com/games/guess-the-number/)

---

## The Game

The game this code works for is a 4-digit number guessing game. Your objective is to guess a 4 digit number (where all the digits are unique) in as little guesses as possible. After every guess, the game will tell you how many digits are correct and in the right place and the number of digits that are correct but in the wrong place. While this game seems easy from the simple set of rules, it is quite difficult to combine information from different guesses, which, if done properly, can result in a lesser amount of guesses. If you do not believe me, try it out yourself (or use my code to help you beat your friends :))!

---

## Instructions
1. Go to the website above.
2. Copy the code below in a new Jupyter Notebook.
3. Run the cell block. It will prompt you to enter your first guess. Your guesses should be 4 unique digits with no spaces or other characters (e.x. 1234). Your first guess should ***always*** be **1234**. 
4. Enter the same guess into the website and hit 'Try it!'. It will then tell you the number of numbers in the correct place and the number of numbers but in the wrong place. Enter these values in that order one-by-one, as prompted by the code (e.x. enter 1, then 'return', then '2', and 'return' again). 
5. The code will then give you an optimal second guess. Enter this into the website. This time, all you will have to enter are the number of  numbers in the correct place and the number of numbers but in the wrong place (like Step 4).
6. Repeat Step 5 until you guessed the right number. Enter '4', then 'return', then '0', and 'return' again in order to finish the game from the code side.
7. Repeat Steps 1-6 for as many games as you want! If you want to see how effective the code is, attempt the game without the code and then with the code; see how many more steps it took you to complete the game (or less if you had a lucky guess :)).

---

**Always guess** ***1234*** **to start**

## Code
<pre>
    import itertools
import random

digits = "0123456789"
length = 4
combinations = list(itertools.permutations(digits, length))

def next_best_guess(combinations, last_try, correct, diff_position, total_correct):
    if correct == 4:
        return ('Correct Guess: ', last_try, correct)
    
    to_remove = correct + diff_position
    filtered = []
    for combination in combinations:
        first = last_try[0] in combination
        second = last_try[1] in combination
        third = last_try[2] in combination
        fourth = last_try[3] in combination
        lst = [first, second, third, fourth]
        
        first_corr = last_try[0] == combination[0]
        second_corr = last_try[1] == combination[1]
        third_corr = last_try[2] == combination[2]
        fourth_corr = last_try[3] == combination[3]
        lst_corr = [first_corr, second_corr, third_corr, fourth_corr]
        
        if sum(lst) == to_remove and sum(lst_corr) == correct:
            filtered.append(combination)
    total_correct += to_remove
    if last_try == ('1', '2', '3', '4'):
        next_guess = ('5', '6', '7', '8')  
    elif last_try == ('5', '6', '7', '8') and total_correct < 3:
        combos09 = []
        for combo in filtered:
            if '0' in combo and '9' in combo:
                combos09.append(combo)
        next_guess = random.choice(combos09)
    else:
        next_guess = random.choice(filtered)
    
    if len(filtered) == 1:
        print(F"This will be the correct guess: {next_guess}")
    else:
        print(F"Guess this: {next_guess}")
    
    return next_guess, filtered, total_correct

def get_valid_input(message, lower_limit, upper_limit):
    while True:
        try:
            value = int(input(message))
            if lower_limit <= value <= upper_limit:
                return value
            else:
                print(f"Invalid input. Please enter a value between {lower_limit} and {upper_limit}.")
        except ValueError:
            print("Invalid input. Please enter an integer.")
            
def get_valid_input2(message):
    while True:
        try:
            value = tuple(input(message))
            if int(value[0]) != int(value[1]) and int(value[0]) != int(value[2])\
            and int(value[0]) != int(value[3]) and int(value[1]) != int(value[2])\
            and int(value[1]) != int(value[3]) and int(value[2]) != int(value[3]) and len(value) == 4:
                return value
            else:
                print(f"Invalid input. Please enter 4 unique numbers between 0 and 9.")
        except ValueError:
            print("Invalid input. Please enter only integers with no spaces.")

input_tuple = get_valid_input2("Enter a guess of 4 different digits: ")

num_correct = get_valid_input("Enter the number of digits in correct position: ", 0, 4)
diff_position = get_valid_input("Enter the number of digits correct but not in the right position: ", 0, 4)

total_correct = 0

while True:
    next_guess, result, total_correct = next_best_guess(combinations, input_tuple, num_correct, diff_position, total_correct)
    
    if next_guess == 'Correct Guess: ':
        print(next_guess + "".join(result))
        break
    
    combinations = result
    input_tuple = next_guess
    
    num_correct = get_valid_input("Enter the number of digits in correct position: ", 0, 4)
    diff_position = get_valid_input("Enter the number of digits correct but not in the right position: ", 0, 4)

</pre>
   
