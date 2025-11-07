def round_robin(processes, time_quantum):
    n = len(processes)
    remaining_time = [p["burst"] for p in processes]
    completion_time = [0] * n
    waiting_time = [0] * n
    turnaround_time = [0] * n

    t = 0
    ready_queue = []
    processes.sort(key=lambda x: x["arrival"])
    visited = [False] * n
    completed = 0

    while completed != n:
        for i in range(n):
            if processes[i]["arrival"] <= t and not visited[i]:
                ready_queue.append(i)
                visited[i] = True

        if not ready_queue:
            t += 1
            continue

        current = ready_queue.pop(0)
        exec_time = min(time_quantum, remaining_time[current])
        remaining_time[current] -= exec_time
        t += exec_time

        for i in range(n):
            if processes[i]["arrival"] <= t and not visited[i]:
                ready_queue.append(i)
                visited[i] = True

        if remaining_time[current] == 0:
            completion_time[current] = t
            completed += 1
        else:
            ready_queue.append(current)

    for i in range(n):
        turnaround_time[i] = completion_time[i] - processes[i]["arrival"]
        waiting_time[i] = turnaround_time[i] - processes[i]["burst"]

    print("\n--- Round Robin CPU Scheduling (Preemptive) ---\n")
    print("{:<5} {:<10} {:<10} {:<12} {:<12} {:<10}".format(
        "PID", "Arrival", "Burst", "Completion", "Turnaround", "Waiting"
    ))
    for i in range(n):
        print("{:<5} {:<10} {:<10} {:<12} {:<12} {:<10}".format(
            processes[i]["pid"], processes[i]["arrival"], processes[i]["burst"],
            completion_time[i], turnaround_time[i], waiting_time[i]
        ))

    avg_tat = sum(turnaround_time) / n
    avg_wt = sum(waiting_time) / n
    print("\nAverage Turnaround Time:", round(avg_tat, 2))
    print("Average Waiting Time:", round(avg_wt, 2))


processes = []
n = int(input("Enter number of processes: "))
time_quantum = int(input("Enter Time Quantum: "))

for i in range(n):
    pid = input(f"\nEnter Process ID for process {i+1}: ")
    arrival = int(input("Enter Arrival Time: "))
    burst = int(input("Enter Burst Time: "))
    processes.append({"pid": pid, "arrival": arrival, "burst": burst})

round_robin(processes, time_quantum)
