---
layout: post
title: Tic Tac Toe
subtitle: The game of kings and coding interviews
date: '2019-06-28 17:49:43'
tags:
- ruby
- misc-2
- misc
---

I'm back! And in Toronto now. There hasn't been a post in quite some time. It turns out that moving to a different country is more demanding of your time than I had once thought. So after spending the obligatory month-and-a-bit making sure I wasn't homeless, getting a credit score and going through the financial turmoil of furnishing a living space I'm back to doing what I love, coding.

I thought I'd start with something relevant to me at the moment, which is a coding test. This particular one was given to me in my last place of work for my coding interview. Tic Tac Toe.

side note: If you're a perspective employer reading this, hello, I'm great and a super great employee.

OK, lets get started. 

The game is Tic Tac Toe, everyone's played it and it makes for a good coding exercise. Its a 3 x 3 board where you each take a turn marking an X or an O with the hopes of getting three in a row. simple

lets begin with creating the Game class

~~~ ruby
class Game
  def initialize()

  end
end
~~~

Now we have a class with an initialize method in it, easy money. Now lets get the contestants.

~~~ ruby
class Game
  def initialize()
   setup_players
  end
  
  def setup_players
    puts("Hello and welcome to Tic Tac Toe")
    puts("Please enter the name of player 1:")
    @player1 = gets.chomp()

    puts("Please enter the name of player 2:")
    @player2 = gets.chomp()
    puts("welcome #{@player1} and #{@player2}. #{@player1} goes first.")
  end
end

~~~

So here we've added a simple method to setup the players, it prompts them to enter their names and grabs the text and stores it in a class variable. We also want this to run as soon as the game is run so we call the method in our initialize method.

Its all well and good having the players but they have nothing to play, lets create the board.

~~~ ruby
  def setup_board
    @board = {one: 1, two: 2, three: 3, four: 4, five: 5,
              six: 6, seven: 7, eight: 8, nine: 9}
  end

  def get_board
    puts("\n")
    puts("#{@board[:one]} | #{@board[:two]} | #{@board[:three]}")
    puts("- + - + -")
    puts("#{@board[:four]} | #{@board[:five]} | #{@board[:six]}")
    puts("- + - + -")
    puts("#{@board[:seven]} | #{@board[:eight]} | #{@board[:nine]}")
    puts("\n")
  end
~~~

Now we're cooking with gas, we have a method that creates a hash giving each tile on the board a number, the idea being that if player ones presses 1 it will be replaced with an x. The get board method will get the board in its current state and print it to the screen. so when we run the game we'll be asked for our names and presented with the board

~~~ shell
1 | 2 | 3
- + - + -
4 | 5 | 6
- + - + -
7 | 8 | 9
~~~

This game is shaping up nicely, but the players can't interact with the board in any way. This might stifle gameplay so lets address that next. We want to make it possible for players to take turns choosing tiles and having the board update to reflect those changes. So, 1.) we need to prompt the player to pick a tile. 2.) we need to start a counter to govern who's turn it is and 3.) we need to print the new updated board back so the players can see where they are in the game.

~~~ ruby
  # the updated initialize method with our new methods in it.
  def initialize()
    @turn_counter = 0
    setup_players
    setup_board
    get_board
    player_move
  end
 

  def player_move
    if @turn_counter < 9 && @turn_counter.even?
      print("#{@player1} please select a tile:")
      key = gets.chomp().to_i
        @board[@board.keys[key - 1]] = 'x'
        @turn_counter += 1
      end
    else
      print("#{@player2} please select a tile:")
      key = gets.chomp().to_i
      @board[@board.keys[key - 1]] = 'o'
      @turn_counter += 1
     end
     get_board
    end
~~~

Lets take a look at what we've done. The player_move method checks the counter and if its even (0 is even too) its player1's go, and if its odd its player2's go. Simple, now it prompts the player to pick a tile, stores the input as a int, checks the value against the hash we created (subtracting 1 from the number because this is programming and everything starts at 0) earlier and updates it. so when we print the board again at the end of this method we can see the updated board with the player's input on it.

~~~ shell
x | o | x
- + - + -
o | 5 | 6
- + - + -
7 | 8 | 9
~~~

So far so good, but there's no winning in this game, just a board you get to update. The game looses its charm after you notice it never ends and you can update tiles that have already been chosen. Its a modern take on the game thus far, but certainly the game-breaking "features" takes away from the experience somewhat. So next on the list are the win/draw conditions and making sure the players can only update a tile once.

First lets bang out the issue of multiple-use tiles:

~~~ ruby
  def tile_taken(tile)
    tile.kind_of?(String)
  end

  #updated player_move to check for used tiles
  def player_move
    if @turn_counter < 9 && @turn_counter.even?
      print("#{@player1} please select a tile:")
      key = gets.chomp().to_i
      if tile_taken(@board[@board.keys[key - 1]])
        puts("that tile is taken, please select another")
      else
        @board[@board.keys[key - 1]] = 'x'
        @turn_counter += 1
      end
    else
      print("#{@player2} please select a tile:")
      key = gets.chomp().to_i
      if tile_taken(@board[@board.keys[key - 1]])
        puts("that tile is taken, please select another")
      else
        @board[@board.keys[key - 1]] = 'o'
        @turn_counter += 1
      end
    end
    get_board
  end
~~~

Now that that's been ironed out, onto the win/draw conditions. Tic Tac Toe as a game only has 8 possible combinations that result in a player winning, with a number that low we can just hard-code them in and do a quick check to see if a player has won after each turn. And because there's only nine tiles, after 9 moves the game will end anyway because the board is full and the game is no longer playable.

~~~ ruby
  def win_condition_met
    if @board.values_at(:one,:two,:three).uniq.length == 1 ||
      @board.values_at(:four,:five,:six).uniq.length == 1 ||
      @board.values_at(:seven,:eight,:nine).uniq.length == 1 ||
      @board.values_at(:one,:four,:seven).uniq.length == 1 ||
      @board.values_at(:two,:five,:eight).uniq.length == 1 ||
      @board.values_at(:three,:six,:nine).uniq.length == 1 ||
      @board.values_at(:one,:five,:nine).uniq.length == 1 ||
      @board.values_at(:three,:five,:seven).uniq.length == 1
      @turn_counter.even? ? puts("Congratulations #{@player2} you are the winner!") : puts("Congratulations #{@player1} you are the winner!")
      exit()
    end
  end

  def move_limit_reached
    @turn_counter >= 9 ? puts("its a draw!") && exit() : player_move
  end

  #added the win/draw methods to check after each turn. 
  def player_move
    if @turn_counter < 9 && @turn_counter.even?
      print("#{@player1} please select a tile:")
      key = gets.chomp().to_i
      if tile_taken(@board[@board.keys[key - 1]])
        puts("that tile is taken, please select another")
      else
        @board[@board.keys[key - 1]] = 'x'
        @turn_counter += 1
      end
    else
      print("#{@player2} please select a tile:")
      key = gets.chomp().to_i
      if tile_taken(@board[@board.keys[key - 1]])
        puts("that tile is taken, please select another")
      else
        @board[@board.keys[key - 1]] = 'o'
        @turn_counter += 1
      end
    end
    get_board
    win_condition_met
    move_limit_reached
  end
end
~~~

So, rather than checking twice, once for player1 and once for player2 to see who's won we can shorten it down a bit and use the [.uniq](https://apidock.com/ruby/Array/uniq) method to grab an array of the values and see if there's only one value in it. So if it's either all x or all o it will return true. Then do a quick check with the turn counter to see who's won. Almost every problem can be solved by putting stuff into arrays, remember: Work smart, not hard.

the next thing we did was to check to see if the game ended in a draw, and that's easily done by checking the counter. So not much to explain there. We also put in an exit() at the end of each method because the game is over and you don't want the game to continue after it's over.

And that's about it, this took just over an hour to so a perfect interview-amount-of-time for a coding challenge. Here's the end result:

~~~ shell
peter please select a tile:7

x | o | x
- + - + -
o | x | o
- + - + -
x | 8 | 9

Congratulations peter you are the winner!
~~~

If you were following along great, you've definitely run it by now and have had fun playing Tic Tac Toe. If not, Heres the TLDR, just copy and paste this code into a file and run it.

~~~ ruby
class Game
  def initialize()
    @turn_counter = 0
    setup_players
    setup_board
    get_board
    player_move
  end

  def setup_board
    @board = {one: 1, two: 2, three: 3, four: 4, five: 5, six: 6, seven: 7, eight: 8, nine: 9}
  end

  def setup_players
    puts("Hello and welcome to Tic Tac Toe")
    puts("Please enter the name of player 1:")
    @player1 = gets.chomp()

    puts("Please enter the name of player 2:")
    @player2 = gets.chomp()

    puts("welcome #{@player1} and #{@player2}. #{@player1} goes first, please select a tile")
  end

  def get_board
    puts("\n")
    puts("#{@board[:one]} | #{@board[:two]} | #{@board[:three]}")
    puts("- + - + -")
    puts("#{@board[:four]} | #{@board[:five]} | #{@board[:six]}")
    puts("- + - + -")
    puts("#{@board[:seven]} | #{@board[:eight]} | #{@board[:nine]}")
    puts("\n")
  end

  def win_condition_met
    if @board.values_at(:one,:two,:three).uniq.length == 1 ||
      @board.values_at(:four,:five,:six).uniq.length == 1 ||
      @board.values_at(:seven,:eight,:nine).uniq.length == 1 ||
      @board.values_at(:one,:four,:seven).uniq.length == 1 ||
      @board.values_at(:two,:five,:eight).uniq.length == 1 ||
      @board.values_at(:three,:six,:nine).uniq.length == 1 ||
      @board.values_at(:one,:five,:nine).uniq.length == 1 ||
      @board.values_at(:three,:five,:seven).uniq.length == 1
      @turn_counter.even? ? puts("Congratulations #{@player2} you are the winner!") : puts("Congratulations #{@player1} you are the winner!")
      exit()
    end
  end

  def move_limit_reached
    @turn_counter >= 9 ? puts("Its a draw!") && exit() : player_move
  end

  def tile_taken(tile)
    tile.kind_of?(String)
  end

  def player_move
    if @turn_counter < 9 && @turn_counter.even?
      print("#{@player1} please select a tile:")
      key = gets.chomp().to_i
      if tile_taken(@board[@board.keys[key - 1]])
        puts("that tile is taken, please select another")
      else
        @board[@board.keys[key - 1]] = 'x'
        @turn_counter += 1
      end
    else
      print("#{@player2} please select a tile:")
      key = gets.chomp().to_i
      if tile_taken(@board[@board.keys[key - 1]])
        puts("that tile is taken, please select another")
      else
        @board[@board.keys[key - 1]] = 'o'
        @turn_counter += 1
      end
    end
    get_board
    win_condition_met
    move_limit_reached
  end
end

Game.new()

~~~
