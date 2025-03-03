---
layout: post
title: 'PEXT Tables'
author: Lorentz Vedeler
date: 2025-03-03
tags:   
    - Chess Engines
    - Development
---

## Introduction
Chess engines need to be incredibly fast. A common way to achieve fast generation of moves from a chess position, is to pre-calculate all the moves available for each piece type, from each square on the board, but disregarding pins and checks, this is usually called pseudo-legal moves. For pawns, knights and the king, this is fairly straightforward. Rooks, bishops and queens are a different matter; the moves available from each square is also dependent on where the other pieces are placed on the board.

## Bitboards
One of the most common ways to represent the board is to store the location of pieces in several 64 bit integers. This technique, called bitboards, relies on the fact that we can flip a bit for each square on the board to indicate whether a piece is placed on a square. For example if a piece is moved from A1 to E1 we would flip the first bit to 0 and the 5th bit to 1. 

A bitboard of the white pawns in the starting position would look like this: 0b00000000_00000000_00000000_00000000_00000000_00000000_11111111_00000000; or perhaps more legible:

<pre>
00000000
00000000
00000000
00000000
00000000
00000000
<span style="color:red">11111111</span>
00000000
</pre>

We can also use bitboards to represent the pseudolegal moves for a piece from a position. For example a rook on D4 on an empty board could represent its legal moves with the bitboard:

<pre>
000<span style="color:red">1</span>0000
000<span style="color:red">1</span>0000
000<span style="color:red">1</span>0000
000<span style="color:red">1</span>0000
<span style="color:red">111</span>0<span style="color:red">1111</span>
000<span style="color:red">1</span>0000
000<span style="color:red">1</span>0000
000<span style="color:red">1</span>0000
</pre>

A huge advantage of bitboards is the fact that a lot of set operations become incredibly fast - you can use bitwise operations: bitwise and (`&`) is equivalent to set intersection and bitwise or (`|`) is equivalent to union.

You can use https://bitboard-editor.vercel.app/ to play around with bitboard representations.

## Precomputing moves

Knights are by far the easiest pieces to precompute moves for. They jump over other pieces, the only restriction is that they cannot move to a square occupied by a friendly piece. A single array of 64 bitboards, one for each potential square on the board, can hold all the (almost) pseudo-legal moves for knights of both colors. Each bitboard should assume the board is empty, to get the true pseudo-legal moves for a position, we can use a bitboard for the pieces occupied by each color and use bitwise operations to exclude moves which would land on friendly pieces. 

King moves are pretty much the same as knight moves, but also have castling. Pawns are a bit more tricky as they have en-passant and the option to move two squares from their initial position. 

Then there's the bishops, rooks and queens, the so called sliders. They are another beast, they can be blocked by the other pieces on the board. With the number of possible chess positions estimated to be on the order of 10^40 give or take, it seems impossible to pre-compute all the slider moves. 

Our first insight is that the queens pseudo-legal moves from a square, are just the union of pseudo-legal rook moves and pseudo-legal bishop moves from that square. We know that unions on bitboards are really cheap, so we only need to store rook and bishop moves.

The next insight is that most of the pices in a position are irrelevant to the moves available. I.e. for a rook, only pieces on the same rank and file are relevant, and for the bishop, only pices on the same diagonal and anti-diagonal are relevant.

Next, it doesent matter which piece is blocking our slider if we assume that we can capture all blocking pieces. We can then use the same trick we used with our knight lookup, we just discard moves that would capture a friendly piece when we retrive the bitboard.

These insights should reduce the storage requirements by a lot. The question is, how do we compute the index for our precomputed moves?

## Two peculiar instructions

The x86 architecture have two extensions for bit manipulation, BMI 1 and 2. Most modern CPUs from Intel and AMD support these instruction sets. The Bit Manipulation Instruction Set 2 contains two instructions with cryptic names; PEXT and PDEP. These operations can be a bit tricky to understand, especially if you read the offical descriptions. PEXT or parallel bit extract will extract the bits of an operand according to a mask. It is easier to understand with an example:

Assume we want to use the pext instruction to extract bits from a byte according to a mask `0b11001010`. The value of the operand does not matter for this example, in stead we will label the bits in the byte with a letter from a through h.

`pext(abcdefgh, 0b11001010) = 0000abeg`

PDEP or parallel bit deposit is the inverse of parallel bit extract. Meaning we can "spread" the bits in a number over the mask. 

`pdep(0000abcd, 0b10101010) = a0b0c0d0`

It turns out that these two instructions provide an elegant and efficient primitive for indexing our table of pre-computed bishop and rook moves.

## Connecting the dots

Let's say we just want to create the moves for a rook on the A1 square. The bitboard for the pseudo-legal moves on an otherwise empty board would be:

<pre>
<span style="color:red">1</span>0000000
<span style="color:red">1</span>0000000
<span style="color:red">1</span>0000000
<span style="color:red">1</span>0000000
<span style="color:red">1</span>0000000
<span style="color:red">1</span>0000000
<span style="color:red">1</span>0000000
0<span style="color:red">1111111</span>
</pre>

We will call this the move mask from a1. If we count the number of bits set in the move mask from a1, we get 14. Meaning that there are 2^14 or 16384 possible ways pieces can be placed along the move mask to block our rook.

Next we need to generate a bitboard of occupied pieces for every possible combination of blockers along the move mask. We can achieve this by counting from 0 to 2^14 and using the PDEP instruction to "spread" our number along the move mask.

Now assume we already have a function for generating the pseudo-legal moves. If you want don't want to implement this on your own, there are multiple articles on chess programming wiki explaining different approaches to this, for example: https://www.chessprogramming.org/Sliding_Piece_Attacks and https://www.chessprogramming.org/Efficient_Generation_of_Sliding_Piece_Attacks

Filling up the table for A1 becomes pretty simple:

```csharp
// An array containing a bitboard of pseudo-legal moves from A1 for
// all possible, relevant configurations of other pieces on the board. 
ulong[] A1_RookMoves = new ulong[16384];

// This is the hex value of the move mask shown above
ulong RookA1_MoveMask = 0x1010101010101fe;

for (int i = 0; i < 16384; i++)
{
    ulong blockers = Pdep(i, RookA1_MoveMask);
    A1_RookMoves[j] = GenerateRookAttacks(sq, blockers);
}
```

To get the index in our lookup table for any position with our rook on A1, we can use PEXT with the bitboard of the occupied squares and the `RookA1_MoveMask`.

Doing this for each of the 63 other squares on the board and then 64 more times for bishops, we have our PEXT table.

If you do this, you will realize that the table becomes very big. Luckily there is another optimization we can do to reduce the memory footprint of our lookup tables.

Remember that we assume all blockers are enemy pieces and thus can be captured. Therfore, when the blockers are along the edge of the board, the pseudo-legal moves will be the same as on an empty board. We can therefore ignore edges on our move mask. 

Below is the full implementation of pext tables in my chess engine:

```csharp
// The bitboards class can be found here: https://github.com/loldot/lolbot/blob/main/Lolbot.Engine/Core/Bitboards.cs
public static ulong[] RookPextMask = new ulong[64];
public static ulong[] BishopPextMask = new ulong[64];
public static uint[] RookPextIndex = new uint[64];
public static uint[] BishopPextIndex = new uint[64];

public static ulong[] PextTable = new ulong[107_648];


public static void InitPextTable()
{
    var sw = Stopwatch.StartNew();
    uint currentIndex = 0;
    for (byte i = 0; i < 64; i++)
    {
        var sq = Squares.FromIndex(i);

        RookPextIndex[i] = currentIndex;
        RookPextMask[i] = Rooks[i] & GetEdgeFilter(i);

        var mask = RookPextMask[i];

        ulong max = 1UL << Bitboards.CountOccupied(mask);
        for (ulong j = 0; j < max; j++)
        {
            var blockers = Bitboards.Pdep(j, mask);
            // Note that this implementation of generating attacks uses the empty bitboard instead of occupied, hence the inversion of blockers.
            PextTable[currentIndex++] = GenerateRookAttacks(sq, ~blockers);
        }
    }

    for (byte i = 0; i < 64; i++)
    {
        var sq = Squares.FromIndex(i);

        BishopPextIndex[i] = currentIndex;
        BishopPextMask[i] = Bishops[i] & GetEdgeFilter(i);

        var mask = BishopPextMask[i];
        ulong max = 1UL << Bitboards.CountOccupied(mask);
        for (ulong j = 0; j < max; j++)
        {
            var blockers = Bitboards.Pdep(j, mask);
            PextTable[currentIndex++] = GenerateBishopAttacks(sq, ~blockers);
        }
    }

    Console.WriteLine($"info Pext init took {sw.ElapsedMilliseconds} ms");
    sw.Stop();
}

private static ulong GetEdgeFilter(byte i)
{
    var result = (Bitboards.Masks.Rank_1 | Bitboards.Masks.Rank_8) & ~Bitboards.Masks.GetRank(i);
    result |= (Bitboards.Masks.A_File | Bitboards.Masks.H_File) & ~Bitboards.Masks.GetFile(i);

    return ~result;
}
```

