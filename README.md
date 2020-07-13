# NES_Golf_Guide
Just a readme stored somewhere with my notes on editing NES GOLF.
Note I found this all out on the (e) release of Golf (so Golf(E).nes) this may not pertain to the other releases such as the versus.
My initial investigation to all of this can be found here:
https://www.romhacking.net/forum/index.php?topic=31006.0

## Table of Contents
1. Intro
2. Terms I'll be Using
3. Fairway Theroy and how it works
4. Designing a new Fairway
5. (Putting) Green Theory and how it works
6. Designing a new set of predefined Greens
7. Cheat Sheets for Tiles (Fairway and Green)
8. Memory Addresses Table
9. TODO's

### Intro
NES Golf. Not a big game, not a complex game. I liked it as a kid; it wasn't a BIG part of my gaming life but when I did rent it I liked it. It had Mario (or so I thought) and was simple.
There are better golf games (looking at you NES Open) which I may want to cover in the future (same developer...can't imagine it's SUPER different in how it works) but for now starting with good ol' GOLF seemed like a nice idea. We can maybe get some new courses!

### Terms I'll be Using
- **Fairway** | The portion of a course when you are not putting
- **Green** | The portion of a course when you are putting
-- note that in context I may say green referring to the putting green visible on the fairway!
- **OB** | out of bounds
- **WH** | Water Hazard
- **ST** | Sand Trap
- **Macro Tiles** or **Macros** | these are special tiles which can be defined with 1 hex but cover a variable amount of tiles.
- **Tree Macros** or **Trees**| Trees are an always defined macro tile. A# = # of trees in a row throughout course development.
- **address** or **location** | forgive me I am inconsistent with my use of these two words and both to me generally mean memory location (context will be needed at all times for whether I mean RAM or ROM)

### Fairway Theroy and How it Works
The course is a big grid of tiles. the dimensions are **16 x 30**.
If you were to play the rom legally on FCEUX, you'd see under the Name Table Viewer, or if you look at the NES RAM (using the hex editor) at address 300 how our course is represented. It's just hex values, and each tile of our 16 x 30 grid has a defined number. Blank space is 9D, standard green is B8. Trees are A<something>. E-Z.
 Once you do the math however it becomes slightly but not too complicated. You have **212** (total hexes for a course description - 3 (Macro tile slots) - 3 (Seperator Tiles) - 1 (Hex to define Flag color) which gives us **~205** (I could be missing more) hexes to define **420** squares. Within this 205 we also have to define **sandtraps** and the placement of the **green** and the **initiall tee**. Ack! How are we going to fit all of this??
 Compression. There's a simple compression scheme in place.
 Each course follows this formula:
 
 **[Macro Tiles 0,1,2]**->**[Tile layout From top-left working right and down]**->**[Divider Hex]**->**[Pin Definition]**->**[Green Definition]**->**[Tee Definition]**->**[Sandtrap Definitions]**->**[FF to signal no more level data]**
 
The secret to success here is those early **macro tile** definitions AND **tree macros**

#### Macro Tiles
The first 3 hexes are reserved to be used for large swathes of the course. They can be anything. Usually you see 9D (blank) ,B8 (Green), or whatever water water is there.
after these are defined you will address these values by their index and how many you want to see (minus one) in a row.
ex: I have 9d,b8,b9
- we want 4 b8's in a row? **13**
- we want 16 9d's (super common) ? **0f**
- we want 1 b9 ? **20**
Note! 0 = 1, 1=2, etc when giving a quantity.

#### Tile layout and Divider Hex
Time to define the entire non-sandtrap/green of the course. We have our macros, we have a guide with all the possible tile values.
Tiles are defined **from TOP-LEFT** and **move RIGHT DOWN** wrapping every 16 tiles. That means if you had 03 followed by 0F in a row (for some odd reason) you'd add 4 tiles in row A, than add the next 12 tiles in row A and 4 tiles in row B.
After you have defined all 420 tiles, add FF to signify you are ready to start definining....

#### Pin Definition
3 hexes. All ya need.
- Hex 1 | Left digit defines the Y of the pin. right digit does nothing
- Hex 2 | Color of the pole and Flag
- Hex 3 | Left digit defines the X of the pin. right digit does nothing

#### Green Definition
3 Hexes
- Hex 1 | each digit controls a half of the green's shape
- Hex 2 | Left digit defines the Y of the pin. right digit does nothing
- Hex 3 | Left digit defines the X of the pin. right digit does nothing

#### Tee Definition
2 Hexes
- Hex 1 | Left digit defines the Y of the pin. right digit does nothing
- Hex 2 | Left digit defines the X of the pin. right digit does nothing

#### Sandtrap Definitions and Final End Mark
This is trickier than the last parts. As long as we still have hexes we have the ability to make sand traps it appears.
They follow the same pattern of definition until we reach **FF which is our Final end Mark for the course**
3 Hexes
- Hex 1 | pre-defined sandtrap shape
- Hex 2 | Left digit defines the Y of the pin. right digit does nothing
- Hex 3 | Left digit defines the X of the pin. right digit does nothing
 





