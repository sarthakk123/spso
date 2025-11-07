def best_fit(partitions, processes):
    n = len(partitions)
    m = len(processes)
    allocation = [-1] * m
    partition_used = [False] * n
    for i in range(m):
        best_idx = -1
        for j in range(n):
            if partitions[j] >= processes[i]:
                if best_idx == -1 or partitions[j] < partitions[best_idx]:
                    best_idx = j
        if best_idx != -1:
            allocation[i] = best_idx
            partitions[best_idx] -= processes[i]
            partition_used[best_idx] = True
    print("\n--- Best Fit Memory Allocation ---\n")
    print("{:<10} {:<15} {:<15}".format("Process", "Size (K)", "Allocated Partition"))
    for i in range(m):
        if allocation[i] != -1:
            print("{:<10} {:<15} {:<15}".format(f"P{i+1}", processes[i], f"Partition {allocation[i]+1}"))
        else:
            print("{:<10} {:<15} {:<15}".format(f"P{i+1}", processes[i], "Not Allocated"))
    print("\nRemaining Memory in Each Partition:")
    for i in range(n):
        print(f"Partition {i+1}: {partitions[i]}K")

partitions = list(map(int, input("Enter memory partitions (space separated in K): ").split()))
processes = list(map(int, input("Enter process sizes (space separated in K): ").split()))
best_fit(partitions, processes)
