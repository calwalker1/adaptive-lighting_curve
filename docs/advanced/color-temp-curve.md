---
icon: lucide/thermometer
---

# Custom Color Temperature Curve

By default, Adaptive Lighting reaches `min_color_temp` (fully warm) exactly
at sunset. In winter, when sunset happens as early as 5:30 PM, this can make
your lights turn fully warm well before you're ready to wind down for the
night.

The `color_temp_mode: custom` option lets you decouple "fully warm" from
"sunset": lights reach an intermediate `sunset_color_temp` at sunset and
*hold* there — no further change — until a time you choose, at which point
they switch over to whatever Adaptive Lighting would normally be showing.

## How It Works

With `color_temp_mode: custom`:

- **At sunset**, color temperature reaches `sunset_color_temp` (instead of `min_color_temp`).
- **From sunset until the hold ends**, it stays flat at `sunset_color_temp` — it does not gradually ramp during this window.
- **Once the hold ends** (`sunset_color_temp_delay`/`sunset_color_temp_time`), it switches to whatever the normal `default` curve would show right now (`min_color_temp`, or further along if `transition_until_sleep` is enabled).
- **Mornings are unaffected** — the sunrise ramp still runs between `min_color_temp` and `max_color_temp` as usual.

The hold end point is set with one of:

| Option | Description |
|--------|-------------|
| `sunset_color_temp_delay` | Duration in seconds after sunset. Takes priority if set to a value > 0. |
| `sunset_color_temp_time` | A fixed clock time (`HH:MM:SS`). Used only if `sunset_color_temp_delay` is `0`. |

If neither is set, the hold ends immediately at sunset (equivalent to setting `min_color_temp` as `sunset_color_temp`).

The switch-over itself isn't animated by this option — Home Assistant's
own periodic adaptation `transition` setting (checked roughly every
`interval` seconds) is what makes the jump from `sunset_color_temp` to the
normal value look smooth on the light, the same way any other Adaptive
Lighting color change does.

!!! tip "Choosing `sunset_color_temp`"
    Pick a value roughly midway between `min_color_temp` and
    `max_color_temp`, not right up near `max_color_temp`. Since the value
    is held flat for the whole window, how close it is to your endpoints
    only affects how big a jump happens at the switch-over — it doesn't
    make the transition itself more gradual.

## Example: Hold Until 9 PM

Matches the motivating case: an early winter sunset (5:30 PM) that only
partially warms the lights, switching to fully warm at 9 PM.

```yaml
adaptive_lighting:
  - name: "Living Room"
    lights:
      - light.living_room
    min_color_temp: 2000
    max_color_temp: 5500
    color_temp_mode: custom
    sunset_color_temp: 3500    # hold at a "medium warm" 3500K after sunset
    sunset_color_temp_time: "21:00:00"  # switch to 2000K at 9 PM
```

## Example: Fixed Delay Instead of a Clock Time

```yaml
adaptive_lighting:
  - name: "Living Room"
    lights:
      - light.living_room
    color_temp_mode: custom
    sunset_color_temp: 3500
    sunset_color_temp_delay: 5400  # switch over 90 minutes after sunset
```

See the [Configuration](../configuration.md) page for the full options table.
