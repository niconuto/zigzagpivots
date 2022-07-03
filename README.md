//@version=3

study("zigzag Pivots", overlay=true)


trackprice = input(title='Track Price', type=bool, defval=false)
length1 = input(title='Support / Resistance length:', type=integer, defval=10)
top1 = valuewhen(high >= highest(high, length1), high, 0)
bot1 = valuewhen(low <= lowest(low, length1), low, 0)
plot(top1, color=top1 != top1[1] ? na : black, linewidth=1, offset=0, trackprice=trackprice, transp=0)
plot(bot1, color=bot1 != bot1[1] ? na : green, linewidth=1, offset=0, trackprice=trackprice, transp=0)
barlength = input(title='Support1 / Resistance length1:', type=integer, defval=10)

length = input(title="Length", type=integer, defval=14, minval=1)
src = input(title="Source", type=source, defval=close)

factor = 0.0
slope = 0.0

for i = 1 to length
    factor := 1 + 2 * (i - 1)
    slope := slope + (src[i - 1]) * (length - factor) / 2

shmma = sma(src, length) + (6 * slope) / ((length + 1) * length)

newtop = shmma >= highest(shmma, barlength)
newbot = shmma <= lowest(shmma, barlength)
top = valuewhen(newtop, shmma, 0)
bot = valuewhen(newbot, shmma, 0)
plot(top, color=top != top[1] ? na : red, linewidth=1, offset=0, transp=0)
plot(bot, color=bot != bot[1] ? na : lime, linewidth=1, offset=0, transp=0)
//
mode = input('ATR', options=["ATR", "Traditional"])
modeValue = input(14.000, type=float)
bricksize = na
if mode == "ATR"
    bricksize := atr(round(modeValue))
if mode == "Traditional"
    bricksize := modeValue

showOverlay = input(false)

ropen = na
propen = nz(ropen[1])
ropen := close > propen+bricksize or high > propen+bricksize ? propen+bricksize : close < propen-bricksize or low < propen-bricksize ? propen-bricksize : propen

rclose = na
rclose := ropen > propen ? ropen-bricksize : ropen < propen ? ropen+bricksize : nz(rclose[1])
direction = na
direction := ropen > propen ? 1 : ropen < propen ? -1 : nz(direction[1])

rc = direction == 1 ? green : direction == -1 ? maroon : na

p00 = plot(not showOverlay ? na : ropen, style=cross, color=rc, linewidth=3)
p01 = plot(not showOverlay ? na : rclose, style=circles, color=gray, linewidth=2)

fill(p00, p01, color=gray, transp=75)

//  ||-->   ZigZag:
extreme = na
prev_extreme = change(direction) != 0 ? ropen : fixnan(extreme[1])
extreme := direction == 1 and high >= prev_extreme ? high : direction == -1 and low <= prev_extreme ? low : prev_extreme
plot(change(direction) != 0 ? extreme[1] : na)
//
t=change(direction) != 0 ? extreme[1] : na
showsig = input(title='Showsig', type=bool, defval=true)
f=crossover(t,bot)
q=crossunder(t,top)
plotshape(showsig and f, title='', style=shape.cross, location=location.abovebar, color=red, text='H', offset=-2,transp=0)
plotshape(showsig and q, title='', style=shape.cross, location=location.belowbar, color=lime, text='L', offset=-2,transp=0)
