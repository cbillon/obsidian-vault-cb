---
link: https://konradreiche.com/blog/an-introduction-to-pid-controllers/
byline: Konrad Reiche
excerpt: >-
  I first encountered PID controllers while working with Donovan Baarda on a
  cache implementation for a GopherCon talk in 2024, where he proposed using one
  as a feedback mechanism to regulate eviction rate efficiently.

  PID controllers belong to a broader family of feedback control mechanisms that
  stabilize systems under changing conditions. With relatively simple rules,
  they can keep complex systems near a desired target even as load and capacity
  fluctuate, a property that can translate directly to backend systems like
  autoscalers, concurrency limiters, and caches. I reimplemented one in Go and
  applied it to a familiar, everyday example to build a clearer mental model.
  This post walks through that process.
slurped: 2026-06-21T10:22
title: An introduction to PID controllers
---

Apr 7, 2026

I first encountered PID controllers while working with [Donovan Baarda](https://github.com/dbaarda) on a cache implementation for a [GopherCon](https://speakerdeck.com/konradreiche/building-a-highly-concurrent-cache-in-go-a-hitchhikers-guide) talk in 2024, where he proposed using one as a feedback mechanism to regulate eviction rate efficiently.

PID controllers belong to a broader family of feedback control mechanisms that stabilize systems under changing conditions. With relatively simple rules, they can keep complex systems near a desired target even as load and capacity fluctuate, a property that can translate directly to backend systems like autoscalers, concurrency limiters, and caches. I reimplemented one in Go and applied it to a familiar, everyday example to build a clearer mental model. This post walks through that process.

## What is a PID Controller?

[Wikipedia](https://en.wikipedia.org/wiki/Proportional%E2%80%93integral%E2%80%93derivative_controller) offers us a starting definition:

> A proportional–integral–derivative controller (PID controller or three-term controller) is a feedback-based control loop mechanism commonly used to manage machines and processes that require continuous control and automatic adjustment.

While PID controllers originated in physical process control, they are used across a wide range of industrial applications where maintaining a precise, stable value is critical.

Examples include injection molding and plastics manufacturing, where material temperature must stay within tight tolerances, packaging lines that require consistent pressure or flow, and HVAC systems that balance energy efficiency with comfort. What they all have in common is that _some_ measurable quantity needs to track a target value.

## Your Oven as a Control System

For this post, we will use a household oven as our running example. Suppose you want to bake your first batch of macarons. Macarons are notoriously unforgiving. A deviation of just 5°F can be the difference between a perfect shell and a ruined one. Run too hot and the shells brown before the insides finish cooking. Run too cold and they fail to develop their characteristic ruffled feet.

You set the oven, it heats up, and for a moment everything seems fine. But the oven is constantly influenced by its surroundings. The ambient temperature of the kitchen affects how quickly it loses heat, and the controller has to decide when to start heating again, and by how much. This is exactly the kind of problem a PID controller is designed to solve.

The simplest approach would be to heat whenever the temperature falls below the target, and stop once it is reached. Let’s call this an **on-off controller**. To prevent rapid on/off cycling, we add a deadband: a tolerance zone around the target temperature within which the heater stays off. In Go, that looks like this:

```
func (c *onOffController) Update(target, current float64) float64 {
	if current < target-deadband {
		return maxHeaterPower
	}
	return 0.0
}
```

Now let’s see how this performs over time:

![Simple on-off controller with deadband](app://obsidian.md/blog/an-introduction-to-pid-controllers/on-off.svg)

First the temperature overshoots beyond the 300°F target, it then cycles around the target rather than settling near a fixed value. When it drops below the deadband threshold, the heater fires at full power, overshoots again, then slowly cools back down due to heat loss to the environment. It never settles at 300°F. The on-off controller cannot do better than this because it only knows one thing: whether the temperature is above or below the target.

## PID Controller

A PID controller knows more. It is made up of three terms, each capturing a different aspect of the error. In control theory, error does not mean something went wrong. It refers to the gap between where you want the system to be and where it is right now:

```
controlError := target - current
```

Each term responds to a different aspect of that error, weighted by its own gain parameters: proportional gain, integral gain and derivative gain. Another way to think about PID is that each term compensates for a different limitation of simple control: the proportional term improves responsiveness, the integral term removes persistent bias, and the derivative term anticipates change to reduce overshoot.

Tuning a PID controller is a matter of finding the right values for these three constants. The sum of all three terms produces our control signal:

```
controlSignal := pTerm + iTerm + dTerm
```

### Proportional Term

The proportional gain controls the response to the current error. Increasing it makes the system respond more quickly, but can cause overshoot and oscillations.

```
pTerm := proportionalGain * controlError
```

A good starting point is to run the controller on proportional gain alone. With the proportional gain set to 1, the improvement over the on-off controller is immediate:

![p-only controller](app://obsidian.md/blog/an-introduction-to-pid-controllers/p_only.svg)

The cycling disappears and the temperature rises smoothly without overshooting. However, it settles at 298°F rather than the target 300°F. This gap is called steady-state error, and it is a limitation of proportional control alone, because the proportional term requires a nonzero error to produce any output, so in this example it can never fully close the gap. Increasing the proportional gain can close the gap, but can push the system back toward oscillation.

### Integral Term

The proportional term only reacts to the error right now. It has no memory of how long the error has persisted. If the temperature has been sitting at 298°F for ten minutes, the proportional term treats that the same as if it just arrived there a second ago. The integral term accounts for this by accumulating error over time:

```
integral += controlError * step
iTerm := integralGain * integral
```

Each tick, the current error is added to a running sum. The longer the system sits below the target, the larger that sum grows, and the harder the integral term pushes. A persistent small error that the proportional term ignores will eventually accumulate enough to force a correction.

![pi controller](app://obsidian.md/blog/an-introduction-to-pid-controllers/pi.svg)

With an integral gain of 3, the steady-state error disappears and the temperature settles near 300°F. The initial overshoot still occurs, followed by small oscillations around the target. Higher values eliminate the offset faster but increase the risk of further overshoot or oscillation, since the accumulated sum does not reset when the error crosses zero.

One practical issue with the integral term is integral windup. If the controller output is saturated, for example when the heater is already at maximum power, the integral term can continue accumulating error. When the system finally recovers, this stored-up correction can cause significant overshoot. Real implementations often include anti-windup mechanisms to prevent this.

### Derivative Term

The proportional term reacts to how large the error is. The integral term reacts to how long it has persisted. The derivative term reacts to how fast it is changing. Consider the initial heat-up. The oven is climbing quickly from room temperature toward 300°F. The proportional term sees a large error and pushes hard, but it has no sense of how fast the temperature is already rising. Left unchecked, the system overshoots the target before it can correct. The derivative term acts as a brake: if the error is shrinking rapidly, it reduces the control signal preemptively, smoothing the approach.

The same logic applies in reverse when something causes a sudden drop. Opening the oven door mid-bake is a common example. The temperature falls sharply, the error grows fast, and the derivative term responds with more urgency than the proportional term alone would justify at that moment.

```
derivative := (controlError - previousControlError) / step
dTerm := derivativeGain * derivative
```

A higher derivative gain reduces overshoot and dampens oscillation, but it amplifies noise, since any rapid fluctuation in the measurement, or even discretization effects in a sampled system, can look like a large rate of change. In practice this means derivative control is often applied to a smoothed signal rather than the raw measurement. Not knowing any better, we set the derivative to 0.5:

![pid controller](app://obsidian.md/blog/an-introduction-to-pid-controllers/pid.svg)

We amplified the oscillation where the temperature swings between 298°F and 306°F. In practice, the derivative term is often applied to the measurement rather than the error, and frequently to a smoothed signal. This avoids derivative kick, where sudden changes in the target cause large spikes in the control signal, and reduces sensitivity to noise.

### Tuning

With three interdependent gain parameters, finding good values by intuition alone is difficult. Each parameter affects the others: increasing the proportional gain may require adjusting the integral gain to avoid instability, and so on. One of the most well-known is the [Ziegler-Nichols method](https://en.wikipedia.org/wiki/Ziegler%E2%80%93Nichols_method).

The idea is to deliberately push the controller into sustained oscillation. Start with integral and derivative gain set to 0, then increase the proportional gain until the temperature settles into sustained oscillations. This is called the ultimate gain, and the period of that cycle is called the ultimate period.

For our oven model, this occurs at a proportional gain of 2.01, producing the following behavior:

![Sustained oscillation at ultimate gain](app://obsidian.md/blog/an-introduction-to-pid-controllers/ultimate_gain.svg)

Finding the value can also be tricky. If you see oscillation but it’s shrinking, you need to increase the proportional gain parameter, if you see the oscillation growing, you need to lower the proportional gain.

The oscillation is clean and regular, with a period of two minutes. From these two values, the Ziegler-Nichols method derives the gain parameters through a fixed set of formulas, giving us:

|Term|Gain|
|---|---|
|Proportional|1.21|
|Integral|1.21|
|Derivative|0.301|

These values assume the standard form of the PID controller, where the proportional, integral, and derivative terms are summed independently. Applying these to the simulation:

![PID controller tuned with Ziegler-Nichols](app://obsidian.md/blog/an-introduction-to-pid-controllers/zn_pid.svg)

The result is an improvement. The controller reaches the setpoint quickly, though still overshoots and while it stays more closely to the setpoint now than before, some jitter around the target temperature is still visible. To improve this further and reduce, or even eliminate, the overshoot, take a look at the [demo code](https://github.com/konradreiche/pid/tree/69aa66ed3a6856c6cb3189d5ba71d1f484e6a5c7/example/oven), which also includes anti-windup clamping and an optional setpoint filter.

### When PI Is Better Than PID

One interesting detail emerged while experimenting with the tuned controller. Although the Ziegler–Nichols method suggested a full PID controller, the derivative term did not noticeably improve the behavior of this oven model. In fact, removing it produced slightly smoother regulation around the setpoint:

![PID controller tuned with Ziegler-Nichols and derivative gain set to 0](app://obsidian.md/blog/an-introduction-to-pid-controllers/zn_pi.svg)

The reason lies in the dynamics of the system. Our oven behaves like a simple first-order thermal system: applying heat raises the temperature, while heat loss to the surrounding air naturally damps the response. Because of this inherent damping, the derivative term provides little benefit and mostly reacts to small fluctuations in the measurement, which can introduce minor oscillations.

Derivative control is more useful in systems with inertia or momentum, where the controller needs to anticipate overshoot. In contrast, thermal systems slow down naturally as heating is reduced.

For that reason, many real temperature controllers use PI control rather than full PID. In this case, keeping the proportional and integral gains and setting the derivative gain to zero results in faster settling and more stable behavior around the setpoint.

### Go Library

If you want to use a PID controller in Go, you can check out the library I wrote [here](https://github.com/konradreiche/pid). At each step, it requires three inputs: a target, a measurement, and the time since the last update. Everything else follows from those.

```
const maxHeaterPower = 20

controller, err := pid.New(
    pid.WithZieglerNicholsMethod(2.01, 2),
    pid.WithDerivativeGain(0.0),
    pid.WithOutputLimit(0.0, maxHeaterPower),
)
if err != nil {
    log.Fatal(err)
}

const target = 1.0
timeStep := 100 * time.Millisecond

var measurement float64
for i := range 10 {
    control := controller.Update(target, measurement, timeStep)
    measurement += 0.25 * control
    fmt.Printf("step=%d control=%.2f measurement=%.2f\n", i, control, measurement)
}
```

The time step determines how often the controller updates. The output limit clamps the control signal to a usable range, in the oven’s case that maps to heater’s maximum power.

How you apply the control signal depends entirely on your system. You don’t feed it back into the measurement directly. Instead it drives an actuator, for example the heater, a valve, a throttle, that nudges the system toward the target.

## When PID Works Well

A PID controller is not a universal solution. It works best when a few conditions hold.

1. The system responds roughly proportionally to the control signal. An oven that heats faster when you supply more power is a good fit. A system where small inputs have no effect until a threshold is crossed, then change abruptly, is not.
    
2. The dynamics are not dominated by large time delays. If there is a long lag between applying a correction and seeing it reflected in the measurement, the controller is flying blind. It will keep correcting based on stale information and overshoot.
    
3. The measurement is clean and timely. The derivative term in particular is sensitive to noise, since any rapid fluctuation looks like a fast-changing error. Noisy measurements either require filtering or a reduced derivative gain.
    
4. The system has a single measurable output. When one output depends on several inputs, or changing one variable affects others, the error signal becomes ambiguous. A PID controller has no way to reason about coupled dynamics. In those cases the error needs to be filtered or transformed before use, or a different control strategy is more appropriate.
    
5. The target is stable and achievable. A PID controller can track a slowly moving target, but it is designed around the assumption that the setpoint is a fixed goal.
    

When these conditions hold, PID is an effective tool for a small amount of code. When they do not, the failure modes tend to be oscillation, instability, or persistent steady-state error that no amount of tuning will fully resolve.

## Applications in Backend Systems

Physical and software systems share more than it might seem. Both exhibit delays, inertia, saturation, and noise. A queue that takes time to drain, a service whose latency climbs gradually under load, a worker pool that cannot scale instantaneously: these behave like physical systems, and can be controlled like them.

In software, what control theory calls the plant is typically something like queue depth, latency, CPU load, or throughput. Some areas where a PID controller could be a natural fit:

- **Autoscaling**: scale the number of instances up or down based on the error between target and observed latency or queue depth, rather than simple threshold rules.
- **Concurrency limiting**: adjust the number of in-flight requests to keep latency near a target, similar to how TCP congestion control uses AIMD to regulate throughput.
- **Rate limiting and traffic shaping**: modulate request rates to hold a downstream system near its optimal operating point rather than hammering it until it fails.
- **Cache eviction**: dynamically adjust the eviction threshold to maintain a target hit rate, rather than maintaining a costly ordered structure under contention.

Of these, cache eviction is particularly interesting in practice. A heap-based priority queue does not scale well under high concurrent load, which makes a PID-based approach worth exploring. This is the specific problem we will look at in a follow-up post.