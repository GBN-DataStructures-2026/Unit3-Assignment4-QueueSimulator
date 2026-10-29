# Lab: Discrete Event Queue Simulation (Wait-Time Analytics)

## Objective
Build a discrete time-step simulation representing an 8-hour workday (480 minutes) to analyze how queue architecture impacts customer wait times and server efficiency. You will compare a **Multi-Queue System** (grocery store style) against a **Single Shared Queue System** (airport security/bank snake line).

---

## Project Timeline (3 Class Periods)

* **Period 1 (90 mins) — Design & Single-Server Engine**
  * **First 20–30 mins:** Mandatory Whiteboard / Paper Design Session (Instructor Check-Off required before typing!).
  * **Remaining 60 mins:** Build `Customer` model, base 480-minute tick loop, arrival generator, and `VERBOSE` console output.
* **Period 2 (70 mins) — Multi-Server Architectures**
  * Expand simulation to $N$ servers.
  * Implement dispatch logic for both **Multi-Queue** and **Single Shared Queue** modes.
  * Refactor tick logic into clean, isolated helper methods.
* **Period 3 (90 mins) — Statistical Analytics & Final Demonstrations**
  * Run statistical trials across both queue architectures.
  * Collect metrics (Average Wait, Max Wait, Max Queue Length, Server Utilization).
  * Answer reflection questions and demonstrate live terminal output to the instructor.

---

## 🛑 MANDATORY PHASE 0: Whiteboard & Paper Design Gate

> **RULE:** You may NOT open VS Code or write a single line of Java until your pair has drawn out your system blueprint on paper or a whiteboard and received an instructor check-off mark.

Keyboard-first coding in simulation labs leads to hours of painful debugging. Spend your first 20–30 minutes answering these questions on paper:

### What to Draw & Write on Your Board:
1. **Queue Implementation Choice:** Decide as a pair whether you will use Java's built-in `java.util.Queue` interface (e.g., `LinkedList` or `ArrayDeque`) or copy over your own custom `ArrayQueue` / `LinkedQueue` implementation created earlier in Unit 3. *(If using your own ADT, make sure it supports all required methods!)*
2. **Class Taxonomy:** What classes will you create? (You decide! Will you make a `Server` class? A `QueueManager`? How many classes do you need, and what instance variables will each hold?)
3. **The "Minute Tick" Flowchart:** Draw the exact order of operations that occurs inside minute tick $t$:
   * *Do customers arrive before or after servers update their remaining service time?*
   * *When does a customer's wait time start and stop accumulating?*
4. **Paper Simulation Trace:** Hand-trace 3 customers through 5 minutes of simulated time on paper:
   * **Customer A:** Arrives minute 1, needs 3 mins service.
   * **Customer B:** Arrives minute 2, needs 2 mins service.
   * **Customer C:** Arrives minute 2, needs 4 mins service.
   * *Write down the state of your queue(s) and server(s) for ticks 1, 2, 3, 4, and 5.*

---

## Technical Specifications

### Simulation Parameters
Configure these as **constants or configuration fields** in your code (no `Scanner` input needed) so you can run test trials quickly:

* `SIMULATION_MINUTES = 480`: Total duration of the simulation (8-hour workday).
* `ARRIVAL_PROBABILITY = 0.35`: The chance (`0.0` to `1.0`) that a new customer arrives during any given minute tick.*
* `MIN_SERVICE_TIME = 3` & `MAX_SERVICE_TIME = 10`: Service duration range (in minutes) assigned randomly to each customer.
* `NUM_SERVERS = 3`: Total cashiers/tellers on duty.
* `VERBOSE = true`: Debugging flag controlling whether tick-by-tick logs are displayed.

*\*Note: Evaluating a constant arrival probability every minute tick creates stochastic customer flow, statistically known as a Poisson arrival process.*

---

## The `VERBOSE` Debugging Flag

Printing terminal output on every tick helps you debug, but printing 480 ticks during batch runs creates massive console clutter. Use a `boolean VERBOSE` flag to control display detail:

```java
public class Simulation {
    // Set to true while building/debugging; set to false for clean statistical runs
    private static final boolean VERBOSE = true; 

    public void run() {
        for (int tick = 0; tick < SIMULATION_MINUTES; tick++) {
            processDepartures(tick);
            processArrivals(tick);
            dispatchCustomers(tick);

            // Print tick-by-tick snapshot only when VERBOSE is enabled
            if (VERBOSE && tick < 30) { // Limit output to first 30 ticks during debug
                printState(tick);
            }
        }
        
        // Always print final summary statistics regardless of VERBOSE flag
        printSummaryStats();
    }
}
```

---

## Sample State Display (`printState`)

When `VERBOSE` is set to `true`, `printState(tick)` should output a clean visual snapshot of your system so you can see customers moving through lines in real time:

### Multi-Queue Visual Output Example
```text
=== Time: Minute 142 ===
[Event] Customer #84 arrived -> Joined Queue 2

  Queue 0: [Serving C78 (3m left)] C81 C83
  Queue 1: [Serving C79 (1m left)] C82
  Queue 2: [Serving C80 (5m left)] C84
--------------------------------------------------
```

### Single Shared Queue Visual Output Example
```text
=== Time: Minute 142 ===
[Event] Customer #86 arrived -> Joined Main Line

  Main Line: [C84 (6m), C85 (3m), C86 (8m)]
  Teller 0:  Serving C81 (1m remaining)
  Teller 1:  Serving C82 (4m remaining)
  Teller 2:  Serving C83 (2m remaining)
--------------------------------------------------
```

---

## Helper Method Guidelines

While your pair designs the class structure, you must ensure your main simulation loop stays clean and readable by decomposing your tick operations into clear helper methods (e.g., `processArrivals`, `processDepartures`, `dispatchCustomers`, `printState`, and `printSummaryStats`).

---

## Core Experiment: Multi-Queue vs. Single Shared Queue

You will run your simulation under two distinct modes using identical arrival parameters:

1. **Multi-Queue Mode (Grocery Store):**
   * Arriving customers evaluate all lines and join the shortest line.
   * Once in a line, customers remain there until served.
2. **Single Shared Queue Mode (Bank Snake Line):**
   * All arriving customers enter one central queue.
   * As soon as any server becomes free, they dequeue the next customer from the front of the main line.

---

## Sample Final Summary Output

```text
==================================================
           480-MINUTE QUEUE SIMULATION            
==================================================
Parameters: 3 Servers | Arrival Prob: 0.35 | Service: 3-10 mins

--- MULTI-QUEUE RESULTS ---
Total Customers Served: 168
Average Wait Time:       11.4 minutes
Max Wait Time:           38.0 minutes
Max Queue Length:        9 customers
Server Utilization:      88.2%
Customers Remaining:     4

--- SINGLE SHARED QUEUE RESULTS ---
Total Customers Served: 171
Average Wait Time:       4.2 minutes
Max Wait Time:           12.0 minutes
Max Queue Length:        5 customers
Server Utilization:      94.1%
Customers Remaining:     1

CONCLUSION: Single Shared Queue reduced average wait time by 63.1% 
and reduced max wait time from 38 mins down to 12 mins.
```

---

## Reflection Questions

1. **The Unlucky Customer:** Why does the Multi-Queue system produce a significantly worse **Max Wait Time** than the Single Shared Queue, even when average wait times are relatively close?
2. **Eliminating Inefficiency:** How does a Single Shared Queue guarantee that no server sits idle while customers are waiting?
3. **Peak Traffic Scaling:** If `ARRIVAL_PROBABILITY` increases by just 10%, what happens to queue length over a 480-minute shift? Is the growth linear or exponential?

---

## Early Finisher Extensions

If your team completes the base assignment early, implement one of these extensions:

### Extension 1: Customer Reneging & Balking (Impatience Modeling)
* **Balking:** If all queues exceed a threshold length (e.g., $> 8$ customers), an arriving customer leaves immediately without joining a queue. Track `totalBalkedCustomers`.
* **Reneging:** If a customer's wait time exceeds a maximum patience threshold (e.g., $15$ minutes in line), they leave the queue unsatisfied. Track `totalReneges`.

### Extension 2: Dynamic Server Scaling (Peak Hour Management)
* Implement a dynamic manager that monitors line lengths.
* If average wait time or line length exceeds a threshold, automatically "open" an extra backup server window (`NUM_SERVERS++`). When lines shrink below a lower threshold, close the extra window (`NUM_SERVERS--`).

### Extension 3: Express / Priority Queue (VIP Line)
* Add a second customer tier: `STANDARD` vs. `EXPRESS` (e.g., customers needing $\le 3$ minutes of service time).
* Implement an express lane or priority dispatch rule where express customers jump ahead of standard customers in the shared queue. Evaluate how this impacts standard customer wait times.
