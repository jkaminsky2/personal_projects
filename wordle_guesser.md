# [Link to Wordle]([https://www.goobix.com/games/guess-the-number/)](https://www.nytimes.com/games/wordle/index.html)

---

## The Game

The game this code works for is a 5-letter guessing game. You have 6 guesses to guess a random 5-letter English words. On each guess, letters of the guess are marked in one of three colors: green, yellow, or gray. Green indicates the letter is correct, yellow says that the letter is in the word but not in that place, and gray means that the letter is not in the word. If a letter only appears in a word once and you guess that letter twice in a word in two incorrect places, then only the first will be yellow and the second will be gray (if you guess the same letter 3 times but all are in the wrong place and the letter occurs in the word twice, then the first two occurences will be yellow and the third will be gray). Similarly, if you guess the same letter 3 times all in the incorrect place when the letter only occurs once, only the first occurence of the ltter will be yellow and the last two will be gray. You are to use these clues to narrow down your guesses to guess the word in the least amount of guesses. Feel free to use the code below to assist you and beat your friends in the daily Wordle.

---

## Instructions
1. Go to the website above.
2. Copy the code below in a new Jupyter Notebook.
3. Run the cell block. It will give you a first guess.
4. Enter the same guess into the website and hit 'Enter'. The game will give you the hints mentioned above. 
5. The code will ask for this information. To properly input it, when asked for "letters in the correct places", indicate the index of letters in the correct place, separated by a comma. For example, if "i" and "e" are correct in "sizes", input 1,3. Do the same for the letters in the incorrect place (yellow).
6. The code will then give you an optimal second guess. Enter this into the website. Repeat steps 4 and 5 until you guess the right word.
7. Repeat Steps 1-6 for as many games as you want! If you want to see how effective the code is, attempt the game without the code and then with the code; see how many more steps it took you to complete the game (or less if you had a lucky guess :)).

---

## Code

<pre>
import requests
from bs4 import BeautifulSoup
import random
import pandas as pd

all_words = []
for i in range(1, 27):
    if i == 1:
        response = requests.get(F'https://www.bestwordlist.com/5letterwords.htm')
    else:
        response = requests.get(F'https://www.bestwordlist.com/5letterwordspage{i}.htm')
    soup = BeautifulSoup(response.content, "html.parser")

    table = soup.find('table')
    words = []
    if table:
        ct = 0
        rows = table.find_all('tr')
        for row in rows:
            columns = row.find_all('td')
            if ct == 0:
                ct += 1
                for column in columns:
                    word = column.text.strip()
                    words.append(word)
    if i == 1:
        all_words.extend(words[1][269:-862].split())
    elif i == 2 or i == 3:
        all_words.extend(words[1][277:-862].split())
    elif i >= 4 and i < 9:
        all_words.extend(words[1][277:-867].split())
    elif i == 9:
        all_words.extend(words[1][277:-868].split())
    elif i == 10:
        all_words.extend(words[1][278:-869].split())
    elif i >= 11 and i <= 23:
        all_words.extend(words[1][278:-870].split())
    else:
        all_words.extend(words[1][278:-865].split())


def get_unique_letters(string):
    unique_letters = []
    for char in string:
        if char not in unique_letters:
            unique_letters.append(char)
    return unique_letters

tf_idf_vals = pd.read_csv("/Users/joeykaminsky/Downloads/word_tfidf.csv")

not_in = []

for word in all_words:
    if word not in list(tf_idf_vals.columns):
        not_in.append(word)

        row_values = tf_idf_vals.iloc[0]
df_sorted = tf_idf_vals[row_values.sort_values(ascending = False).index]

all_words_updated = list(df_sorted.columns)

all_words_updated.extend(not_in)
all_words_updated = [i.upper() for i in all_words_updated]

all_words_grouped = {}
important_letters = {'S': 10, 'E': 9, 'A': 8, 'O': 7, 'R': 6, 'I': 5, 'L': 4, 'T': 3, 'N': 2, 'D': 1}

words_with_unique_letters_above = {}
words_with_3_4_letters_above = {}
words_with_dupe_3_4_letters_above = {}
words_with_1_2_letters_above = {}
words_with_dupe_1_2_letters_above = {}
words_with_0_letters_above = {}
words_with_dupe_0_letters_above = {}

for word in all_words_updated:
    curr_ct = 0
    has_dupe = False
    for letter in set(word):
        letter_ct = word.count(letter)
        if letter_ct > 1:
            has_dupe = True
        if letter in important_letters:
            curr_ct += 1
    if curr_ct == 5:
        words_with_unique_letters_above[word] = 1
    elif (curr_ct == 3 or curr_ct == 4) and has_dupe == False:
        words_with_3_4_letters_above[word] = 2
    elif (curr_ct == 3 or curr_ct == 4) and has_dupe == True:
        words_with_dupe_3_4_letters_above[word] = 3
    elif (curr_ct == 1 or curr_ct == 2) and has_dupe == False:
        words_with_1_2_letters_above[word] = 4
    elif (curr_ct == 1 or curr_ct == 2) and has_dupe == True:
        words_with_dupe_1_2_letters_above[word] = 5
    elif curr_ct == 0 and has_dupe == False:
        words_with_0_letters_above[word] = 6
    else:
        words_with_dupe_0_letters_above[word] = 7
        
all_words_grouped.update(words_with_unique_letters_above)
all_words_grouped.update(words_with_3_4_letters_above)
all_words_grouped.update(words_with_dupe_3_4_letters_above)
all_words_grouped.update(words_with_1_2_letters_above)
all_words_grouped.update(words_with_dupe_1_2_letters_above)
all_words_grouped.update(words_with_0_letters_above)
all_words_grouped.update(words_with_dupe_0_letters_above)


def next_best_guess(combinations, last_try, correct, diff_position, count):
    #if word is correct
    guessed = False
    if len(correct) == 5:
        print(F'Correct Guess in {count} tries:', last_try)
        guessed = True
        return guessed, last_try, [], count
    
    #next guess number
    count += 1
    #eliminate words without correct letters in correct places
    filtered_pre = {}
    
    for combination, rank in combinations.items():
        ct = 0
        needed = len(correct)
        for item in correct:
            if last_try[int(item)] == combination[int(item)]:
                ct += 1
        if ct == needed:
            filtered_pre[combination] = rank
    diff_position_letters = [last_try[int(i)] for i in diff_position]
    
    filtered = filtered_pre
    #num_wrong = 5 - len(correct) - len(diff_position)
    
    #for combination, rank in filtered_pre.items():
    ##    wrong_letter_not_in_word = 0
    #    for i in range(len(last_try)):
    #        if last_try[i] not in correct and last_try[i] not in diff_position_letters:
    #            if last_try[i] not in combination:
    #                wrong_letter_not_in_word += 1
    #    if wrong_letter_not_in_word == num_wrong:
    #        filtered[combination] = rank
    
    #scenario where guessed letter twice but only one in word or guessed three times and in word twice
    filtered2 = {}
    dups = []
    need = []
    two_one = []
    three_two = []
    for it in range(5):
        if str(it) not in correct and str(it) not in diff_position:
            dups.append(it)
        else:
            need.append(it)
            
    freq_dict = {}
    for item in need:
        if last_try[item] not in freq_dict:
            freq_dict[last_try[item]] = 1
        else:
            freq_dict[last_try[item]] += 1
    for item in dups:
        if last_try[item] in freq_dict.keys():
            if freq_dict[last_try[item]] > 1:
                three_two.append(item)
            else:
                two_one.append(item)
    
    
    if len(two_one) + len(three_two) == 0:
        filtered2 = filtered
    else:
        for word, rank in filtered.items():
            curr = 1
            for letter in two_one:
                if word.count(letter) == 1:
                    if word.index(letter) in need:
                        filtered2[word] = rank
            for letter in three_two:
                if word.count(letter) == 2:
                    start_index = 0
                    curr_ct = 0
                    for _ in range(2):
                        index = word.find(letter, start_index)
                        if index in need:
                            curr_ct += 1
                        start_index = index + 1
                    if curr_ct == 2:
                        filtered2[word] = rank 
                        
    #eliminate words with incorrect letters that aren't dupes        
    filtered3 = {}
    look_for = []
    for item in dups:
        if item not in three_two and item not in two_one:
            look_for.append(item)

    for word, rank in filtered2.items():
        curr_ct = 0
        to_comp = len(look_for)
        for index in look_for:
            if last_try[index] not in word:
                curr_ct += 1
        if curr_ct == to_comp:
            filtered3[word] = rank
    #eliminate words with correct letters in wrong spot
    filtered4 = {}
    for word, rank in filtered3.items():
        ovr_count = 0
        for position in diff_position:
            if last_try[int(position)] != word[int(position)] and last_try[int(position)] in word:
                ovr_count += 1
        if ovr_count == len(diff_position):
            filtered4[word] = rank
    print(filtered4)
    #scenario where guessed letter three times but only one/two in word
    if last_try == 'SERAI' and len(correct) + len(diff_position) <= 3:
        next_guess = 'DONUT'
    
    else:
        if count < 4:
            top_val = min(filtered4.values())
            if top_val < 5:
                to_look_at = [i for i, x in filtered4.items() if x <= top_val + 2]
            else:
                to_look_at = list(filtered4.values())
            next_guess = random.choice(to_look_at)
        else:
            next_guess = random.choice(list(filtered4.keys()))
            if len(get_unique_letters(next_guess)) <= 3 and len(correct) + len(diff_position) < 3:
                while len(get_unique_letters(next_guess)) <= 3:
                    next_guess = random.choice(list(filtered4.keys()))

    if len(list(filtered2.keys()))  == 1 and count == 6:
        print(F"This will be the correct last guess: {next_guess}")
    elif len(list(filtered2.keys())) == 1:
        print(F"This will be the correct guess in {count} tries: {next_guess}")
    elif count == 6:
        print(F"This will be your last guess: {next_guess}")
    else:
        print(F"Guess this: {next_guess}")

    return guessed, next_guess, filtered4, count

def get_valid_numbers():
    while True:
        try:
            # Ask the user for a list of numbers separated by commas
            user_input = input("Please enter a list of of the indexes where the character is in the correct place between 0 and 4 (separated by commas): ")
            
            if not user_input.strip():  # Check if the input is empty
                return []
            
            # Convert the input string to a list of integers
            numbers = [num.strip() for num in user_input.split(',')]

            # Check if all numbers are within the specified range
            if all((num == '0' or num == '1' or num == '2' or num == '3' or num == '4') for num in numbers):
                return numbers  # Return the valid list of numbers
            else:
                print("Invalid input. Please ensure all numbers are between 0 and 4.")
        except ValueError:
            print("Invalid input. Please enter valid integers separated by commas.")

def get_valid_numbers_2():
    while True:
        try:
            # Ask the user for a list of numbers separated by commas
            user_input = input("Please enter a list of the indexes where the character is in the wrong place between 0 and 4 (separated by commas): ")
            
            if not user_input.strip():  # Check if the input is empty
                return []
            
            # Convert the input string to a list of integers
            numbers = [num.strip() for num in user_input.split(',')]

            # Check if all numbers are within the specified range
            if all((num == '0' or num == '1' or num == '2' or num == '3' or num == '4') for num in numbers):
                return numbers  # Return the valid list of numbers
            else:
                print("Invalid input. Please ensure all numbers are between 0 and 4.")
        except ValueError:
            print("Invalid input. Please enter valid integers separated by commas.")

# Get a valid list of numbers from the user

def correct_guess_maybe():
    while True:
        try:
            # Ask the user for a list of numbers separated by commas
            user_input = input("Was that the correct guess? Enter 'Yes' or 'No'")
            if user_input == "Yes":
                return True
            elif user_input == "No":
                return False
            else:
                print("Invalid input. Please answer 'Yes' or 'No'")
        except ValueError:
            print("Invalid input")
guessed = False
guess_number = 1
new_combos = all_words_grouped
while guessed == False and guess_number < 6:
    if guess_number == 1:
        new_guess = "SERAI"
        print("Guess the following: SERAI")
    correctly_guessed = correct_guess_maybe()
    if correctly_guessed == True:
        if guess_number == 1:
            print(F"Congrats, you guessed the right word in 1 guess!")
        else:
            print(F"Congrats, you guessed the right word in {guess_number} guesses!")
        guessed = True
        continue
    correct = get_valid_numbers()
    wrong_spot = get_valid_numbers_2()
    
    guessed, new_guess, new_combos, guess_number = next_best_guess(new_combos, new_guess, correct, wrong_spot, guess_number)
</pre>
