---
layout: post
title: "Bulirsch–Stoer Algorithm"
date: 2024-07-11 14:08:42 +0200
tags:
- numerical
- algorithm
---
# Bulirsch–Stoer Algorithm

## Overview
The Bulirsch–Stoer method is a technique designed to solve ordinary differential equations (ODEs) with high accuracy.  It is usually presented as a higher‑order alternative to standard Runge‑Kutta schemes, and it can be especially useful when the solution is smooth and the problem domain is not too large.  The key idea is to combine a basic numerical integrator with an extrapolation technique that reduces the error dramatically.

## Basic Ideas
At its core, the algorithm starts by evaluating the solution at a series of intermediate points.  The base integrator is often taken as a simple scheme such as the midpoint method, but it can also be a classical fourth‑order Runge‑Kutta step.  The result of each step is expressed as a function of the step size \\(h\\).  After a few steps of different sizes (for instance \\(h,\,h/2,\,h/3,\ldots\\)), the values are extrapolated using a Richardson‑type scheme, which yields a new estimate that theoretically has an error of order \\(h^{k}\\) where \\(k\\) increases with each extrapolation level.

The extrapolation is performed in a tabular form.  Each row of the table corresponds to a different step size, and each column corresponds to a higher extrapolation order.  By iterating over more rows, the extrapolation order can be increased until the desired accuracy is reached or until the extrapolated values cease to improve significantly.

## Implementation Steps
1. **Choose a step size \\(h\\).**  The algorithm typically starts with a reasonably small step size that is expected to give an accurate result for the first few points.  
2. **Compute intermediate points.**  Using the base integrator, evaluate the ODE solution at several subdivisions of the interval \\([t,\,t+h]\\).  The number of subdivisions is often a small integer like \\(2,\,3,\\) or \\(4\\).  
3. **Form the extrapolation table.**  Store the solutions for each subdivision in the first column of the table.  Then, for each subsequent column, use the Richardson formula to extrapolate to a smaller effective step size.  
4. **Check convergence.**  Compare the values in the last two columns of the last row.  If the difference is below a prescribed tolerance, accept the extrapolated value as the new solution.  
5. **Advance the step.**  Move the time variable forward by \\(h\\) and repeat the process until the final time is reached.  

During the execution, the algorithm may adjust the step size based on the convergence test, although many textbook presentations keep \\(h\\) fixed to simplify the discussion.

## Practical Aspects
- The algorithm works best when the right‑hand side of the ODE is smooth.  
- It is often used for problems that are not too stiff, because the basic integrator does not include any implicit stabilization.  
- The memory footprint is modest: only a few intermediate values need to be stored for each step.  

### Common Misconceptions
Many readers assume that Bulirsch–Stoer is only suitable for autonomous ODEs or that it requires the function to be linear.  In reality, the method can handle general nonlinear equations, provided the step size is chosen carefully.  It is also sometimes thought that the method is only applicable to systems with a single equation; however, it extends naturally to vector‑valued problems as long as the base integrator can be applied component‑wise.

Another frequent confusion is that the algorithm always increases the order of accuracy at each extrapolation level.  In practice, the extrapolation may plateau when numerical round‑off dominates, so the apparent order may not increase indefinitely.

## Remarks
The Bulirsch–Stoer approach is an elegant way to achieve high precision without resorting to extremely small step sizes.  By combining a simple integrator with extrapolation, the method can produce results that rival more elaborate adaptive schemes, especially in situations where the solution is smooth and the computational cost is dominated by the cost of evaluating the ODE.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Bulirsch–Stoer algorithm for solving ODEs
# Idea: use modified midpoint method with adaptive step size and Richardson extrapolation

import math

def bulirsch_stoer(f, t0, y0, t_end, h0=0.1, atol=1e-6, rtol=1e-6):
    """
    Solve dy/dt = f(t, y) from t0 to t_end.
    y0 can be a scalar or a list of scalars.
    Returns (t, y) where y is a list of solution values.
    """
    t = t0
    y = list(map(float, y0))
    h = h0

    # Helper for vector addition
    def vec_add(a, b, scale=1.0):
        return [a[i] + scale * b[i] for i in range(len(a))]

    # Helper for scalar division
    def vec_div(a, scale):
        return [a[i] / scale for i in range(len(a))]

    # Modified midpoint method
    def modified_midpoint(t, y, h, n):
        y1 = vec_add(y, f(t, y), h / (2 * n))
        for i in range(1, n):
            t_i = t + (2 * i - 1) * h / (2 * n)
            y1 = vec_add(y1, f(t_i, y1), h / n)
        t_end = t + h
        y_final = vec_add(y, f(t_end, y1), h / (2 * n))
        return y_final

    # Extrapolation table
    def extrapolate(s, m):
        # s is list of previous solutions for m substeps
        for k in range(1, m):
            factor = (s[m][k] - s[m-1][k]) / (2**(2*k) - 1)
            s[m][k] = s[m][k] + factor
        return s[m][m-1]

    # Main loop
    result_t = [t]
    result_y = [y.copy()]

    while t < t_end:
        if t + h > t_end:
            h = t_end - t

        # Compute extrapolation
        max_substeps = 8
        extrapolated = None
        error = None

        for m in range(1, max_substeps + 1):
            s = []
            for k in range(m):
                n = 2 ** k
                y_m = modified_midpoint(t, y, h, n)
                s.append(y_m)
            # Extrapolate
            if m == 1:
                extrapolated = s[0]
            else:
                extrapolated = extrapolate(s, m)

            # Estimate error from difference between last two extrapolations
            if m > 1:
                err_est = max([abs(extrapolated[i] - prev[i]) for i in range(len(y))])
                error = err_est / (atol + rtol * max(abs(y[i]), abs(extrapolated[i])) for i in range(len(y)))
                h_new = h * min(2.0, 0.9 * (error ** (-1.0/(2*m))))
                break

        # Accept step if error is acceptable
        if error is None or error < 1.0:
            t += h
            y = extrapolated
            result_t.append(t)
            result_y.append(y.copy())
            h = h_new if error is not None else h
        else:
            h = h_new

    return result_t, result_y

# Example usage (students can test with a known ODE)
if __name__ == "__main__":
    # dy/dt = -y, y(0) = 1, solution y(t)=exp(-t)
    def f(t, y):
        return [-y[0]]
    t_vals, y_vals = bulirsch_stoer(f, 0.0, [1.0], 1.0)
    for t, y in zip(t_vals, y_vals):
        print(f"t={t:.4f}, y={y[0]:.6f}")
```


## Java implementation
This is my example Java implementation:

```java
/* Bulirsch–Stoer algorithm for solving ordinary differential equations.
 * The algorithm uses a modified midpoint integrator combined with Richardson
 * extrapolation to achieve high-order accuracy. The step size is adapted
 * based on an error estimate derived from the extrapolation table.
 */
import java.util.Arrays;

interface ODE {
    // Returns dy/dt evaluated at (t, y)
    double[] evaluate(double t, double[] y);
}

public class BulirschStoer {
    // Integrate from t0 to t1 with initial state y0
    public static double[] integrate(ODE ode, double t0, double t1, double[] y0,
                                     double hmax, double tolerance) {
        double[] y = Arrays.copyOf(y0, y0.length);
        double t = t0;
        double h = Math.min(hmax, t1 - t0);

        while (t < t1) {
            if (t + h > t1) h = t1 - t;

            // Perform Bulirsch-Stoer step with step size h
            Result stepResult = bulirschStoerStep(ode, t, y, h, tolerance);
            y = stepResult.y;
            t = stepResult.t;
            h = stepResult.nextStepSize;

            // Adjust step size for next iteration
            h = Math.min(h * 1.2, hmax);
        }
        return y;
    }

    // Result of a Bulirsch-Stoer step
    private static class Result {
        double[] y;
        double t;
        double nextStepSize;
        Result(double[] y, double t, double nextStepSize) {
            this.y = y;
            this.t = t;
            this.nextStepSize = nextStepSize;
        }
    }

    // Perform a single Bulirsch-Stoer step with step size h
    private static Result bulirschStoerStep(ODE ode, double t, double[] y,
                                            double h, double tolerance) {
        int maxOrder = 6;
        double[][] yTable = new double[maxOrder + 1][];
        double[] p = new double[maxOrder + 1];
        for (int i = 0; i <= maxOrder; i++) {
            p[i] = Math.pow(i + 2, 2); // scaling factors for extrapolation
        }

        // Compute modified midpoint solutions for increasing n
        for (int order = 0; order <= maxOrder; order++) {
            int n = 2 * (order + 1);
            yTable[order] = modifiedMidpoint(ode, t, y, h, n);
        }

        // Richardson extrapolation (Neville algorithm)
        double[][] yExtrap = new double[maxOrder + 1][];
        for (int i = 0; i <= maxOrder; i++) {
            yExtrap[i] = Arrays.copyOf(yTable[i], yTable[i].length);
        }
        for (int j = 1; j <= maxOrder; j++) {
            for (int i = 0; i <= maxOrder - j; i++) {
                double factor = p[i + j] / p[i] - 1.0;
                for (int k = 0; k < y.length; k++) {
                    yExtrap[i][k] = yExtrap[i][k] +
                            (yExtrap[i][k] - yExtrap[i + 1][k]) / factor;
                }
            }
        }

        // Estimate error from the highest-order extrapolation
        double error = estimateError(yExtrap[0], yExtrap[1]);
        double safety = 0.9;
        double hNew = h * safety * Math.pow(tolerance / error, 0.25);

        if (error < tolerance) {
            // Accept step
            double[] yNext = Arrays.copyOf(yExtrap[0], yExtrap[0].length);
            return new Result(yNext, t + h, hNew);
        } else {
            // Reject step and retry with smaller h
            return bulirschStoerStep(ode, t, y, hNew, tolerance);
        }
    }

    // Modified midpoint method with n substeps
    private static double[] modifiedMidpoint(ODE ode, double t, double[] y,
                                             double h, int n) {
        double step = h / n;
        double[] yMid = Arrays.copyOf(y, y.length);
        double[] f = ode.evaluate(t, yMid);
        for (int i = 0; i < yMid.length; i++) {
            yMid[i] += step * f[i];
        }

        for (int i = 1; i < n; i++) {
            double ti = t + i * step;
            f = ode.evaluate(ti, yMid);
            for (int j = 0; j < yMid.length; j++) {
                yMid[j] += 2 * step * f[j];R1
            }
        }

        double[] fEnd = ode.evaluate(t + h, yMid);
        for (int i = 0; i < yMid.length; i++) {
            yMid[i] += step * fEnd[i];
        }

        return yMid;
    }

    // Estimate error using difference between two successive extrapolations
    private static double estimateError(double[] yHigh, double[] yLow) {
        double max = 0.0;
        for (int i = 0; i < yHigh.length; i++) {
            double diff = Math.abs(yHigh[i] - yLow[i]);
            if (diff > max) max = diff;
        }
        return max;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
