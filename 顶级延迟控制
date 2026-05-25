local abs = math.abs
local floor = math.floor
local max = math.max
local GrainMS = 1
local SpinGuard = 0.5
local Limit = 0.35
local Fast = 0
local Jitter = 0
local LastExit = 0
local function Clamp(v, lo, hi)
    if v < lo then 
        return lo 
    end
    if v > hi then 
        return hi 
    end
    return v
end
local function Sort(buf, n)
    for i = 2, n do
        local val = buf[i]
        local j = i - 1
        while j > 0 and buf[j] > val do
            buf[j + 1] = buf[j]
            j = j - 1
        end
        buf[j + 1] = val
    end
end
local function Calibrate()
    for _ = 1, 4 do 
        Sleep(1) 
    end
    local N = 20
    local buf = {}
    local cap = 18
    local cnt = 0
    for _ = 1, N do
        local t0 = GetRunningTime()
            Sleep(1)
        local d = GetRunningTime() - t0
        if d < cap then
            cnt = cnt + 1
            buf[cnt] = d
        end
    end
    if cnt < 6 then
        cnt = 6
        for i = 1, 6 do 
            buf[i] = 1 
        end
    end
    Sort(buf, cnt)
    local lo = 3
    local hi = cnt - 2
    local sum = 0
    for i = lo, hi do 
        sum = sum + buf[i] 
    end
    local trim = sum / (hi - lo + 1)
    local mid = floor(cnt / 2)
    local med = (buf[mid] + buf[mid + 1]) / 2
    GrainMS = max(med * 0.5 + trim * 0.5, 0.5)
    SpinGuard = GrainMS * 0.8 + 0.15
end
local function Propagate(err, exitNow)
    local gap = exitNow - LastExit
    LastExit = exitNow
    local alpha = (gap > 0 and gap < 3) and 0.1 or 0.06
    Fast = Clamp(Fast * (1 - alpha) + err * alpha, -Limit, Limit)
    Jitter = Jitter * 0.88 + abs(err) * 0.12
end
local function Comfrey(ms)
    if ms <= 0 then 
        return 
    end
    local t1 = GetRunningTime()
    local Ideal = t1 + ms
    local Target = Ideal - Clamp(Fast, -Limit, Limit)
    local SpinWin = SpinGuard + Jitter * 0.5
    if ms < 2 then
        local SpinStart = Target - SpinWin
        if SpinStart > t1 then
            repeat until GetRunningTime() >= SpinStart
        end
        repeat until GetRunningTime() >= Target
        local exitNow = GetRunningTime()
        Propagate(exitNow - Ideal, exitNow)
        return
    end
    local SleepThresh = SpinWin + GrainMS * 2
    while true do
        local Now = GetRunningTime()
        local Remain = Target - Now
        if Remain <= 0 then
            Propagate(Now - Ideal, Now)
            return
        end
        if Remain > SleepThresh then
            local amt = floor(Remain - SleepThresh)
            if amt >= 1 then Sleep(amt) 
            end
        else
            local SpinStart = Target - SpinWin
            repeat until GetRunningTime() >= SpinStart
            repeat until GetRunningTime() >= Target
            local exitNow = GetRunningTime()
            Propagate(exitNow - Ideal, exitNow)
            return
        end
    end
end
do
    Calibrate()
    LastExit = GetRunningTime()
end