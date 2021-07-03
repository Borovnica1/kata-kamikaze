Kata:

This kata is inspired by Plants vs. Zombies, a tower defense video game developed and originally published by PopCap Games.

The battlefield is the front lawn and the zombies are coming. Our defenses (consisting of pea-shooters) are in place and we've got the stats of each attacking zombie. Your job is to figure out how long it takes for them to penetrate our defenses.

Mechanics
The two images below represent the lawn (for the test example below) at separate stages in the game.
Left: state at move 3, before shooters fire. Right:state at move 5, after shooters fire. (Moves are 0-based)

![plants vs zombies demo picture](plantsvszombies.png)
Moves: During a move, new zombies appear and/or existing ones move forward one space to the left. Then the shooters fire. This two-step process repeats.
If a zombie reaches a shooter's position, it destroys that shooter. In the example image above, the zombie at [4,4] on the left reaches the shooter at [4,2] and destroys it. The zombie has 1 health point remaining and is eliminated in the same move by the shooter at [4,0].
Numbered shooters shoot straight (to the right) a given number of times per move. In the example image, the green numbered shooter at [0,0] fires 2 times per move.
S-shooters shoot straight, and diagonally upward and downward (ie. three directions simultaneously) once per move. In the example image, the blue and orange S-shooters can attack zombies in any of the blue and orange squares, respectively (if not blocked by other zombies).
At move 3 the blue shooter can only hit the zombie at [1,5] while the orange shooter hits each of the zombies at [1,5], [2,7], and [4,6] once for that move.
Shooting Priority: The numbered shooters fire all their shots in a cluster, then the S-shooters fire their shots in order from right to left, then top to bottom. Note that once a zombie's health reaches 0 it drops immediately and does not absorb any additional shooter pellets.
In the example image, the orange S-shooter fires before the blue one.
Input
Your function will receive two arguments:

Lawn Map: An array/list consisting of strings, where each string represents a row of the map. Each string will consist of either " " (space character) which represents empty space, a numeric digit (0-9) representing a numbered shooter, or the letter S representing an S-shooter.
Zombie Stats: An array of subarrays representing each zombie, in the following format:
[i,row,hp] - where i is the move number (0-based) when it appears, row is the row the zombie walks down, and hp is the initial health point value of the zombie.
When new zombies appear, they start at the farthest right column of their row.
Input will always be valid.

Output
Return the number of moves before the first zombie penetrates our defenses (by getting past column 0), or null/None if all zombies are eliminated.

Test Example
let lawn = [
    '2       ',
    '  S     ',
    '21  S   ',
    '13      ',
    '2 3     '
];
let zombies = [[0,4,28],[1,1,6],[2,0,10],[2,4,15],[3,2,16],[3,3,13]];
plantsAndZombies(lawn,zombies); //10
For another Tower Defense-style challenge, check out Tower Defense: Risk Analysis

If you enjoyed this kata, be sure to check out my other katas.

My SOl:

    // something
    function plantsAndZombies(lawn,zombies){
        const zombiesRows = [];
        for (let i = 0; i < lawn.length; i++) {
            zombiesRows.push([]);
        }
        for (let i = 0; i < lawn.length; i++) {
            zombiesRows[i] = zombiesRows[i].sort((a, b) => b[0] - a[0]);
        }
        lawn = lawn.map(row => row.split(''));
        let moves = 0;
        let noZombies;
        while (true) {
  
    // add zombies
     zombies.forEach((zombie, k) => {
      if (zombie[0] === 0) {
        zombiesRows[zombie[1]].push([zombie[0],zombie[2]])
      }
    })
    zombies = zombies.filter(zombie => zombie[0] > 0);
    
    // first only numbered plants
    lawn.forEach((row, i) => {
      row.forEach((spot, j) => {
      
        // switch fire numbered,, later make for S from right and top
        if (spot !== ' ' && spot !== 'S') {
          let damage = Number(spot);
          zombiesRows[i].some((zombie, k) => {
          
            // first zombie we can hit
            const plantSpot = row.length - j-1;;
            if (zombie[0] < plantSpot) {
              zombie[1] -= damage;
              damage = 0
              
              // check if this killed him and remove him if so
              if (zombie[1] <= 0) {
                // save damage for next one
                damage += zombie[1] * -1;
              }
              if (damage == 0) return true
            }
            return false
          })
          zombiesRows[i] = zombiesRows[i].filter(zombie => zombie[1] > 0);
        }
      })
    })
    let i = 0;
    let j = lawn[0].length - 1;
    while (j !== -1) {
      const spot = lawn[i][j];
      if (spot === 'S') {
      
        // get zombies from all 3 lanes
        // horizontal lane
          zombiesRows[i].some((zombie, k) => {
            // first zombie we can hit
            const plantSpot = lawn[0].length-1-j;
            const damage = 1;
            if (zombie[0] < plantSpot) {
              zombie[1] -= damage;
              if (zombie[1] <= 0) {
                zombiesRows[i].splice(k, 1);
              }
              return true;
            }
            return false;
          })
          
          // diagonal lanes
          // diagonal up
          let iDia = i
          let jDia = j
          while (true) {
            iDia--;
            jDia++;
            if (iDia < 0 || jDia > lawn[0].length-1) break
            // check if any zombies on this place
            let zombieShot = false; 
            zombiesRows[iDia].some((zombie, k) => {
              if (zombie[0] == lawn[0].length-1 - jDia) {
                zombie[1] -= 1;
                // check if this killed him and remove him if so
                if (zombie[1] <= 0) {
                  zombiesRows[iDia].splice(k, 1);
                }
                zombieShot = true;
                return true;
              }
              return false;
            })
            if (zombieShot) break;
          }
          
          // diagonal down
          iDia = i
          jDia = j
          while (true) {
            iDia++;
            jDia++;
            if (iDia > lawn.length-1 || jDia > lawn[0].length-1) break
            
            // check if any zombies on this place
            let zombieShot = false;
            zombiesRows[iDia].some((zombie, k) => {
              if (zombie[0] == lawn[0].length-1 - jDia) {
                zombie[1] -= 1;
                // check if this killed him and remove him if so
                if (zombie[1] <= 0) {
                  zombiesRows[iDia].splice(k, 1);
                }
                zombieShot = true;
                return true;
              }
              return false;
            });
            if (zombieShot) break
          }
        }
      if (i === lawn.length-1) {
        i = -1;
        j--;
      }
      i++;
    }
    
    // update all zombie moves by one
    zombiesRows.forEach((zRow, i) => zRow.forEach(zombie => {
      zombie[0]++;      
      let j = lawn[0].length-1-zombie[0];
      // check if zombie got onto some plant
      if (lawn[i][j] !== ' ') lawn[i][j] = ' ';
    }))
    // update all zombies that are waiting to get on board
    zombies.forEach(zombie => {
      zombie[0]--;
    })
    moves++;
    
    // if zombieRows all empty or any zombie has move to 0
    noZombies = zombiesRows.every(zRow => zRow.length === 0) && zombies.length === 0;
    const zombieMoveIsZero = zombiesRows.some(zRow => zRow.some(zombie => zombie[0] == lawn[0].length));
    if (noZombies || zombieMoveIsZero) break
    }
    return noZombies ? null : moves;
    }
