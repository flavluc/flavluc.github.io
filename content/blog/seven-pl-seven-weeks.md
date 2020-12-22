+++
title = "Study notes: Seven Languages in Seven Weeks"
description = "These are my annotations and exercises when reading the book Seven Languages in Seven Weeks by Bruce Tate."
date = 2020-07-07
+++

# Introduction

### 1.1 Method to the madness

For each programming language you're, answers for the following questions:

  - What is the typing model?
  - What is the programming model?
  - How will you interact with it?
  - What are the decision constructs and core data structures?
  - What are the core features that make the language unique?

### 1.2 The languages

### 1.3 Buy this book

### 1.4 Don't buy this book

### 1.5 A Final Charge

# 2 Ruby

### 2.1 Quick History
  
  - Created by Yukihiro Matsumoto in 1993
  - Object oriented, duck (dynamically) typed, interpreted
  - Productivity over performance, simplicity over safety

### 2.2 Day 1: Finding a Nanny
  
#### Using Ruby with the Console

#### The Programming Model

  - Pure object oriented

#### Decisions

  - Simplicity for conditionals
    - block form: if _condition_, statements, _end_
    - one-line form: _statement_ if _condition_
  - The same goes for _while_ and _until_
  - Logical operators
    - Short circuit: &&, ||, and, or
    - Non-short circuit: &, |

#### Duck typing

  - Strongly typed: the type is always treated as expected
    - There are some exceptions
  - Duck typed

#### Day 1 Self-Study

  - Print the string “Hello, world.”
```
puts 'hello world'
```
  - For the string “Hello, Ruby,” find the index of the word “Ruby.”
```
/Ruby/ =~ 'Hello, Ruby'
```
  - Print your name ten times.
```
10.times {puts 'your name'}
```
  - Print the string “This is sentence number 1,” where the number 1 changes from 1 to 10.
```
(1..10).each {|x| puts "This is sentence number #{x}"}
```
  - Run a Ruby program from a file.
```
irb filepath.rb
```
  - Bonus problem: If you’re feeling the need for a little more, write a program that picks a random number. Let a player guess the number, telling the player whether the guess is too low or too high.
```
def main
  num = rand(1..10)
  loop do
    guess = gets.to_i
    case
    when guess > num
      puts "too high"
    when guess < num
      puts "too low"
    when guess == num
      puts "you got it ;)"
      break
    end
  end
end
```