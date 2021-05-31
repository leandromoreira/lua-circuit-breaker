# lua-circuit-breaker
![License](https://img.shields.io/badge/license-MIT-green.svg)

## Overview

The function of this library is to wrap functions with circuit breaker functionality. It is inspired by [moonbreaker](https://github.com/Invizory/moonbreaker) library.

## Sample Usage

```lua
--Import Circuit breaker factory.
local circuit_breaker_lib = require "lua-circuit-breaker.factory"

--Create a new instance of the circuit breaker factory. Always set version=0. This is used to flush the circuit breakers when the configuration is changed.
local circuit_breakers = circuit_breaker_lib:new({version = 0})

-- Get a circuit breaker instance from factory. Returns a new instance only if not already created .
local cb, err = circuit_breakers:get_circuit_breaker( 
    level, -- Level is used to flush all CBs at a certain level (Golbal / Service / Route) when cb configuration is changed at that level.                                                      
    name, -- Name of circuit breaker. This should be unique.
    {  
        window_time = 10, -- Time window in seconds after which the state of the cb is reset
        min_calls_in_window= 20, -- The minimum number of requests in a window that go through the cb after which the breaking strategy is applied
        failure_percent_threshold= 51, -- Failure % threshold after which the cb opens from closed or half open state
        wait_duration_in_open_state= 15, -- Time in seconds for which the cb remains in open state before transitioning to half open state
        wait_duration_in_half_open_state= 120, -- Time in seconds for which the cb remains in half open state
        half_open_max_calls_in_window= 10, -- Maximum calls in half open state after which **too_many_requests** error is returned.
        half_open_min_calls_in_window= 5, -- Minimum calls in half open state after which the calculation to open/close the circuit is done in half open state.
        version = 1, -- Version is used to flush the cbs if the configuration is changed.
        notify = function(state) -- Print a log when the state of cb changes.
            print(string.format("Breaker %s state changed to: %s", state._state))
        end
    }
)
-- Check state of cb. This function returns an error if the state is open or half_open_max_calls_in_window is breached.
local _, err_cb = cb:_before() 
if err_cb then
    return false, "Circuit breaker open error"
end
 
-- Make the http call for which circuit breaking is required.
local res, err_http = makeHttpCall(options)

-- Update the state of the cb based on successfull / failure response.
local ok = res and res.status and res.status < 500
cb:_after(cb._generation, ok) -- generation is used to update the counter in the correct time window.
```
