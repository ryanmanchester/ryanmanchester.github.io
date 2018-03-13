---
layout: post
title:      "Clean Conditionals : The Magic of the Ternary"
date:       2018-03-13 05:00:58 +0000
permalink:  clean_conditionals_the_magic_of_the_ternary
---


Conditional statements are a fantastic way to tell Ruby to perform a task based on a specific set of guidlines.  For example, here is some code from my (in progress) Tic Tac Tow with AI project:

```
def current_player
  if board.turn_count % 2 == 0
	  @player_1
	else
	  @player_2
	end
end
```

This code performs perfectly. It access an instance of the game board (from the Board class), runs #turn_count to see if the count is divisible by 2, or an even number. If it is even,then it is Player 1's turn, if odd, Player 2. 

Pretty straightforward, but while building a CLI program with three separate classes and two separate modules, wouldn't we want to take up as little space when possible? Wouldn't that make it easier to read? Even better, what if we could rewrite the same method so the body only takes up one line? Enter the ternary: 
```
def current_player
  board.turn_count.even? ? @player_1 : @player_2
	end
```

We went from four lines (plus and extra 'end') to one line! 

When I first discovered ternary statements, I wanted to use them everywhere instead of if/else. Well, you can't! Everything exists in Ruby for a reason. For example, ternary statements are useful for rewriting a basic if/else into one line. You NEED the else. You can't build a ternary on if alone. 


       
