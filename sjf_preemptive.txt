def sjf_preemptive(processes):
    n = len(processes)
    remaining_time = [p["burst"] for p in processes]
    complete = 0
    t = 0
    shortest = None
    finish_time = [0] * n
    waiting_time = [0] * n
    turnaround_time = [0] * n

    while complete != n:
        shortest = None
        min_remaining = float("inf")
        for i in range(n):
            if processes[i]["arrival"] <= t and remaining_time[i] > 0:
                if remaining_time[i] < min_remaining:
                    min_remaining = remaining_time[i]
                    shortest = i
        if shortest is None:
            t += 1
            continue
        remaining_time[shortest] -= 1
        t += 1
        if remaining_time[shortest] == 0:
            complete += 1
            finish_time[shortest] = t
            turnaround_time[shortest] = finish_time[shortest] - processes[shortest]["arrival"]
            waiting_time[shortest] = turnaround_time[shortest] - processes[shortest]["burst"]

    print("\n--- SJF (Preemptive) CPU Scheduling ---\n")
    print("{:<5} {:<10} {:<10} {:<12} {:<12} {:<10}".format(
        "PID", "Arrival", "Burst", "Completion", "Turnaround", "Waiting"
    ))
    for i in range(n):
        print("{:<5} {:<10} {:<10} {:<12} {:<12} {:<10}".format(
            processes[i]["pid"], processes[i]["arrival"], processes[i]["burst"],
            finish_time[i], turnaround_time[i], waiting_time[i]
        ))
    avg_tat = sum(turnaround_time) / n
    avg_wt = sum(waiting_time) / n
    print("\nAverage Turnaround Time:", round(avg_tat, 2))
    print("Average Waiting Time:", round(avg_wt, 2))

processes = []
n = int(input("Enter number of processes: "))

for i in range(n):
    pid = input(f"\nEnter Process ID for process {i+1}: ")
    arrival = int(input("Enter Arrival Time: "))
    burst = int(input("Enter Burst Time: "))
    processes.append({"pid": pid, "arrival": arrival, "burst": burst})

processes.sort(key=lambda x: x["arrival"])
sjf_preemptive(processes)
