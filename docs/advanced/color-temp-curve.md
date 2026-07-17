---
icon: lucide/thermometer
---

# Custom Color Temperature Curve

By default, Adaptive Lighting reaches `min_color_temp` (fully warm) exactly
at sunset. In winter, when sunset happens as early as 5:30 PM, this can make
your lights turn fully warm well before you're ready to wind down for the
night.

`color_temp_mode` gives you two alternative curves that decouple "fully
warm" from "sunset": lights reach an intermediate `sunset_color_temp` at
sunset and hold there until a time you choose, at which point they switch
over to whatever Adaptive Lighting would normally be showing — either
instantly (`snap`) or gradually (`fade`).

## The Three Modes

| Mode | Behavior |
|------|----------|
| `default` | Normal Adaptive Lighting curve — reaches `min_color_temp` exactly at sunset. |
| `snap` | Reaches `sunset_color_temp` at sunset, holds flat, then *jumps* straight to the normal value once the hold ends. |
| `fade` | Reaches `sunset_color_temp` at sunset, holds flat, then *ramps gradually* from `sunset_color_temp` to the normal value over the hold window. |

For both `fade` and `snap`:

- **At sunset**, color temperature reaches `sunset_color_temp` (instead of `min_color_temp`).
- **The hold ends** at `sunset_color_temp_delay`/`sunset_color_temp_time`.
- **Once the hold ends**, the light shows whatever the normal `default` curve would show right now (`min_color_temp`, or further along if `transition_until_sleep` is enabled).
- **Mornings are unaffected** — the sunrise ramp still runs between `min_color_temp` and `max_color_temp` as usual.

The hold end point is set with one of:

| Option | Description |
|--------|-------------|
| `sunset_color_temp_delay` | Duration in seconds after sunset. Takes priority if set to a value > 0. |
| `sunset_color_temp_time` | A fixed clock time (`HH:MM:SS`). Used only if `sunset_color_temp_delay` is `0`. |

If neither is set, the hold ends immediately at sunset (equivalent to setting `min_color_temp` as `sunset_color_temp`).

### `snap`: instant switch-over

Nothing changes on the light between sunset and the hold end — it's held
flat at `sunset_color_temp`. At the hold end, the target value jumps to the
normal curve's value all at once. Home Assistant's own periodic adaptation
`transition` setting (checked roughly every `interval` seconds) is what
makes that jump look smooth on the light, the same way any other Adaptive
Lighting color change does — this option doesn't animate the jump itself.

### `fade`: gradual switch-over

Also holds flat until the hold end, but instead of jumping, the value
ramps linearly from `sunset_color_temp` to the normal curve's value across
the *whole* hold window (sunset → hold end). Choose this if you'd rather
the light drift gradually over the last stretch before the hold ends,
instead of changing all at once.

!!! tip "Choosing `sunset_color_temp`"
    Pick a value roughly midway between `min_color_temp` and
    `max_color_temp`, not right up near `max_color_temp`. With `snap`, how
    close it is to your endpoints only affects how big the jump is at the
    hold end. With `fade`, it affects how far the light has to travel
    during the fade — the closer to `max_color_temp` it is, the more
    change gets compressed into that final stretch.

## Example: `snap` at 9 PM

Matches the motivating case: an early winter sunset (5:30 PM) that only
partially warms the lights, snapping to fully warm at 9 PM.

```yaml
adaptive_lighting:
  - name: "Living Room"
    lights:
      - light.living_room
    min_color_temp: 2000
    max_color_temp: 5500
    color_temp_mode: snap
    sunset_color_temp: 3500    # hold at a "medium warm" 3500K after sunset
    sunset_color_temp_time: "21:00:00"  # switch to 2000K at 9 PM
```

## Example: `fade` with a Fixed Delay

```yaml
adaptive_lighting:
  - name: "Living Room"
    lights:
      - light.living_room
    color_temp_mode: fade
    sunset_color_temp: 3500
    sunset_color_temp_delay: 5400  # fade to normal over 90 minutes after sunset
```

See the [Configuration](../configuration.md) page for the full options table.
