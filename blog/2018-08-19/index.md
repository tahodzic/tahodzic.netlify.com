---
date: "2018-08-19"
title: "Conway's Game of Life"
category: "Software Development"
---

I recently stumbled upon Conway's Game of Life and was pretty amazed by how some simple rules can create something that really looks like "life". Cellular life that is. 
Academically speaking Game of Life is a [__cellular automation__](https://en.wikipedia.org/wiki/Cellular_automaton "Cellular Automation on Wiki"), simply speaking though it's just a zero-player game. That is, you set the initial state of the entire playing field, i.e. you spread out some cells on the field, and then you click play. The cells then die or revive each generation (each cycle) according to the rules. These are:
1. Any live cell with fewer than two live neighbors dies, as if by under population.
2. Any live cell with two or three live neighbors lives on to the next generation.
3. Any live cell with more than three live neighbors dies, as if by overpopulation.
4. Any dead cell with exactly three live neighbors becomes a live cell, as if by reproduction.

Programming wise it's not really that difficult to implement the rules, the difficult part is actually knowing how to structure your project. Which classes are needed? Which variables should be public? What methods do you need? etc.
As a novice I still have trouble with that, but nonetheless it was fun developing this "game".

I used C++ and SDL, since I already programmed a [__CHIP-8 emulator__](https://tahodzic.netlify.com/chip-8-エミュレータを開発してみました "Chip-8 emulator, Japanese") once and thought that reusing some of the code might be a good idea (Which it actually was).

Off to the [__code__](https://github.com/tahodzic/game-of-life "Game of life repo"). 