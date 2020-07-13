# NES_Golf_Guide
Just a readme stored somewhere with my notes on editing NES GOLF.
Note I found this all out on the (e) release of Golf (so Golf(E).nes) this may not pertain to the other releases such as the versus.
My initial investigation to all of this can be found here:
https://www.romhacking.net/forum/index.php?topic=31006.0

## Table of Contents
1. Intro
2. Terms I'll be Using
3. Fairway Theroy and How it Works
4. Designing a new Fairway
5. (Putting) Green Theory and How it Works
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
- **slope** or **slope arrows** or **green arrows** | the arrows which indicate slope while putting

### Fairway Theroy and How it Works
The course is a big grid of tiles. the dimensions are **16 x 30**.
If you were to play the rom legally on FCEUX, you'd see under the Name Table Viewer, or if you look at the NES RAM (using the hex editor) at address 300 how our course is represented. It's just hex values, and each tile of our 16 x 30 grid has a defined number. Blank space is 9D, standard green is B8. Trees are A<something>. E-Z.
 Once you do the math however it becomes slightly but not too complicated. You have **212** (total hexes for a course description - 3 (Macro tile slots) - 3 (Seperator Tiles) - 1 (Hex to define Flag color) which gives us **~205** (I could be missing more) hexes to define **420** squares. Within this 205 we also have to define **sandtraps** and the placement of the **green** and the **initial tee**. Ack! How are we going to fit all of this??
 Compression. There's a simple compression scheme in place. It also doesn't hurt that the generic tiles are defined as part of the background, and we use the same data blob to add the sandtraps and green which are sprites it appears.Either or
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

### Designing a New Fairway
 <TODO -- don't have all the tile hex values yet>
 
### (Putting) Green Theory and How it Works
<TODO -- how do I move the xy of the hole!?>
Putting Greens are intereseting. Until I stepped through this game I thought it was complex and was expecting a huge data blob to define the arrows for slope and what not and to define these super cool putting greens  with interesting shapes and and and...there isn't that much to work with here.
There are 3 different "greens" which we can tweak slightly so it appears there are different putting greens.
 All of the arrows point the same direction for slope. All of the arrows are spaced evenly. It's an incredible let down once you see it; so lets get on with it.
 The general area where the code starts is **255f**. All greens **Share their macro tiles**. They are 255f,2560,2561.
 From there jump and skip and we have 4 hex ranges we care about.
 - **2562->2573** | Slope Arrow Spacing Hexes
 - **2574->2585** | Slope Arrow Direction (right digit only)
 - **2586->2597** | Which "green" shape to use (right digit only)
 - **259e->268f**| The whole range of the 3 pre-defined greens
 -- **259e->25e8** | Shape 1
 -- **25e9->2636** | Shape 2 (it's just mirroed shape 1)
 -- **2637->268f** | Shape 3

#### Defining the Shapes
They work exactly like the fairways except their macros come from **255f,2560,2561** and we obviously don't have that pesky sandtrap definition business to contend with. See how macro tiles work in the fairway section.

#### Defining the Arrows
- Starting ROM address **2562**, counting to the course number you wish to edit gets you to your spacing #. Give a a value (remember hex is 16 based) and that'll be the space set between arrows.
- Starting ROM address **2574**, counting to the course number you wish to edit gets you to your direction #
-- the values work clockwise starting from 12 o clock with 00, moving all 8 directions
					**00** = up,
					**01** = up-right,
					**02** = right,
					**03** = down-right,
					**04** = down,
					**05** = down-left,
					**06** = left,
					**07** = up-left,
- TODO - Idk how to define the location of the hole yet. oh well.

### Designing a new set of predefined Greens
 <TODO -- don't have all the tile hex values yet>




