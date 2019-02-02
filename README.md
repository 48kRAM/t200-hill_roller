# Hill Roller

Hill Roller - A game for the Tandy 200 in 10 lines of BASIC

Entry to the 2019 BASIC 10-line game competition at http://gkanold.wixsite.com/homeputerium/kopie-von-basic-10liners-2019

## Gameplay

You control a ball rolling down a path on a hill. Use the left and right cursor keys to move the ball left and right and avoid the edges of the path. But beware - the path gets narrower as you roll down the hill! Roll as far as you can to get the highest score. The game ends when you run into the side of the path or when then path width reduces completely. Your score (based on how long you rolled) is presented at the end of the game.

## Running

This game requires a Tandy model 200 portable computer with at least 8k of RAM (so, any T200). It will not work on any other M100/Kyocera-type system as it needs a 16-line display. Sorry.

### Real hardware

To load the game onto real hardware, use a DOS (teeny, ts-dos, floppy) and TPDD emulator (LaddieAlpha, mComm, desklink) to load the `HILL.DO` file into the T200. Open BASIC and run
```
LOAD "HILL.DO
RUN
```

### Emulation

Use the Virtual-T emulator:
 * Windows: https://sourceforge.net/projects/virtualt/files/Win32/v1.7/
 * Linux: https://sourceforge.net/projects/virtualt/files/Linux/v1.7/
 * Mac: https://sourceforge.net/projects/virtualt/files/OSX/v1.5%20Universal/10.4%20Tiger/
 
Under the "Emulation" menu, set "Model" to "T200". Select "File" then "Load file from HD". Select the `HILL.DO` file and click okay. Start BASIC and load the game:
```
LOAD "HILL.DO
RUN
```

## Code Explanation

As the 10-line version has no room for comments, the code is annotated below:

* Line 10
  * `POKE-2259,PEEK(-3553)` - Initialize the PRNG routine
  * `CLS` - Clear system screen
  * `DEFINTO-Z` - Define all required variables as integer (avoid slow floating-point math)
  * `DIMS(9):DIMO(9)` - Define a pair of circular buffers to store path position (S) and width (O)
  * `W=12:Z=9` - Initially set path position (Z) at 9 (chars from left of screen) and width at 12 chars
  * `R=Z+W/2` - Position player (R) in middle of the path to start game
  * `GOTO40` - Jump into the middle of the game loop to skip keyboard response (since there is none, yet)
* Line 20
  * `PRINT@639,""` - Draw to the bottom-left of the screen to trigger a scroll
  * `IF X=0 THEN V=-1 ELSE IF X=2 THEN V=1` - Adjust the path vector based on randomly-generated factor (X). Negative vectors move path to the left. Random factors of 1 or 3 are ignored, so the path vector remains unchanged (path continues in same direction as last time).
* Line 30
  * `Z=Z+V` - Adjust path position based on vector
  * `IF Z<2 THEN Z=2 ELSE IF Z>38-W THEN Z=38-W` - Constrain path position to what fits on the screen of the T200
* Line 40
  * `T=T+1:U=U+1` - Increment counters for path narrowing (T) and scorekeeping (U)
  * `A$=INKEY$` - Read a character scan from the keyboard
  * `IF P<6 THEN Q=P+4 ELSE Q=P-6` - Compute the point in the circular buffers that represents the vertical position of the player (i.e., look 7 lines back in the buffer)
* Line 50
  * `?@600+Z,CHR$(239);:?@600+Z+W,CHR$(239);` - Draw the left and right edges of the path, using semicolon to inhibit scrolling
  * `?@0,Y;` - Print the current score to the upper-left corner of the screen 
  * `IF U=10 THEN Y=Y+1:U=0` - Increment the player's score for every 10 lines of travel and reset the score timer
* Line 60
  * `IF A$=CHR$(29) THEN R=R-1 ELSE IF A$=CHR$(28) THEN R=R+1` - Move player left or right based on key input from line 40.
* Line 70
  * `?@R+320,"o";` - Draw the player's character (rolling ball) in its new position on the screen
  * `IF R<=S(Q) THEN 99 ELSE IF R>=S(Q)+O(Q) THEN 99` - Detect collisions with left or right sides of path. Left side is stored in buffer "S" directly. Right side is computed as sum of position and width values from buffers "S" and "O". These buffers allow collisions to be detected with previous values of the path's position and width rather than what was just drawn 7 lines lower. If a collision is detected, jump to the collision-handling routine at the end.
* Line 80
  * `X=RND(1)*4` - Generate a new random factor that will determine the next placement of the path edge
  * `P=P+1:IF P>9 THEN P=0` - Increment the pointer (P) into the circular buffers. Wrap around if needed.
* Line 90
  * `S(P)=Z: O(P)=W` - Store this line's position (Z) and width (W) into their respective buffers
  * `IF T>99 THEN W=W-1:T=0` - After the player traverses 100 lines of the path (T), decrement the path width to increase the difficulty of the game gradually and reset the line counter for the next 100 lines.
  * `GOTO 20 ELSE 20` - Regardless of the value of T, jump back to the start of the game loop. This `GOTO` statement must be added to both paths of the `IF` statement since BASIC doesn't allow you to terminate an `IF` statement except by ending the line.
* Line 99
  * `IF S(Q)=0 THEN 80` - The collision detection routine at line 70 doesn't account for the initial state of the game before the player's ball actually enters the path, so the first seven lines always jump to this "Game over" routine. This test jumps back out until the ball actually enters the path (when the circular buffer actually contains valid path position data).
  * `ELSE:?@R+320,CHR$(255)` - Draw a "crash" tile at the point of collision
  * `?@614,"Score: ";:?Y` - Print the player's score to the screen
  * `SOUND11000,40` - Play an appropriate "You screwed up!" tone on the speaker for dramatic effect. :)
  
