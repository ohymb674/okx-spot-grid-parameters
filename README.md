# OKX Spot Grid Bot: How It Works, What to Set, and When It Quietly Loses Money (Parameters, Trailing Grid Rules, Fees and Stop Options)

An OKX spot grid bot places a ladder of limit buy and sell orders inside a price range you pick, then recycles those orders as price moves up and down through the levels. It makes money in choppy, range-bound markets, bleeds in one-way trends, and OKX charges nothing extra for running one: bot orders pay the same trading fee as orders you place by hand. When you stop a bot, you can sell the crypto at market or keep it, and either way the funds land back in your trading account.

What follows is the setup side and the failure side. You'll get the parameters and what each one breaks when it's wrong, a worked example using OKX's own published numbers so you can see where the profit actually comes from, the trailing-grid rules that decide whether your stop-loss moves with the market, and the specific situations where you should leave the bot switched off.

> **Quick read:** This suits you if you have a range you believe in for the next few weeks and you're willing to check on it. It does not suit you if you expect a straight-line move up or down, or if you want something you can forget about for a month.
> 👉 [Start a spot grid bot on OKX with invite code CASH20](https://okx.com/join/CASH20)

## The parameters, and what each one breaks if you get it wrong

Every OKX spot grid bot is defined by the same handful of fields. None of them are hidden, but two of them (grid quantity and investment amount) do most of the damage when set carelessly.

| Parameter | What it controls | What goes wrong if you set it badly |
| --- | --- | --- |
| Lower limit | The bottom of the working range. Below it, the bot stops placing orders | Set too high, the bot idles through the dip you wanted to buy |
| Upper limit | The top of the working range. Above it, the bot stops placing orders | Set too tight, you sell out early and sit in cash while price runs |
| Grid quantity | How many price levels the range is cut into | Too many grids, each fill is tiny and fees eat the spread |
| Grid mode | Arithmetic (equal price gaps) or geometric (equal percentage gaps) | Geometric suits wide ranges, arithmetic suits narrow ones; mismatching distorts order sizing |
| Investment amount | Total capital assigned across the grids | Too little leaves grids unfunded and trades get skipped |
| Take-profit price | Bot stops and market-sells all base assets when hit | Set inside the range, it fires early and ends the strategy |
| Stop-loss price | Bot stops and market-sells all base assets when hit | Set too close, normal noise closes the position at a loss |
| Trailing up / down | Lets the grid shift or stretch as price moves | Changes whether your stop-loss follows the grid (covered below) |

The range is the whole strategy. A bot with a 90,000–120,000 USDC range on BTC/USDC does nothing new once price leaves that band. OKX is explicit that the bot won't place orders outside it.

## Three ways to fund the first orders

The initial investment mode decides which direction your first orders go, and it changes what you're actually betting on.

| Mode | First orders placed | What it means in practice |
| --- | --- | --- |
| Quote currency only (e.g. USDC) | Limit buy orders for the base asset | You start in cash and want to accumulate BTC on dips |
| Base currency only (e.g. BTC) | Limit sell orders | You already hold the coin and want to sell into strength |
| Both | Buy and sell limit orders at once | A neutral start; the bot can work in either direction immediately |

Pick quote currency if you're entering fresh. Pick base currency if you're sitting on a position you'd like to trim gradually rather than in one market sell.

There are four ways to fill in the parameters once you're in the Spot Grid screen: set them yourself, use Auto fill (OKX's backtested recommendation), use AI strategy (parameters from a weekly backtest of that specific pair), or copy a lead trader's bot from the Marketplace in one click. AI strategy is a starting point, not a forecast. OKX describes it as backtested parameters, nothing more.

👉 [Open the spot grid setup on OKX](https://okx.com/join/CASH20)

## A worked example, with the arithmetic and geometric maths shown

OKX's own documentation uses BTC/USDC with a 90,000–120,000 USDC range, 6 grids, and 10,000 USDC invested. Here is what those six levels actually are.

**Arithmetic mode** keeps an equal price gap: (120,000 − 90,000) / 6 = 5,000 USDC. The levels are 90,000, 95,000, 100,000, 105,000, 110,000, 115,000, 120,000.

**Geometric mode** keeps an equal percentage gap: (120,000 / 90,000)^(1/6) = 1.04911506. The levels are 90,000, 94,420, 99,058, and so on, each roughly 4.9% above the last.

The two modes diverge in the middle of the range. Arithmetic puts more levels near the bottom in percentage terms, which front-loads buying at lower prices. Geometric spaces levels evenly by percentage, which behaves better over wide ranges where a fixed dollar gap would be meaningless at one end and cramped at the other.

Now the execution, using the arithmetic version with quote currency as the investment mode and an instant trigger at a current price of 100,000 USDC.

**T=0.** Parameters entered: 90,000–120,000, 6 grids, 10,000 USDC, instant trigger, BTC at 100,000.

**T=1.** The bot calculates a buy quantity per grid of 0.01709 BTC. Buy limit orders are placed at every level. Because the current price is 100,000, the orders at the 3rd, 4th, 5th and 6th grid levels are at or above market and fill immediately. The remaining buy orders wait below. The bot also reserves the USDC it will need if price reaches those lower triggers, so that money is locked while the bot runs.

**T=2.** BTC moves to 102,000. The account holds a mix of BTC and leftover USDC, now valued at 10,137 USDC.

**T=3.** BTC reaches 106,000 and the sell order attached to grid 3 fills. Total realized profit is (106,000 − 100,000) × 0.01709 = **102.54 USDC**. Of that, the amount attributed to the grid itself is (105,000 − 100,000) × 0.01709 = **85.45 USDC**. The 17.09 USDC difference comes from the fill happening above the grid's own sell level, and it's yours to trade or withdraw.

**T=4.** That grid cycle restarts: a new buy order goes in at the lower level of grid 3, ready to repeat.

Two things about that walk-through matter more than the profit figure. First, the limit order that filled at 106,000 was placed at the 105,000 level. Grid orders are limit orders, and in a fast move they fill at better prices than the level, which is where the extra 17.09 USDC came from. Second, OKX's example ignores fees and slippage on purpose. Real results are lower, and this is exactly why grid quantity matters so much.

## What each setting does to your return

OKX gives four direct trade-offs, and they're worth taking at face value.

**Narrow range.** More trades, faster turnover, potentially a higher percentage return on the capital you put in. The catch is that a price jump out of the band stops the bot entirely, and you're left holding whatever it bought.

**Wide range.** The bot survives big swings and stays alive in volatile conditions. It also trades less often, and your capital is spread across more levels, so ROI per dollar is slower.

**More grids.** Smaller gaps, frequent fills, thin profit per fill. Every fill pays a fee, so the profit per trade gets shaved by a fixed cost. Set 200 grids on a 3% range and you've built a fee-generating machine that occasionally nets you something.

**Fewer grids.** Bigger gaps, fewer trades, larger profit per fill. Costs stay low, but you may sit through long stretches with nothing happening.

**Bigger investment.** Funds every grid, increases order size, raises potential dollar profit. It also locks up more capital and increases your exposure when price moves against you.

**Smaller investment.** Caps risk and capital lock-up, at the cost of leaving some grids unfunded, which means missed trades.

OKX puts over-optimisation on its own risk list: too many grids or too narrow a range reduces efficiency and ties up funds. That's the official position, not a reviewer's opinion.

## Trailing grid: the part most guides skip

Trailing up and trailing down are optional, and they change where your stop-loss sits. This is the detail that catches people out.

**Trailing Up.** When price touches the upper limit, the bot cancels the lowest buy order at the bottom of the grid and places a new buy order at the original upper limit. Once that fills, a sell order goes one level above the original upper limit. The whole grid moves up, including both limits, and the size of the range stays the same.

OKX's example: a 25,000–30,000 USDT range with 5 grids, 1,000 USDT apart. If BTC/USDT breaks above 30,000, the 25,000 buy order is cancelled and a new order goes in at 31,000. The range is now 26,000–31,000 in effect.

Here's the part worth highlighting. If you set a **take-profit by price, it does not move with the grid**. It stays where you configured it. If you set a **stop-loss by price, it moves up in step with the grid**, shifting at the same interval as the lower limit. So trailing up raises your floor automatically, which is good, but it also raises your exit trigger, which can matter if the rally reverses hard.

**Trailing Down.** When price falls below the lower limit, the bot places a new buy order one level below the old lower limit, using **additional funds from your main account**. If there isn't enough, no order is placed and the grid stops extending. Once that buy fills, a sell order is raised to complete the pair, and the lower limit shifts down one level while the upper limit stays put. The range gets bigger.

OKX's example: 25,000–30,000 USDT with 5 grids at 1,000 apart. Price drops below 25,000, so a buy order goes in at 24,000, and the grid count becomes 6.

During trailing down, **both take-profit and stop-loss stay fixed** where you set them. Nothing follows the market downward. That's a meaningful asymmetry: trailing up tightens your stop, trailing down doesn't loosen it.

You can reserve funds at creation to cover trailing-down extensions. If you don't, the bot will try to pull from your main account balance when it needs to, and if that fails, the grid stops growing.

Both features carry obvious risks that OKX states directly. Trailing up leaves you holding expensive buys if the rally fails. Trailing down can leave you over-accumulated in a long bear market, buying all the way down with a stop-loss that never moved.

## Fees and locked capital

Bot trades pay the same fee as manual trades. OKX doesn't charge a separate bot fee, a subscription, or a monthly plan. There's no tier to upgrade to. If you see a "bot plan" being sold anywhere, it isn't an OKX product.

OKX doesn't publish a single minimum investment figure in these docs, and the maximum is normally whatever balance you hold in your trading account for the chosen currency. Start with an amount where the per-grid profit, after fees, is still worth having.

Capital is isolated the moment you create the bot. The funds leave your trading account and are used only for grid trading, which means they're not available for other positions. If you enable trailing down and haven't reserved extra funds, the bot will reach into your main balance; if that balance is short, the bot may terminate. Plan the total capital commitment, not just the initial investment.

👉 [See the current OKX spot grid setup and fund a bot](https://okx.com/join/CASH20)

## When it loses money

OKX's loss example is worth reading closely. Take a BTC range of 20,000–24,000 with 10 grids. Price chops inside the band, the bot buys near 20,000–21,000 and sells near 23,000–24,000, and small profits accumulate. That's the good case.

Now the same setup with BTC falling from 22,000 to 18,000. Every buy order below fills. Price never rebounds to the sell levels. The bot stops placing new orders because the market is outside the range, and you're holding a position with an unrealized loss. A stop-loss would have sold at a lower price and locked the loss in, which is better than watching it deepen but still a loss.

The structural version of this: if price falls below the lower limit and stays there, the bot stops working, and the crypto it holds keeps losing value. OKX recommends placing a stop-loss below the lower limit for exactly this reason.

Other ways it goes wrong:

- **Strong trends.** In a sharp uptrend or downtrend, many orders never fill, or filled buy orders accumulate losses.
- **Volatility storms.** Frequent fills during high volatility raise transaction costs and eat profits.
- **Sudden news.** A single event can push price straight through your range and leave the bot useless.
- **Suspension or delisting** of the underlying asset stops the bot automatically.
- **Partial execution.** In a multi-leg strategy like a grid, market movement may fill only one leg, leaving you with exposure you didn't plan for.

OKX states plainly that bot use doesn't guarantee returns and isn't investment advice. You own the parameters and the monitoring.

## Changing or stopping a running bot

You can edit the price range and grid quantity while the bot is live. That's the first option when price drifts outside your band and you still believe in the setup.

You can also withdraw grid profits to your trading account without stopping the bot, or replicate the whole configuration with one click if you want a second bot on another pair.

**Stopping.** OKX's current documentation says when you stop the bot, all pending orders are cancelled, and you then choose whether to sell the crypto at market price or keep it. Either way, the funds or the assets go back to your trading account. Some older versions of the help pages say the system sells at market automatically on stop. If you see that wording elsewhere, the current behaviour is the one with the choice.

If price leaves your range, you have three options: edit the range, terminate the strategy (cancels all orders and closes at current market price, assets returned to your account), or wait for price to come back.

The Bot Details page shows filled orders (total completed orders) and matched trades (a buy and a sell completed at a higher level). Matched trades are the ones that actually produced grid profit.

## FAQ

**Will the bot use my whole balance?**
No. You set the investment amount, and only that amount is allocated and isolated. The rest of your trading account stays untouched unless you enable trailing down without reserved funds.

**What happens when price leaves my range?**
The bot stops placing new orders. It keeps holding whatever it bought. You can widen the range, stop the bot, or wait for price to return.

**Can I change the range and grid count after it starts?**
Yes, both are editable while the bot runs.

**Do bot trades cost more in fees?**
No. Same fee schedule as manual orders, no additional bot charge. But more grids means more fills, and each fill pays that fee again.

**Are more grids always better?**
No. More grids means smaller profit per fill against a fixed per-trade cost. Past a point you're paying fees to trade with yourself.

**Does the take-profit move when trailing up is on?**
No. Take-profit by price stays fixed. Stop-loss by price moves up with the grid.

**Can I run this on any pair?**
OKX's docs don't publish a list of supported pairs, so check the pair selector in the bot screen for what's actually available to you.

## Which setup fits you

If you just want to test the mechanics, fund a small amount on a major pair, keep the range narrow enough that trades actually happen, and put a stop-loss below the lower limit before you start. The point of the first bot is to watch filled orders and matched trades diverge in real time.

If you have a genuine range view (you think BTC chops between two levels for the next month), go wider with fewer grids. You'll trade less, pay fewer fees, and survive a wobbly week without the bot shutting down.

If you expect a one-way move in either direction, don't start a grid. A spot grid in a trending market either leaves orders unfilled on the way up or accumulates bags on the way down, and neither outcome needs a bot to produce.

Two things to confirm before you fund anything: the total capital you're willing to lock up, including any trailing-down reserve, and where your stop-loss sits relative to the lower limit. Get those two numbers right and the rest is tuning.

👉 [Set up your first OKX spot grid bot with code CASH20](https://okx.com/join/CASH20)
