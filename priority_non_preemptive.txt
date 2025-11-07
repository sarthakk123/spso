def priority_scheduling(processes):
    n = len(processes)
    processes.sort(key=lambda x: (x["arrival"], x["priority"]))
    time = 0
    completed = []
    remaining = processes.copy()

    while remaining:
        ready_queue = [p for p in remaining if p["arrival"] <= time]
        if not ready_queue:
            time += 1
            continue
        ready_queue.sort(key=lambda x: x["priority"])
        current = ready_queue[0]
        current["start"] = time
        current["completion"] = time + current["burst"]
        current["turnaround"] = current["completion"] - current["arrival"]
        current["waiting"] = current["turnaround"] - current["burst"]
        time = current["completion"]
        completed.append(current)
        remaining.remove(current)

    print("\n--- Priority Scheduling (Non-Preemptive) ---\n")
    print("{:<5} {:<10} {:<10} {:<10} {:<10} {:<12} {:<10} {:<10}".format(
        "PID", "Arrival", "Burst", "Priority", "Start", "Completion", "Turnaround", "Waiting"
    ))
    for p in completed:
        print("{:<5} {:<10} {:<10} {:<10} {:<10} {:<12} {:<10} {:<10}".format(
            p["pid"], p["arrival"], p["burst"], p["priority"], p["start"],
            p["completion"], p["turnaround"], p["waiting"]
        ))
    avg_tat = sum(p["turnaround"] for p in completed) / n
    avg_wt = sum(p["waiting"] for p in completed) / n
    print("\nAverage Turnaround Time:", round(avg_tat, 2))
    print("Average Waiting Time:", round(avg_wt, 2))


processes = []
n = int(input("Enter number of processes: "))

for i in range(n):
    pid = input(f"\nEnter Process ID for process {i+1}: ")
    arrival = int(input("Enter Arrival Time: "))
    burst = int(input("Enter Burst Time: "))
    priority = int(input("Enter Priority (lower number = higher priority): "))
    processes.append({"pid": pid, "arrival": arrival, "burst": burst, "priority": priority})

priority_scheduling(processes)
