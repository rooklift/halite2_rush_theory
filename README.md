# Theory and Practice of Halite 2 Rushes

My [Halite 2 bot](https://github.com/fohristiwhirl/gohalite2) is ordinary in many ways, but in one way it's special: I spent a lot of effort on the 1v1 melee code. In the situations it was designed for, it's one of the best bots there is. Sadly, these situations only arose in about 13.77% of games...

At the time of writing (Jan 10th), looking at my last 10,000 games, exactly 1,377 of them could be considered 1v1 rush games (2 players only, 10 or fewer ships produced). In those games, my win/loss record is **1090-287**.

## Theory: 6 Ship Battles

The case we are most interested in is where 3 ships from each team are in close proximity. What we want is to do more damage to the enemy ships than they do to us. However, players make moves simultaneously, so it is impossible to make moves that are guaranteed to do this.

On the other hand, it *is* possible to make moves that can't possibly lose, but which might win if the opponent does the Wrong Thing, which is a common event.

Consider the following. Every ship can -- by moving at speed 7 -- do damage to enemies that are up to 13 units away from its starting location, because weapon range is effectively 6 (measuring centre to centre).

Therefore, certain sweet spots emerge on the map where *at most* one enemy ship can come into range. If we put all our ships in those sweet spots, we can't lose, but we might win.

![Sweet Spots](https://raw.githubusercontent.com/fohristiwhirl/scraps/master/ranges.gif)

On the left are the threat ranges of the three Blue ships - i.e. how far they can do damage *after they move*. On the right are highlighted the sweet spots available for the Pink ships (though only the bottom one is relevant).

When Pink places his ships in a sweet spot, one of two things will happen:

* Blue will take the same amount of damage as Pink (possibly zero).
* Blue will take more damage than pink, because one of Blue's ships came within range of two (or three) Pink ships.

## Real World Example

Note that the sweet spots involved are often fairly small. In the diagram below (turn 11 of a [real game](https://halite.io/play/?game_id=7421675)), we need to get our ships into the tiny blue zone (we only care about the centre of the ships though, since we are measuring weapons range centre to centre). They all fit, barely, and the enemy gets obliterated.

![Sweet Spots 2](https://raw.githubusercontent.com/fohristiwhirl/scraps/master/ranges2.gif)

## Practice: Genetic Algorithm

A smarter person than me might use mathematics to put his ships in the sweet spots. However, I chose to use a Genetic Algorithm, which works as follows. First, we generate a random "genome" (list of moves) and then do the following:

* Mutate the genome randomly, giving one of the ships a new move.
* Simulate the result, and score it according to some "fitness" function.
* If the new genome is an improvement, keep it, otherwise discard.
* Repeat.

The fitness function I use is based on getting ships into the sweet spot where possible, or near it otherwise; while not crashing ships into each other, or into planets, or into the game edges. A few lesser factors also come into play.

## Practice: Metropolis Coupling

When I constructed my Genetic Algorithm, I wasn't sure exactly what fitness function I would end up using. But I wanted to avoid local optima. To avoid these, I run multiple chains of evolution at once, with different "heats". Hot chains are allowed to accept bad mutations (the hotter the chain, the looser its standards are). Between iterations, the chains are sorted so that the colder chains have the better genomes. In this way, the cold chains can be pulled out of local optima. I believe this whole process is called "Metropolis Coupling".

Honestly I'm not sure how useful it is. In some rare cases it can find superior solutions.

## Example Games

Ideally, the result should look [something like this](https://halite.io/play/?game_id=7146061). Even rather strong bots can be safely dealt with ([one](https://halite.io/play/?game_id=6987743), [two](https://halite.io/play/?game_id=7102762)).

## Problems

Sometimes two bots will get into a situation where neither is willing to move. I try to detect such situations and then use the fact that we know where they will be to make perfectly destructive moves. I still use the genetic algorithm, but with a different fitness function based on expected damage. [This game](https://halite.io/play/?game_id=7094226) shows the result, at turns 37, 67, and 121.

Sometimes we "win" the rush but the enemy has successfully built a ship and escaped with it. If we chase it forever, he will win on tiebreaker. To avoid this, if possible, I turn off the GA, send one ship to chase the enemy and another ship to dock and build up a fleet, as in [this game](https://halite.io/play/?game_id=7453830).

Our theory doesn't take into account differences in ship health. If a 63 health ship fights a 127 health ship, they will do the same damage to each other (which is an acceptable draw according to our theory) but one will die. In [this game](https://halite.io/play/?game_id=7095394) I temporarily fall behind on ships (though not on total health) at turn 95. I never bothered worrying about this, because it didn't seem to matter.

Our theory of combat suffers from literal edge and corner cases: we will generally be backing away from enemy ships, possibly leading to us running out of space. For example, see [this game](https://halite.io/play/?game_id=7066056) at around turn 67. I fixed this in a later version by making the fitness function dislike being near the edge. Planets sometimes cause the same problem. At turn 11 in [this game](https://halite.io/play/?game_id=7328811), one of my ships can't make any good move because it's come too close to a planet. Again, change the fitness function...

Sometimes the enemy just runs away, as in [this game](https://halite.io/play/?game_id=7069201)...

The biggest problems come when the enemy splits up. For example, I shouldn't have won [this game](https://halite.io/play/?game_id=7226052). This sort of thing was my Achilles' Heel, really. In [this game](https://halite.io/play/?game_id=7361129) at around turn 9, the Genetic Algorithm must split the ships so as to get near both groups of enemies. Unfortunately it makes the wrong choice of how to do this. I couldn't think of a good way to make the fitness function prefer the correct split.

## Defense

If I'm not rushing myself, I defend by noticing I'm getting rushed and undocking. One thing few players spotted is that you can issue a thrust command during the final step of undocking (where `DockedStatus == UNDOCKING`) *and it will work*. You can also issue an undock command during the final step of docking.

These two facts allow me to make useful moves 2 turns earlier than a naive defender; after undocking I simply transition into rush mode.

## 4 Player Games

In 4 player games, there are initially two sub-games: the left side 1v1, and the right side 1v1. Rushes are therefore possible, but there's a sort of prisoner's dilemma: if I rush my opponent, and he defends adequately, we are likely to get 3rd and 4th. If we both play normally, we might both get a chance at 1st or 2nd.

I never rush in 4 player games. I do defend as above, though. Here's a [rare example](https://halite.io/play/?game_id=7286792) of that going as well as possible.
