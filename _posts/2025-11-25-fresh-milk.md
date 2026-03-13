---
layout: post
title:  "fresh milk"
date:   2025-10-01 09:29:20 +0700
categories: jekyll update
usemathjax: true
---


Your Only Move Is Hustle (YOMI Hustle) is a turn-based fighting game that takes the usual turn-based formula (a la Pokemon) and flips it on its head. Users can pick from a pool of available moves from their current position, and they are executed at the same time, sort of like a convoluted game of rock-paper-scissors. The game freezes on the frame when at least one combatant becomes actionable, and allows for actions to be made again.

Again, Yomi is a turn-based fighting game. A game in which strategy and decision making comes before reaction time and techs.

Today, I talk about Fresh Milk, the bug that involves the only frame-perfect input in the game.

Content essentially paraphrased from:

<iframe width="560" height="315" src="https://www.youtube.com/embed/-TQn8ZsN8Dc?si=gnnCALI_DCD5VIdl" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Cowboy

At launch, Cowboy was a monster. Cowboy is armed with a sword, gun and lasso, and possessed 1f instant teleports, being able to deal effective damage at close range, grapple at medium range and shoot at long range, as well as catch up to his opponent really quickly. Horizontal slash was a fast forward attack that outspeeds and outranges many attacks in the game. But patch after patch chipped away at him: his teleports were weakened, his ranged tools nerfed, and his damage output fell behind characters like Wizard, Ninja, and Robot.

Cowboy’s samurai–cowboy aesthetic kept him popular with fans, but competitively he sank to the bottom of the tier list. He was too predictable, too rigid, and lacked explosive options to keep pace with the evolving meta.

That all changed with the introduction of Foresight—a new mechanic meant to fix Cowboy’s movement. Instead, it opened the door to one of the most infamous exploits in fighting game history.

## Foresight

Foresight was a mechanic that limited Cowboy's instant teleports and made it more balanced. Foresight allows you to place an afterimage on action. On future turns, you are allowed to Shift to the afterimage, letting you teleport to it after a few frames of delay. Moreover, since Shift was an additional toggle on top of your current selected action, it made it possible to gap close to an enemy and launch an attack on the same frame. Functionally, it's Flying Raijin from Naruto.

Even better, you can trigger Rift on the foresight afterimage, causing it to explode. Foresight basically lets you zone out an area, keeping your opponent wary of the afterimage. After all, with Cowboy's strong grounded neutrals, Foresight threatens not only the area that you're at, but also the area around your Foresight.

Teleporting in games is a funny thing. When you shift from, say, air to ground, it wouldn't be right to shift with an air move while you reach the ground, right? Essentially, Ivy had it prepared that when a Foresight afterimage was placed on the ground, the moves on the player's UI would toggle to the ground moves, preventing you from using air- or ground-exclusive moves when not in the correct state.

MilkMannerism was a labber that played on the beta branch of Yomi, looking to discover new tech. For example, Foresight preserves your momentum when you shift to it. If you set up with a quick mobility option then shift to a Foresight afterimage, you will appear at the afterimage with the same momentum that you had before, while allowing you to execute different moves at the same time. This greatly extended Cowboy's effective attack range.

However, MilkMannerism, when testing, tried to place another Foresight afterimage after the initial shift. Now, Foresight has limits on how far from the user the afterimage can be placed; normally, it's a set radius. When the user gets too far away from the afterimage, the afterimage will explode (to prevent users from going across the map to stall). What MilkMannerism realised was that even when the afterimage was exploding, if it was placed on the ground, the player's UI would do the ground-air UI shift. But what if you shifted to an afterimage that was already set to explode? MilkMannerism tested it, and found that upon shifting, the player remained in the air, and was even able to execute grounded moves while in the air, effectively bypassing the UI toggle check.

Because you were only able to use the tech when the foresight was expiring, and since it was discovered by MilkMannerism, the tech was coined "Expired Milk".

## Expired Milk

Cowboy's air options are limited. Generally, all of his aerial moves are low-damaging and high knockback. But with access to grounded moves in the air, Cowboy's air combo potential grew immensely. For example, the move Stinger: slow windup, but long poking range and high damaging, and knocking the enemy prone upon hitting the ground. However, it's true usage lies in its property, Infinite Hitstun. Yomi Hustle works on a stun-based system, when you're stunned, you can adjust a Directional Influence (DI) wheel to influence your knockback. DI-ing upward is a great way to escape combos, because your enemies have to use up limited air options to chase you, which will force them to drop the combo. Infinite Hitstun keeps the enemy stunned until they hit the ground. Being able to hit Stinger and turn a combo ender into a combo extender drastically raised damage output.

Even worse, Cowboy has another grounded move, Backslash, which is an upward slash generally used for anti-air. However, Backslash has a sweet spot hitbox that deals extra damage if landed, and is extremely difficult to land on the ground, but very easy to land with the extra positioning in the air.

Expired Milk helped, but wasn't revolutionary, it required a ton of setup (foresight, comboing, positioning, etc)

## Fresh Milk

One day, an innocent message was posted in the discord, which eventually led to a discussion about Expired Milk. A labber called Pilebunker was confused at Expired Milk, apparently he had been doing it a different way the entire time, and just didn't realise. However, the clips he showed were revolutionary; he had heard vaguely about Expired Milk, tested it out, and assumed that what he saw was what everyone else was doing. 

His tech works because YOMI doesn't check the validity of a move before confirmation, because YOMI only puts the moves you're allowed to use onto your UI to begin with (therefore assuming that all moves on your UI are legal). Essentially, the tech works by setting up a grounded foresight while in the air. While still aerial, you activate the foresight Shift toggle (shifting your moves to grounded) and then immediately de-toggle Shift and lock in the move. What this does is confirm the move before the game is able to shift your moves back to aerial, which bypasses the game's check.

This was revolutionary, removing the need to actually transition between aerial and grounded states, and simply using a mechanical input to trick the game into letting you use the wrong moves. Moreover, since Fresh Milk doesn't consume the foresight (hence keeping it fresh), it can be chained as many times as you want. 

Fresh Milk overcame Cowboy's slow ground movement by allowing Cowboy to use his faster air-dash while grounded, letting him match the other characters in terms of grounded mobility. Grounded Lightning Slash only allows you to aim forward, but aerial Lightning Slash can be aimed upward around Cowboy. Aerial blocks are treated as both blocking low and high moves, which removed the risk brought about by simply blocking on the ground, as most attacks could be safely parried. 

## RIP Fresh Milk

Ivy tried and failed multiple times to patch out fresh milk. 

However, this only made the input timing tighter, which made fresh milk still possible on supremely laggy computers and automated macros. For example, by preparing an AutoHotkey script to type 12mb of text into the chatbox at once, the game would lag enough to make fresh milk possible, or even negative edge inputs (holding then releasing keys simultaneously).

Eventually, Fresh Milk was worked into the game officially as Drift, an additional toggle on Foresight. This is only usable if the Foresight afterimage is in the opposite air-ground state from the user, causing the afterimage to explode, and was functionally the same as Fresh Milk, except for the fact that it could be predicted via the game's move system.

This was a really cool addition that's an example of developers turning a bug into a feature.

(Mutant still the better character imo)

## Conclusion

funny milk bug, play the game.



