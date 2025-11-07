def first_fit(partitions, processes):
    n = len(partitions)
    m = len(processes)
    allocation = [-1] * m
    for i in range(m):
        for j in range(n):
            if partitions[j] >= processes[i]:
                allocation[i] = j
                partitions[j] -= processes[i]
                break
    print("\n--- First Fit Memory Allocation ---\n")
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
first_fit(partitions, processes)
