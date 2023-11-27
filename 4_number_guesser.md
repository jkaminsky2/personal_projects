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

    import requests
    from bs4 import BeautifulSoup
    import random
    all_word = []
    for i in range(1, 16):
        if i == 1:
            url = F'https://www.bestwordlist.com/5letterwords.htm'
        else:
            url = F'https://www.bestwordlist.com/5letterwordspage{i}.htm'
        response = requests.get(url)
        soup = BeautifulSoup(response.content, "html.parser")

        table = soup.find('table')
        words = []
        if table:
            rows = table.find_all('tr')
            for row in rows:
                columns = row.find_all('td')
                for column in columns:
                    word = column.text.strip()
                    words.append(word)
        if i == 1:
            a = words[1][509:-506]
            all_word += a.split()
        elif i < 10:
            a = words[1][517:-506]
            if i < 4:
                all_word += a.split()
            elif i < 9:
                a = a[:-5]
                all_word += a.split()
            else:
                a = a[:-6]
                all_word += a.split()
        else:
            a = words[1][518:-506]
            if i < 11:
                a = a[:-7]
                all_word += a.split()
            elif i < 13:
                a = a[:-8]
                all_word += a.split()
            else:
                a = a[:-3]
                all_word += a.split()
        print(a[:30], a[-30:])
       
    
    def get_unique_letters(string):
    unique_letters = []
    for char in string:
        if char not in unique_letters:
            unique_letters.append(char)
    return unique_letters


    def next_best_guess(combinations, last_try, correct, diff_position, count):
        #if word is correct
        if len(correct) == 5:
            print(F'Correct Guess in {count} tries:', last_try)
            return last_try, [], count
        #next guess number
        count += 1
        #eliminate words without correct letters in correct places
        filtered = []
        for combination in combinations:
            ct = 0
            needed = len(correct)
            for item in correct:
                if last_try[int(item)] == combination[int(item)]:
                    ct += 1
            if ct == needed:
                filtered.append(combination)
        #scenario where guessed letter twice but only one in word or guessed three times and in word twice
        filtered2 = []
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
            for word in filtered:
                curr = 1
                for letter in two_one:
                    if word.count(letter) == 1:
                        if word.index(letter) in need:
                            filtered2.append(word)
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
                            filtered2.append(word)      
        #eliminate words with incorrect letters that aren't dupes        
        filtered3 = []
        look_for = []
        for item in dups:
            if item not in three_two and item not in two_one:
                look_for.append(item)

        for word in filtered2:
            curr_ct = 0
            to_comp = len(look_for)
            for index in look_for:
                if last_try[index] not in word:
                    curr_ct += 1
            if curr_ct == to_comp:
                filtered3.append(word)
        #eliminate words with correct letters in wrong spot
        filtered4 = []
        for word in filtered3:
            ovr_count = 0
            for position in diff_position:
                if last_try[int(position)] != word[int(position)] and last_try[int(position)] in word:
                    ovr_count += 1
            if ovr_count == len(diff_position):
                filtered4.append(word)
        print(filtered4)
        #scenario where guessed letter three times but only one/two in word
        if last_try == 'serai' and len(correct) + len(diff_position) <= 3:
            next_guess = 'donut'
        else:
            next_guess = random.choice(filtered4)
            if len(get_unique_letters(next_guess)) <= 3 and len(correct) + len(diff_position) < 3:
                while len(get_unique_letters(next_guess)) <= 3:
                    next_guess = random.choice(filtered4)

        if len(filtered2) == 1 and count == 6:
            print(F"This will be the correct last guess: {next_guess}")
        elif len(filtered2) == 1:
            print(F"This will be the correct guess in {count} tries: {next_guess}")
        elif count == 6:
            print(F"This will be your last guess: {next_guess}")
        else:
            print(F"Guess this: {next_guess}")

        return next_guess, filtered4, count
