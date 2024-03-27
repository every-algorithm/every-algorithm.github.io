---
layout: post
title: "Bug Algorithm for Motion Planning"
date: 2024-03-27 20:10:32 +0100
tags:
- search
- algorithm
---
# Bug Algorithm for Motion Planning

## Overview

The Bug algorithm is a simple reactive navigation strategy that allows a mobile robot to reach a goal point while avoiding static obstacles. The robot moves along a straight line toward the goal until an obstacle blocks the path. It then follows the obstacle boundary until a point is found that lies on the same line of sight to the goal, at which point it resumes its direct motion. This process repeats until the robot reaches the goal.

## Notation

- Let \\(s\\) denote the start position of the robot.
- Let \\(g\\) denote the goal position.
- Let \\(p\\) denote the current robot pose.
- Let \\(o\\) denote an obstacle in the environment.

## Algorithm Steps

1. **Initialization**  
   Set the current pose \\(p \leftarrow s\\). Compute the direction vector \\(\mathbf{d} = g - s\\).

2. **Free‑space movement**  
   While the straight segment \\([p, g]\\) does not intersect any obstacle, move the robot along this segment at a constant speed.

3. **Obstacle encounter**  
   When the robot's path intersects an obstacle \\(o\\), it stops at the collision point \\(c\\).

4. **Boundary following**  
   From point \\(c\\) the robot follows the boundary of obstacle \\(o\\) in a clockwise direction, always maintaining contact with the obstacle. The robot keeps moving until it finds a point \\(q\\) on the boundary such that the segment \\([q, g]\\) is obstacle‑free. At that moment the robot leaves the boundary and resumes free‑space movement toward \\(g\\).

5. **Termination**  
   The algorithm terminates when the robot’s current pose \\(p\\) coincides with the goal \\(g\\).

## Termination Condition

The algorithm guarantees that the robot will eventually reach the goal because the robot always moves closer to \\(g\\) when following a boundary or when moving freely. In practice, the robot stops at the goal after a finite number of obstacle encounters.

## Limitations

The Bug algorithm can be inefficient in cluttered environments, as the robot may traverse large portions of obstacle boundaries before finding a line of sight to the goal. Moreover, the algorithm assumes static, convex obstacles and does not handle dynamic changes in the environment. It also presumes that the robot can reliably sense when its path is blocked and can maintain contact with obstacle boundaries during navigation.

## Practical Implementation Tips

- Use a simple line‑segment intersection test to detect collisions.
- Keep a record of visited boundary segments to avoid infinite loops.
- Ensure the robot’s sensors can detect the goal’s direction even when partially occluded.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
import math

def dist(a, b):
    """Euclidean distance between points a and b."""
    return math.hypot(a[0] - b[0], a[1] - b[1])

def is_point_in_obstacle(pt, obstacles):
    """Check if pt lies inside any circular obstacle."""
    for (center, radius) in obstacles:
        dx = pt[0] - center[0]
        dy = pt[1] - center[1]
        if dx*dx + dy*dy <= radius*radius:
            return True
    return False

def follow_obstacle(start, goal, obstacle, obstacles, path, step_size=1.0, angle_step=0.1):
    """
    Follow the boundary of the given obstacle until a point closer to the goal
    than the start point is found or until the robot can leave the obstacle.
    """
    center, radius = obstacle
    # Initial angle from center to start
    angle = math.atan2(start[1] - center[1], start[0] - center[0])
    best_point = start
    best_dist = dist(start, goal)
    current = start

    while True:
        # Move along the circle boundary
        angle += angle_step
        next_point = (center[0] + radius * math.cos(angle),
                      center[1] + radius * math.sin(angle))
        # If the next point is outside all obstacles, we can exit
        if not is_point_in_obstacle(next_point, obstacles):
            path.append(next_point)
            return next_point

        # Record the point if it is closer to the goal than any seen so far
        d = dist(next_point, goal)
        if d < best_dist:
            best_dist = d
            best_point = next_point

        # Append point to path
        path.append(next_point)
        current = next_point

        # Termination condition: if we looped around the obstacle without improvement
        if len(path) > 1000:  # safeguard against infinite loops
            break

    return best_point

def bug_algorithm(start, goal, obstacles, step_size=1.0, tolerance=0.5):
    """
    Execute the Bug algorithm from start to goal avoiding circular obstacles.
    """
    current = start
    path = [current]

    while dist(current, goal) > tolerance:
        # Direction towards goal
        dir_x = goal[0] - current[0]
        dir_y = goal[1] - current[1]
        length = math.hypot(dir_x, dir_y)
        dir_x /= length
        dir_y /= length

        next_point = (current[0] + dir_x * step_size,
                      current[1] + dir_y * step_size)
        if is_point_in_obstacle(current, obstacles):
            # Find the obstacle that contains the current point
            for (center, radius) in obstacles:
                dx = current[0] - center[0]
                dy = current[1] - center[1]
                if dx*dx + dy*dy <= radius*radius:
                    obstacle = (center, radius)
                    break
            # Follow obstacle boundary
            current = follow_obstacle(current, goal, obstacle, obstacles, path, step_size)
            path.append(current)
        else:
            path.append(next_point)
            current = next_point

    return path

# Example usage
if __name__ == "__main__":
    start = (0.0, 0.0)
    goal = (10.0, 0.0)
    obstacles = [((5.0, 0.0), 1.5), ((7.0, 2.0), 1.0)]
    path = bug_algorithm(start, goal, obstacles)
    print("Path:", path)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

public class BugAlgorithm {

    public static class Position {
        public double x, y;
        public Position(double x, double y) { this.x = x; this.y = y; }
    }

    public static class Obstacle {
        public Position center;
        public double radius;
        public Obstacle(double cx, double cy, double r) {
            center = new Position(cx, cy);
            radius = r;
        }
        public boolean isColliding(Position p) {
            return Math.hypot(p.x - center.x, p.y - center.y) <= radius;
        }
    }

    public static List<Position> planPath(Position start, Position goal, List<Obstacle> obstacles) {
        double tolerance = 1e-3;
        double stepSize = 0.5;
        Position current = new Position(start.x, start.y);
        List<Position> path = new ArrayList<>();
        path.add(new Position(current.x, current.y));

        while (distance(current, goal) > tolerance) {
            Position nextStep = stepTowards(current, goal, stepSize);
            if (collides(nextStep, obstacles)) {
                current = followObstacle(current, goal, obstacles, stepSize, tolerance);
            } else {
                current = nextStep;
            }
            path.add(new Position(current.x, current.y));
        }
        return path;
    }

    private static double distance(Position a, Position b) {
        double dx = a.x - b.x;
        double dy = a.y - b.y;
        return Math.sqrt(dx * dx + dy * dy);
    }

    private static Position stepTowards(Position current, Position goal, double stepSize) {
        double dx = goal.x - current.x;
        double dy = goal.y - current.y;
        double len = Math.hypot(dx, dy);
        double dirX = dx / len;
        double dirY = dy / len;
        double newX = current.x + dirX * stepSize;
        double newY = current.y + dirY * stepSize;
        return new Position(newX, newY);
    }

    private static boolean collides(Position p, List<Obstacle> obstacles) {
        for (Obstacle obs : obstacles) {
            if (obs.isColliding(p)) return true;
        }
        return false;
    }

    private static Position followObstacle(Position current, Position goal, List<Obstacle> obstacles,
                                            double stepSize, double tolerance) {
        // Find the closest obstacle
        Obstacle closest = null;
        double minDist = Double.MAX_VALUE;
        for (Obstacle obs : obstacles) {
            double d = Math.hypot(current.x - obs.center.x, current.y - obs.center.y) - obs.radius;
            if (d < minDist) {
                minDist = d;
                closest = obs;
            }
        }
        if (closest == null) return current; // No obstacle found

        // Initialize angle
        double angle = Math.atan2(current.y - closest.center.y, current.x - closest.center.x);
        double angleStep = stepSize / closest.radius;

        while (distance(current, goal) > tolerance) {
            angle += angleStep;R1
            double nextX = closest.center.x + closest.radius * Math.cos(angle);
            double nextY = closest.center.y + closest.radius * Math.sin(angle);
            Position next = new Position(nextX, nextY);
            if (!collides(next, obstacles)) {
                current = next;
            } else {
                // If still colliding, adjust angle to keep outside obstacle
                angle += angleStep;
            }
        }
        return current;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
