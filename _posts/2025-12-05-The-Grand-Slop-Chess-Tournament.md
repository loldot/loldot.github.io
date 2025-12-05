---
layout: post
title: The Grand Slop Chess Tournament
author: Lorentz Vedeler
date: 2025-12-05
tags:   
    - Vibecoding
    - Development
---

Inspired by Sebastian Lagues Chess bot competition where he made a framework for writing a chess engine and challenged his viewers to make
the best search engine they could within a maximum limit of C# tokens, I ripped out the innards of my chess engine and invited all my imaginary
friends to make their own implementation. Gemini, Claud, Codex and Grok entered to make the best Slopfish version and win the Grand Slop Chess
Tournament 🏆. I decided to not put any other constraints other than that the whole engine must be written in one go, my assumption being
that more features and more code would also mean more bugs.

![The Grand Slop Chess Tournament](/assets/imgs/tgsct.png)

## Warmup round

At first, I had forgotten to remove some tests referring to the removed search function, causing a build failure in the test project.
I can only assume Gemini has never experienced such a failure in its corpus of Googles divine code, and hence was completely derailed
by experiencing this subpar engineering for the first time. It did eventually spit out an implementation which threw away all of its
pieces as soon as it got the chance.

Claude Sonnet was seemingly unaffected by my test project blunder - it didn't need to run any tests. Claude blasted out a chess search function
with alot of advanced pruning techniques in a couple of seconds. When I tested it on my list of test positions, it found the best move in 9/14
positions, an incredibly strong result for an engine that does not have a lot of work behind it. These positions are hard to solve for computers
because they often require a string of sacrifices in an already winning position. Even Stockfish at depth 25 fails to find the correct moves for
all of the positions. It seemed like the tournament was all but decided before it began. Imagine my surprise when I saw the bot in action, losing
all of its pieces immediately. It turns out that my position check should really check the whole line, not just the first move &mdash; Claude's
engine was just very happy to sacrifice its most valuable pieces for no reason.

Codex on the other hand, managed to reimplement all the missing classes and actually make a bot that played at least a few chess moves that did not
instantly lose the game. Kudos to Codex. It won the warmup round on walkover.

## Round one

I decided to cleanup my mess, and let Gemini and Claude try again from scratch. I also let Grok test its ability with an implementation using
Clines agent mode. I let Codex submit its engine again, since it had actually produced something in the warmup round - perhaps giving it an
unfair disadvantage. I may have to let it try again and run another round.

I am not one to keep you in suspense, so here are the results of the 300 game tournament:

| Rank | Name               |  Elo | +/- | Games | Score | Draw  |
|------|--------------------|------|-----|-------|-------|-------|
|  1   | Slopfish (Claude)  |  136 |  55 |  150  | 68.7% | 16.0% |
|  2   | Slopfish (Gemini)  |   28 |  51 |  150  | 54.0% | 17.3% |
|  3   | Slopfish (Grok)    |  -61 |  54 |  150  | 41.3% |  9.3% |
|  4   | Slopfish (Codex)   | -100 |  51 |  150  | 36.0% | 21.3% |

## Claude

[Source](https://github.com/loldot/lolbot/commit/9c45c4f8e8828b9cfc81fc376a2ce1851ac4b457)

Like all the other bots, Claude implemented a principal variation search function with iterative deepening and a quiesence search.
It also implemented an enhanced move ordering function, scoring captures by MVV-LVA, and moving losing captures after quiet moves.
Claude more or less implemented the [Simplified Evaluation Funcion][1] from Chess Programming Wiki.

In addition, Claude added aspiration windows, null-move pruning, late move reductions and a quiesence search with delta pruning.

Both Claude and Grok decided to change the code for the timer, even though I explicitly specified not to modify any other existing
files in the prompt.

After running the tournament with the slop bots, I ran a tournament with some real opposition. Claude's engine became the clear
loser, winning just 10 / 644 games. However that gave me a decent rating estimate of around 1700.

<iframe src="https://lichess.org/embed/game/9MLV9uMy?theme=auto&bg=auto#23"
width=600 height=397 frameborder=0></iframe>

In this game against Gemini, Claude has allowed black to gain a pawn advantage with a tempo on whites bishop, but Stockfish actually
thinks the position is about equal. Claude then plays a very questionable move, Ng5, forcing black to castle and delaying the decision
on how to protect its exposed bishop. Blacks knight is a bit awkward, but Claude does not really have a much better position. Claude
then continues to recklessly exchange a minor piece for two pawns. Gemini quickly manages to grab back these pawns while trading
queens and rooks. In the end Gemini is up a knight in a 3v3 pawn end game, which it easily wins.

## Codex

[Source](https://github.com/loldot/lolbot/commit/ed3dd1ca6bb7f2df885a945ac86e064c627523c3)

Hampered by my broken test suite, Codex decided to skip the most common positional evaluation criteria; piece square tables. Instead
Codex implemented more complex evaluations, like king safety and pawn structure right off the bat. It also spent some tokens to
implement code for static exchange evaluation, but that was never actually used in the search.

In this game against Claude, black actually seems a little better out of the opening. White's rook is stuck and will take a long time
to get in the game &mdash; unless of course black decides to help him develop and lose its bishop while he's at it. As white is
expected to win here, Claude makes quick progress (even though it misses a mate in 11 which Stockfish finds). The game ends with a
cute mate.

<iframe src="https://lichess.org/embed/game/L1GgwdAi?theme=auto&bg=auto#22"
width=600 height=397 frameborder=0></iframe>

## Gemini

[Source](https://github.com/loldot/lolbot/commit/2ee38ad1714178c148343ad400e548c449428b2d)

Weighing in at only 368 lines of code and a very simple feature set, Gemini really punched above its weight in this tournament ending
up in a solid second place.

In this game, Gemini mistakenly thinks the game is a draw. I suspect this is due to cancelled searches getting evaluated to 0, if
such a score ends up in the transposition table it will mess up future searches as well.

<iframe src="https://lichess.org/embed/game/CMIBv8eE?theme=auto&bg=auto#30"
width=600 height=397 frameborder=0></iframe>

## Grok

[Source](https://github.com/loldot/lolbot/commit/c02acfd3553784298b529994515191b9c9f61d19#diff-0ebd21ca04879c0a184404a1a8c2f1a403d656af5ebb0b6e7036b5c2f5c9924e)

With all the techniques Grok managed to implement, it should have been a strong contender. Unfortunately it had some
crippling bugs, I suspect with either the transposition table or how it handled cancelled searches, perhaps a combination.
In many of its games the engine correctly announced mate-in-X only to continue with a complete non-move, like moving its
queen over one square after running a one ply deep search.

<iframe src="https://lichess.org/embed/game/tUnlBlxU?theme=auto&bg=auto#119"
width=600 height=397 frameborder=0></iframe>

In this game against Codex, Grok has the advantage from the beginning. Codex forgot its rook in the corner, leading to a
position where Grok has a forcing sequence to take whites bishop. Then Grok decides to give up its bishop for a pawn with
absolutely no compensation. Black is still up in material, and after shuffling pieces around randomly for a while, manages
to convert to an easily winning endgame. Only to do some more shuffling around before it eventually draws with a threefold
repetion.

## Feature Comparison

| Feature                           | Codex | Claude | Gemini | Grok |
|-----------------------------------|-------|--------|--------|------|
| **Search**                        |       |        |        |      |
| Iterative Deepening               |✅    |      ✅|     ✅|    ✅|
| Aspiration Windows                |❌    |      ✅|     ❌|    ✅|
| Principal Variation Search        |✅    |      ✅|     ✅|    ✅|
| Null-Move pruning                 |❌    |      ✅|     ✅|    ✅|
| Late Move Reductions              |❌    |      ✅|     ❌|    ✅|
| Quiesence Search                  |✅    |      ✅|     ✅|    ✅|
| Delta pruning                     |❌    |      ✅|     ❌|    ✅|
| **Evaluation**                    |       |        |        |      |
| Piece Square Tables               |❌    |      ✅|     ✅|    ✅|
| Mobility Bonus                    |❌    |      ❌|     ❌|    ✅|
| Bishop Pair Bonus                 |✅    |      ✅|     ❌|    ✅|
| King Safety                       |✅    |      ❌|     ❌|    ❌|
| Pawn Structure                    |✅    |      ❌|     ❌|    ❌|
| **Move Ordering**                 |       |        |        |      |
| Move from transposition table     |✅    |      ✅|     ✅|    ✅|
| MVV-LVA                           |✅    |      ✅|     ✅|    ✅|
| Bad Captures                      |❌    |      ✅|     ❌|    ❌|
| Killer Move Heuristic             |✅    |      ✅|     ✅|    ✅|
| History Heuristic                 |✅    |      ✅|     ✅|    ✅|

[1]: https://www.chessprogramming.org/Simplified_Evaluation_Function