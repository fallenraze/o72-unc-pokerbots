# Auction Hold'em Pokerbot: Strategy Overview

By the NC State team: Aryan Shah, Jason Lu, Aadhir Sandeep, Aryan Goyal

Our bot combines near-GTO preflop play, Monte Carlo hand evaluation, and adaptive bluffing and bluff-catching that responds to each opponent's tendencies. Here's how it works.

## Core Strategy

**Preflop.** We wanted to play as close to GTO as possible. Since suits don't affect preflop strength, we store hands by type (`AAo`, `AKs`, etc.) instead of exact cards, which cuts 52×51 = 2,652 combinations down to 169. Hands are ranked from 1 (`AAo`, strongest) to 169, and we raise or fold based on rank, since calling shows weakness. If action continues past a big-blind call, our calling range shrinks with roughly the cube root of the opponent's bet size.

**Auction bidding.** A Monte Carlo simulation estimates our win probability with and without winning the auction, and we size our bid from the difference. We later adjusted it to account for straight and flush draws.

**Postflop.** Each trial deals the opponent two or three random cards (depending on the auction), completes the board, and compares hands using `eval7`. After 125 trials we have a win percentage, which we weigh against pot equity (the call amount divided by the pot minus the call amount). A random number generator then picks between raise, call, check, and fold so we stay unpredictable.

## Exploiting Opponents

**Check bluffs.** We found that many of our opponents played tight and almost never checked with strong hands, so we bluff more each time they check:

| Opponent checks | Our bluff frequency |
|---|---|
| 1 | 30% (slightly lower as big blind) |
| 2 | 75% as big blind, 100% as dealer |

This worked especially well against teams that didn't use ML to adjust during the match. We track the plus-minus from our bluffs, and if it falls below a threshold we drop to ¼ of our normal bluff rate. That protects us against bots that rarely fold.

**Bluff catching.** Midway through, we realized opponents were bluffing like we were. We noticed they bluffed most when they bet right after winning the auction, or right after we checked. Each time we suspect a bluff, we raise an **"unnit" factor** that is subtracted from the pot equity needed to call, so we call more often. A few rules keep it in check:

- Opponents flagged as **aggressive** (frequent pot-sized bets) raise the factor further.
- It rises more against a three-card opponent, since we figured they'd be more aggressive after winning the auction.
- We track how these calls perform and halve the factor if they're losing too much.
- Stacked factors are capped at **0.25**.

**Semi-bluffs.** We bet larger than usual when on a flush or straight draw. In `eval7` these draws usually hit above 25%, so that's our threshold. It started as a way to gauge opponent aggressiveness, but it proved effective against most opponents, so we kept it on for the entire game.

## Time and Risk Management

- **Adaptive simulation depth.** Trials drop from 125 to 100 to 50 as we run low on time.
- **"Nit" factor.** To offset the lower accuracy of fewer trials, we reduce hand strength by 3% (100 trials) or 6% (50 trials) to avoid loose calls.
- **Big-pot discipline.** Once our contribution passes 100, we subtract another 3% from hand strength so we only continue with strong hands.

## Final Tuning

Before the competition, we widened our preflop raising range to counter loose opponents and reordered the hand dictionary to rank suited, connected, and suited-connected hands higher.

## Timeline

| Week | Focus |
|---|---|
| 1 | Baseline strategy: preflop, postflop, auction bidding |
| 2 | Check bluffs, bluff tracking, adaptive trial counts |
| 3 | Bluff catching, aggression detection, unnit factor |
| 4 | Semi-bluffs, preflop range adjustments |