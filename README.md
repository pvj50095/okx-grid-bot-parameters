# OKX Grid Bot Guide: How Spot and Futures Grids Work, How to Fill In the Parameters, and What Happens When Price Leaves Your Range

The OKX grid bot comes in two forms, and choosing the wrong one is the most common way this tool goes badly. Spot Grid splits your starting capital between the two currencies of a pair, divides that capital evenly across the price levels you set, and places a sell order one grid above every buy that fills. Futures Grid builds the same ladder out of perpetual contracts, comes in long, short and neutral versions, and accepts leverage up to 100x. Both are range tools that earn from price oscillating inside the band you choose, and both stop placing new orders the moment price steps outside it. What follows covers how each parameter turns into an actual order, a full walkthrough of OKX's own 5,000 USDT example, the asymmetric behaviour of the trailing grid, and the difference between adding margin and adding position size.

> **Quick verdict**
> - You want spot exposure with no liquidation risk: **Spot Grid**.
> - You have a direction and can survive a forced close: **Futures Grid**, long or short.
> - You expect chop but have no directional view: **Neutral Futures Grid**, at low leverage.

👉 [Open the OKX grid bot and pick Spot or Futures Grid](https://okx.com/join/CASH20)

## Spot Grid vs Futures Grid at a glance

|  | Spot Grid | Futures Grid |
| --- | --- | --- |
| **What it trades** | Spot pairs; capital is split between the base and quote currency | All USDT perpetuals, plus some USDC and coin-margined contracts |
| **Direction** | Long-only by construction, since you hold the asset | Long, short or neutral, chosen at creation |
| **Leverage** | None | Up to 100x; total amount = invested margin × leverage |
| **Parameters unique to it** | Asset (base, quote or both), amount, trailing grid | Invested margin, leverage, open position on creation, estimated liquidation price for long and short |
| **TP/SL behaviour** | Bot stops and sells all base assets at market | Bot stops and closes all positions with a market order |
| **Main risk** | Price sits below your lower limit while you hold the asset | Same, plus forced liquidation once leverage is applied |

## Spot Grid: what your money actually does

Spot grid mode takes your starting capital and splits it between the two assets of the pair you picked. It then generates grid price points between your upper and lower bounds and divides the starting funds **equally between the number of grids**. Every time price reaches an upper grid line, the bot sells a portion of the asset; every time it reaches a lower line, it buys. The profit comes from the spread between those executed buys and sells as price swings back and forth.

You reach it on the web through Trade → Trading bots → Marketplace → Spot grid, and in the app through Trade → Trading bots → Spot grid.

### Four ways to fill in the parameters

- **Set myself:** your own analysis, your own numbers.
- **Set myself - Auto fill:** the bot fills in parameters recommended by a backtested strategy.
- **AI strategy:** recommended parameters based on a **weekly backtest of that pair**.
- **Lead bots:** one-click copy of a lead trader's running bot from the Marketplace.

The AI strategy option deserves a closer look than it usually gets. Its parameters come from one week of backtesting on that specific pair, which makes them a description of what suited the last seven days of that market. They are not recalculated against live conditions as the bot runs. If you use them, treat them as a starting point you still have to sanity-check against the range you expect price to hold.

### The ten parameters, in order

1. **Lower limit:** below this price the bot stops placing orders.
2. **Upper limit:** above this price it stops placing orders.
3. **Grid quantity:** how many slices the range is cut into. A 100–400 range in arithmetic mode with 3 grids becomes 100–200, 200–300, 300–400.
4. **Arithmetic grid:** equal price spacing, as in 1, 2, 3, 4.
5. **Geometric grid:** equal percentage spacing, as in 1, 2, 4, 8.
6. **Asset:** fund the bot in the base currency, the quote currency, or both.
7. **Amount:** what you put in. The ceiling is the maximum transferable amount of that currency in your trading account.
8. **Take-profit price:** reaching it stops the bot, which sells all base assets at market.
9. **Stop-loss price:** falling to it stops the bot, which sells all base assets at market.
10. **Trailing Grid (Up/Down):** shifts the grid automatically as price moves.

Once created, the initial funds sit isolated from your trading account and are used only by that bot.

👉 [Set up a Spot Grid with your own range and grid count](https://okx.com/join/CASH20)

## The official example: 5,000 USDT across 50 grids

OKX's own documentation uses BTC/USDT with a **50,000 to 100,000 USDT** range, **50 grids**, arithmetic spacing, **5,000 USDT** invested, and a BTC price of **60,100 USDT** when the strategy starts. It is an illustration of order mechanics, not a projection of returns.

**Stage 1, initial placement.** The system calculates each level from 50,000 up to 100,000 in 1,000 USDT steps and places a buy order at each one. When a buy fills, a sell order opens one level above it. At launch that means 50 buy orders running from 50,000 to 99,000. With the market sitting at 60,100, every buy order from 61,000 upwards fills immediately, and a sell order is raised for each of those fills. The final layout is a buy order at each level from 50,000 to 60,000 and a sell order at each level from 62,000 to 100,000.

**Stage 2, running.** Price drops below 60,000 and the buy order there fills, with a sell order opening at 61,000. Price rises above 62,000 and the sell order at 62,000 fills, with a buy order opening again at 61,000. That loop is the whole strategy.

**Out of range.** You can edit the range, terminate the strategy, or wait for price to come back. Terminating cancels every pending order, closes the position at market, and credits the assets back to your trading account automatically.

## Trailing Grid behaves differently going up and going down

Most explanations stop at "it follows the price". The two directions do genuinely different things to your capital.

**In a rising market** the bot cancels the lowest pending order and places a new one a grid above the highest level. The width of the range stays the same and the whole band slides up. Take 5 grids between 25,000 and 30,000 USDT with 1,000 USDT spacing: if BTC/USDT breaks above 30,000, the 25,000 buy is cancelled and a new order appears at 31,000. Your committed capital stays roughly where it was.

**In a falling market** the bot only adds an order below the lowest grid, so the range stretches downward and the grid count grows. Same setup, price drops below 25,000, and an order goes in at 24,000. You now have 6 grids instead of 5. More of your money gets committed exactly when the position is moving against you, and the floor of the range walks down toward whatever stop-loss you set. A stop-loss placed below your original lower limit does not move with the grid, so the cushion you planned shrinks every time the trailing grid extends downward. That asymmetry is the part worth deciding on before you enable the feature, not after.

## Futures Grid: the same ladder with direction and leverage

Futures grid trading places contracts at preset intervals inside a price range and profits from price moving through those levels. OKX describes the essence as volatility arbitrage, best suited to a market that is sideways or oscillating within a band. It supports all USDT contracts, along with some USDC contracts and coin-margined contracts.

Direction is a separate choice, and it changes what the bot does at creation:

- **Long Futures Grid:** enters long positions at market when created and takes profit by closing them as price rises. Fits a bullish view inside a range.
- **Short Futures Grid:** holds only short positions and profits by closing them as price falls. Fits a bearish view inside a range.
- **Neutral Futures Grid:** holds both sides, shorting above the base price and going long below it.

Leverage can be set up to **100x**. The amount you actually commit is the invested margin, used only by that bot once it starts, and the total amount is **invested margin × leverage**. There is also an option to open the first position on creation, which is how a long grid ends up holding partial positions at market the moment it goes live.

The two liquidation figures in the setup screen, **Estimated liquidation price (Long)** and **(Short)**, assume that every long or short order in the grid gets filled. They are boundary values for the worst-case fill, not a live liquidation price that updates with each order.

### Increase Positions and Adjust Margin are not the same button

These two sit next to each other in the management panel and get confused constantly.

**Increase Positions** raises the amount allocated per grid above its original value, so every grid from that point forward carries more money. You are adding exposure, and the per-grid risk grows with it.

**Adjust Margin** adds margin to reduce liquidation risk without touching the amount allocated per grid. Nothing about your order sizes changes; you are only widening the distance between your position and a forced close.

If price is drifting toward your estimated liquidation level, the second button is the one that addresses that. The first makes the problem bigger while adding capital at the same time.

## When price leaves the range

Below the lower limit, the bot stops placing new orders. If price keeps falling and never returns to the band, the assets it bought on the way down sit there losing value. OKX's own recommendation is to place a stop-loss price below the lower limit for exactly this case, and with futures leverage the same scenario can end in forced liquidation.

A stop-loss here terminates the bot. It does not pause it, and it does not hold the position for a recovery. Reaching the stop-loss price sells all base assets at the corresponding market price, or closes all positions with a market order on the futures side, and the funds return to your trading account. Restarting means creating a new bot, which the **Replicate parameters** button makes quick, but the old position is gone.

Three more things that only show up in the risk notes:

- Funds moved into a grid bot are isolated from your trading account and used independently, so the rest of your account's position risk is still yours to watch.
- Manual stop and stop-loss both sell at market, so the price you get is the price at that moment, not the level you typed in.
- If the underlying asset is suspended or delisted while the bot runs, the bot stops automatically.

👉 [Create your bot with the stop-loss already filled in](https://okx.com/join/CASH20)

## What the bot costs

There is no subscription tier for grid bots. Costs come from trading fees charged on each order the bot fills, priced by your account's fee tier and by whether the order is maker or taker. The grid documentation doesn't restate those rates or state whether any separate bot fee applies, so check the official fee schedule before you size a position, especially on a futures grid where leverage multiplies the notional value each fee is calculated on. A bot that fills dozens of small orders a day pays that fee dozens of times.

## Who should skip it

Grid bots do well when price oscillates inside a band and poorly when it doesn't. In a strong one-way trend, a spot grid either keeps buying into a decline until it runs out of range or sells its holdings on the way up and watches the rest of the move from the sidelines. Neither outcome is a malfunction; it's the strategy doing what it was built to do in conditions it wasn't built for.

Skip the futures grid entirely if you can't monitor margin or aren't prepared for a forced close. Skip both if you don't yet have an exit plan for the case where price never comes back to your range. The bots are free to create, but the position they hold is real money.

## FAQ

**Where do the AI strategy parameters come from?**
A weekly backtest of the pair you selected. They reflect what fit the previous week, not a live adaptation to current conditions.

**Can I change the range and grid count after creating a bot?**
On Spot Grid, yes. OKX supports editing the preset price range and grid quantity after creation. The Futures Grid management panel lists Increase Positions, Adjust Margin, Stop, Details and Replicate parameters, and doesn't document the same edit option.

**What happens to my money when the bot stops?**
All pending orders are cancelled, assets are sold or positions closed at market, and the proceeds return to your trading account.

**Is a higher grid count always better?**
OKX's documentation doesn't publish a maximum grid count, so the number you can actually enter is whatever the order form accepts at that moment. More grids means tighter spacing, smaller orders and thinner profit per completed cycle, plus more fee-paying fills. Match the count to how much price movement you realistically expect inside the range.

**Can a Futures Grid be liquidated?**
Yes. OKX's risk notes state that a position can be forcibly liquidated if price keeps falling without returning to the range and leverage is applied.

**Does the bot keep running if the coin gets delisted?**
No. Suspension or delisting of the underlying asset stops the bot automatically.

## Which one to open

- **Spot exposure only, no leverage, no liquidation:** Spot Grid with Set myself, a range you can defend, and a stop-loss below the lower limit.
- **A directional view inside a range:** Futures Grid in long or short mode, leverage low enough that the estimated liquidation price sits far outside your lower limit.
- **No view, expecting chop:** Neutral Futures Grid, same leverage caution.
- **No view and no appetite for parameter setting:** Lead bots on the Marketplace, copying a running strategy rather than guessing a range.

Before you fund anything, confirm two things: that OKX serves your jurisdiction and your account is fully verified, and what your actual maker and taker rates are on the fee page at the moment you trade. Neither is something a tutorial can answer for you.

👉 [Start from the OKX grid bot page and create your first bot](https://okx.com/join/CASH20)
