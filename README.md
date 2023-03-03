![Banner](https://github.com/z-bj/Number-Guessing-Script/blob/master/NUMBER_GUESSING_GAME_BANNER.jpg)

![Shell Script](https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white)![postgreSQL](https://camo.githubusercontent.com/281c069a2703e948b536500b9fd808cb4fb2496b3b66741db4013a2c89e91986/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f506f737467726553514c2d3331363139323f7374796c653d666f722d7468652d6261646765266c6f676f3d706f737467726573716c266c6f676f436f6c6f723d7768697465)![vim](https://img.shields.io/badge/Vim-019733.svg?style=for-the-badge&logo=Vim&logoColor=white)


# Number — Guess inc. — Script

This is a simple bash script that allows a user to play a number guessing game. The script prompts the user to enter their username, checks if the user exists in the database, and adds them to the database if they are new. Then, it generates a random number between 1 and 1000 and prompts the user to guess the number. The user is given hints and asked to guess again until they guess correctly. Once the user guesses the number correctly, their guess count and the secret number are stored in the database.

### Usage

To run the game, execute the following command in your terminal:

bashCopy code

`./guessing_game.sh`

You must have PostgreSQL installed and running, and you need to set up a database with the name "number_guess". The script assumes that the database has a "players" table with "user_id" (serial), "username" (text), and "timestamp" (timestamp) columns, and a "games" table with "game_id" (serial), "user_id" (integer), "secret_number" (integer), "number_of_guesses" (integer), and "timestamp" (timestamp) columns.

### Script

``` bash

#!/bin/bash

# variable to query the database
PSQL="psql --username=freecodecamp --dbname=number_guess -t --no-align -c"


# promp player for username
echo -e "\nEnter your username:"
read USERNAME

# get username data
USERNAME_RESULT=$($PSQL "SELECT username FROM players WHERE username='$USERNAME'")
# get user id
USER_ID_RESULT=$($PSQL "SELECT user_id FROM players WHERE username='$USERNAME'")

# if player was not found
if [[ -z $USERNAME_RESULT ]]
  then
    # greet gamer 
    echo -e "\nWelcome, $USERNAME! It looks like this is your first time here.\n"
    # add player to database
    INSERT_USERNAME_RESULT=$($PSQL "INSERT INTO players(username) VALUES ('$USERNAME')")
    
  else
    
    GAMES_PLAYED=$($PSQL "SELECT COUNT(game_id) FROM games LEFT JOIN players USING(user_id) WHERE username='$USERNAME'")
    BEST_GAME=$($PSQL "SELECT MIN(number_of_guesses) FROM games LEFT JOIN players USING(user_id) WHERE username='$USERNAME'")

    echo Welcome back, $USERNAME\! You have played $GAMES_PLAYED games, and your best game took $BEST_GAME guesses.
fi

# generate random number between 1 and 1000
SECRET_NUMBER=$(( RANDOM % 1000 + 1 ))

# variable to store number of guesses/tries
GUESS_COUNT=0

# prompt first guess
echo "Guess the secret number between 1 and 1000:"
read USER_GUESS


# loop to prompt user to guess until correct
until [[ $USER_GUESS == $SECRET_NUMBER ]]
do
  
  # check guess is valid/an integer
  if [[ ! $USER_GUESS =~ ^[0-9]+$ ]]
    then
      # request valid guess
      echo -e "\nThat is not an integer, guess again:"
      read USER_GUESS
      # update guess count
      ((GUESS_COUNT++))
    
    # if its a valid guess
    else
      # check inequalities and give hint
      if [[ $USER_GUESS < $SECRET_NUMBER ]]
        then
          echo "It's higher than that, guess again:"
          read USER_GUESS
          # update guess count
          ((GUESS_COUNT++))
        else 
          echo "It's lower than that, guess again:"
          read USER_GUESS
          #update guess count
          ((GUESS_COUNT++))
      fi  
  fi

done

# loop ends when guess is correct so, update guess
((GUESS_COUNT++))

# get user id
USER_ID_RESULT=$($PSQL "SELECT user_id FROM players WHERE username='$USERNAME'")
# add result to game history/database
INSERT_GAME_RESULT=$($PSQL "INSERT INTO games(user_id, secret_number, number_of_guesses) VALUES ($USER_ID_RESULT, $SECRET_NUMBER, $GUESS_COUNT)")

# winning message
echo You guessed it in $GUESS_COUNT tries. The secret number was $SECRET_NUMBER. Nice job\!

# some comment

```



<img src="https://github.com/z-bj/Number-Guessing-Script/blob/master/deployparrot.gif" width="36">
