//@version=5
indicator("Liquidity Sweep Pro", overlay=true, max_lines_count=500, max_labels_count=500)

len = input.int(5, "Swing length", minval=1)
maxLvls = input.int(10, "Levels to track", minval=1, maxval=50)
maxAge = input.int(100, "Max level age", minval=5)
useVol = input.bool(true, "Volume filter")
volLen = input.int(20, "Volume MA length")
volMult = input.float(1.5, "Volume multiplier", step=0.1)
usePD = input.bool(true, "Prev day high/low")
usePW = input.bool(true, "Prev week high/low")
fwdBars = input.int(10, "Stats: bars to check")
tgtPct = input.float(1.0, "Stats: reversal %", step=0.1)
showTbl = input.bool(true, "Show table")
drawLines = input.bool(true, "Draw lines")

volAvg = ta.sma(volume, volLen)
volOk = not useVol or na(volume) or na(volAvg) or volume > volAvg * volMult

ph = ta.pivothigh(high, len, len)
pl = ta.pivotlow(low, len, len)

var hiPrice = array.new_float()
var hiBar = array.new_int()
var loPrice = array.new_float()
var loBar = array.new_int()

if not na(ph)
    array.push(hiPrice, ph)
    array.push(hiBar, bar_index - len)
    if array.size(hiPrice) > maxLvls
        array.shift(hiPrice)
        array.shift(hiBar)

if not na(pl)
    array.push(loPrice, pl)
    array.push(loBar, bar_index - len)
    if array.size(loPrice) > maxLvls
        array.shift(loPrice)
        array.shift(loBar)

sweepHigh = false
sweepLow = false

// loop from the end bc we remove stuff
if array.size(hiPrice) > 0
    for i = array.size(hiPrice) - 1 to 0
        lvl = array.get(hiPrice, i)
        b = array.get(hiBar, i)
        remove = false
        if bar_index - b > maxAge
            remove := true
        else if high > lvl and close < lvl and volOk
            sweepHigh := true
            remove := true
            if drawLines
                line.new(b, lvl, bar_index, lvl, color=color.red, style=line.style_dashed)
        else if close > lvl
            remove := true
        if remove
            array.remove(hiPrice, i)
            array.remove(hiBar, i)

if array.size(loPrice) > 0
    for i = array.size(loPrice) - 1 to 0
        lvl = array.get(loPrice, i)
        b = array.get(loBar, i)
        remove = false
        if bar_index - b > maxAge
            remove := true
        else if low < lvl and close > lvl and volOk
            sweepLow := true
            remove := true
            if drawLines
                line.new(b, lvl, bar_index, lvl, color=color.green, style=line.style_dashed)
        else if close < lvl
            remove := true
        if remove
            array.remove(loPrice, i)
            array.remove(loBar, i)

// prev day / week levels
pdH = request.security(syminfo.tickerid, "D", high[1], lookahead=barmerge.lookahead_on)
pdL = request.security(syminfo.tickerid, "D", low[1], lookahead=barmerge.lookahead_on)
pwH = request.security(syminfo.tickerid, "W", high[1], lookahead=barmerge.lookahead_on)
pwL = request.security(syminfo.tickerid, "W", low[1], lookahead=barmerge.lookahead_on)

var pdHUsed = false
var pdLUsed = false
var pwHUsed = false
var pwLUsed = false

if ta.change(time("D")) != 0
    pdHUsed := false
    pdLUsed := false
if ta.change(time("W")) != 0
    pwHUsed := false
    pwLUsed := false

pdhSweep = usePD and not pdHUsed and high > pdH and close < pdH
pdlSweep = usePD and not pdLUsed and low < pdL and close > pdL
pwhSweep = usePW and not pwHUsed and high > pwH and close < pwH
pwlSweep = usePW and not pwLUsed and low < pwL and close > pwL

// only one sweep per level per day/week
if pdhSweep or close > pdH
    pdHUsed := true
if pdlSweep or close < pdL
    pdLUsed := true
if pwhSweep or close > pwH
    pwHUsed := true
if pwlSweep or close < pwL
    pwLUsed := true

if pdhSweep
    label.new(bar_index, high, "PDH", style=label.style_label_down, color=color.orange, textcolor=color.black, size=size.tiny)
if pdlSweep
    label.new(bar_index, low, "PDL", style=label.style_label_up, color=color.orange, textcolor=color.black, size=size.tiny)
if pwhSweep
    label.new(bar_index, high, "PWH", style=label.style_label_down, color=color.purple, textcolor=color.white, size=size.tiny)
if pwlSweep
    label.new(bar_index, low, "PWL", style=label.style_label_up, color=color.purple, textcolor=color.white, size=size.tiny)

plot(usePD ? pdH : na, "PDH", color.new(color.orange, 40), style=plot.style_stepline)
plot(usePD ? pdL : na, "PDL", color.new(color.orange, 40), style=plot.style_stepline)
plot(usePW ? pwH : na, "PWH", color.new(color.purple, 40), style=plot.style_stepline)
plot(usePW ? pwL : na, "PWL", color.new(color.purple, 40), style=plot.style_stepline)

bear = sweepHigh or pdhSweep or pwhSweep
bull = sweepLow or pdlSweep or pwlSweep

// stats - did price actually reverse after the sweep
var pStart = array.new_int()
var pPrice = array.new_float()
var pDir = array.new_int()
var total = 0
var wins = 0

if array.size(pStart) > 0
    for i = array.size(pStart) - 1 to 0
        pr = array.get(pPrice, i)
        hit = array.get(pDir, i) == -1 ? low <= pr * (1 - tgtPct / 100) : high >= pr * (1 + tgtPct / 100)
        if hit or bar_index - array.get(pStart, i) >= fwdBars
            total += 1
            if hit
                wins += 1
            array.remove(pStart, i)
            array.remove(pPrice, i)
            array.remove(pDir, i)

if bear
    array.push(pStart, bar_index)
    array.push(pPrice, close)
    array.push(pDir, -1)
if bull
    array.push(pStart, bar_index)
    array.push(pPrice, close)
    array.push(pDir, 1)

cell(t, row, txt, val) =>
    table.cell(t, 0, row, txt, text_color=color.white, text_size=size.small)
    table.cell(t, 1, row, val, text_color=color.white, text_size=size.small)

var tbl = table.new(position.top_right, 2, 3, bgcolor=color.new(color.black, 20))
if showTbl and barstate.islast
    wr = total > 0 ? str.tostring(wins * 100.0 / total, "#.#") + "%" : "-"
    cell(tbl, 0, "Sweeps", str.tostring(total))
    cell(tbl, 1, "Reversed " + str.tostring(tgtPct) + "% in " + str.tostring(fwdBars) + " bars", wr)
    cell(tbl, 2, "Levels tracked", str.tostring(array.size(hiPrice) + array.size(loPrice)))

plotshape(bear, "Sweep high", shape.triangledown, location.abovebar, color.red, size=size.small)
plotshape(bull, "Sweep low", shape.triangleup, location.belowbar, color.green, size=size.small)
bgcolor(bear ? color.new(color.red, 90) : bull ? color.new(color.green, 90) : na)

alertcondition(bear, "Sweep high", "Sweep above highs on {{ticker}} {{interval}}")
alertcondition(bull, "Sweep low", "Sweep below lows on {{ticker}} {{interval}}")
alertcondition(pdhSweep or pdlSweep or pwhSweep or pwlSweep, "HTF sweep", "Prev day/week level swept on {{ticker}}")
