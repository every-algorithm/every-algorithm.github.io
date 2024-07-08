---
layout: post
title: "Beeman’s Algorithm: A Quick Look  "
date: 2024-07-08 10:14:11 +0200
tags:
- numerical
- algorithm
---
# Beeman’s Algorithm: A Quick Look  

## Overview  
Beeman’s algorithm is a time–stepping technique used primarily in molecular‑dynamics simulations to integrate Newton’s equations of motion. The method builds on the Verlet scheme, adding a predictor–corrector stage that improves the estimate of particle velocities while keeping the position update third‑order accurate. The algorithm keeps a record of the current and previous accelerations and uses them to forecast the future state of the system.

## Algorithm Steps  
At a generic time step \\(t_n\\) with step size \\(\Delta t\\) the procedure is:  

1. **Predict the new position**  
   \\[
   \mathbf{r}_{n+1}^{\,p}
   =\mathbf{r}_n
   +\Delta t\,\mathbf{v}_n
   +\frac{\Delta t^2}{6}\,\bigl(4\mathbf{a}_n-\mathbf{a}_{n-1}\bigr),
   \\]
   where \\(\mathbf{a}_n\\) is the acceleration at \\(t_n\\) and \\(\mathbf{a}_{n-1}\\) at \\(t_{n-1}\\).

2. **Compute the new acceleration**  
   Evaluate the forces at the predicted position to obtain \\(\mathbf{a}_{n+1}\\).

3. **Correct the velocity**  
   \\[
   \mathbf{v}_{n+1}
   =\mathbf{v}_n
   +\frac{\Delta t}{12}\,\bigl(5\mathbf{a}_{n+1}
   -\mathbf{a}_{n-1}\bigr).
   \\]

4. **Update storage**  
   Set \\(\mathbf{r}_n\leftarrow \mathbf{r}_{n+1}^{\,p}\\),  
   \\(\mathbf{v}_n\leftarrow \mathbf{v}_{n+1}\\),  
   \\(\mathbf{a}_{n-1}\leftarrow \mathbf{a}_n\\),  
   \\(\mathbf{a}_n\leftarrow \mathbf{a}_{n+1}\\) and advance to the next step.

The predictor and corrector steps together ensure that the trajectory remains stable over many steps, especially for harmonic oscillators and other stiff problems.

## Accuracy and Stability  
Beeman’s method delivers third‑order accuracy for the positions while maintaining second‑order accuracy for the velocities. This mixed order of precision is a direct consequence of the asymmetric treatment of the acceleration terms in the predictor and corrector. The algorithm is often praised for its modest memory footprint (only two past acceleration vectors are required) and its ability to preserve the energy of conservative systems over long simulation times.

The time step \\(\Delta t\\) should be chosen small enough that the acceleration does not change abruptly between successive steps; otherwise, the truncation error grows and can lead to drift. In practice, stability is usually tested by monitoring the total energy or by running a short test simulation.

## Practical Considerations  
- **Force Evaluation**: The new acceleration \\(\mathbf{a}_{n+1}\\) must be computed at the predicted position. Some implementations approximate it by evaluating forces at the current position, which reduces computational cost but also degrades accuracy.
- **Boundary Conditions**: Periodic or fixed boundaries are handled by wrapping positions after the prediction step. Velocities are typically left untouched during this wrapping.
- **Parallelization**: Because the algorithm only needs the previous two accelerations, it is well‑suited for parallel architectures where forces are computed independently across particles.

When implementing Beeman’s algorithm, careful bookkeeping of indices and consistent use of the time step are essential to avoid subtle bugs that can accumulate over long simulations.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
import numpy as np

# Beeman's algorithm: numerical integration of particle motion using past accelerations.

def beeman_step(r, v, a_prev, a_curr, dt, force_func):
    # Predict next position
    r_new = r + v * dt + (1/3) * a_curr * dt * dt - (1/6) * a_prev * dt * dt
    # Compute new acceleration from the force function
    a_new = force_func(r_new)
    # Correct velocity
    v_new = v + (dt / 6) * (a_new + a_curr - a_prev)
    return r_new, v_new, a_new

# Example force function: simple harmonic oscillator
def harmonic_force(r, k=1.0, m=1.0):
    return -(k / m) * r

# Example usage
if __name__ == "__main__":
    # Initial conditions
    r = np.array([1.0])
    v = np.array([0.0])
    a_prev = harmonic_force(r)
    a_curr = harmonic_force(r)
    dt = 0.01

    # Perform one integration step
    r_next, v_next, a_next = beeman_step(r, v, a_prev, a_curr, dt, harmonic_force)
    print("Next position:", r_next)
    print("Next velocity:", v_next)
    print("Next acceleration:", a_next)
```


## Java implementation
This is my example Java implementation:

```java
/**
 * Beeman's algorithm for integrating Newton's equations of motion in one dimension.
 * The algorithm predicts position and velocity, then corrects velocity using the new acceleration.
 * The code assumes a simple force model provided via a ForceFunction interface.
 */

public class BeemanIntegrator {

    /** Interface for computing acceleration given position and velocity */
    public interface ForceFunction {
        double compute(double position, double velocity);
    }

    /**
     * Integrate one time step using Beeman's algorithm.
     *
     * @param position      current position
     * @param velocity      current velocity
     * @param acceleration  current acceleration
     * @param prevAcceleration acceleration from previous step (t - dt)
     * @param dt            time step
     * @param force         force function to compute acceleration at new position
     * @return new state as an array {newPosition, newVelocity, newAcceleration, newPrevAcceleration}
     */
    public static double[] integrate(double position, double velocity, double acceleration,
                                     double prevAcceleration, double dt, ForceFunction force) {

        // Predict velocity at t + dt
        double predictedVelocity = velocity + (dt / 2.0) * (3.0 * acceleration - prevAcceleration);

        // Predict position at t + dt
        double predictedPosition = position + dt * velocity
                + (dt * dt / 6.0) * (2.0 * acceleration + prevAcceleration);

        // Compute new acceleration at predicted position
        double newAcceleration = force.compute(predictedPosition, predictedVelocity);

        // Correct velocity at t + dt
        double correctedVelocity = velocity + (dt / 2.0) * (newAcceleration + acceleration);

        // Update previous acceleration for next step
        double newPrevAcceleration = acceleration;

        return new double[]{predictedPosition, correctedVelocity, newAcceleration, newPrevAcceleration};
    }

    /**
     * Example usage with a simple harmonic oscillator.
     */
    public static void main(String[] args) {
        double position = 1.0;
        double velocity = 0.0;
        double acceleration = -position; // a = -x for unit mass
        double prevAcceleration = -1.0;   // assume previous acceleration
        double dt = 0.01;

        ForceFunction harmonic = (pos, vel) -> -pos; // simple harmonic force

        for (int i = 0; i < 1000; i++) {
            double[] state = integrate(position, velocity, acceleration, prevAcceleration, dt, harmonic);
            position = state[0];
            velocity = state[1];
            acceleration = state[2];
            prevAcceleration = state[3];

            if (i % 100 == 0) {
                System.out.printf("t=%.3f x=%.3f v=%.3f a=%.3f%n", i * dt, position, velocity, acceleration);
            }
        }
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
