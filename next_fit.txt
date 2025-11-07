def next_fit(partitions, processes):
    n = len(partitions)
    m = len(processes)
    allocation = [-1] * m
    last_alloc_index = 0

    for i in range(m):
        count = 0
        allocated = False
        while count < n:
            j = (last_alloc_index + count) % n
            if partitions[j] >= processes[i]:
                allocation[i] = j
                partitions[j] -= processes[i]
                last_alloc_index = j
                allocated = True
                break
            count += 1
        if not allocated:
            allocation[i] = -1

    print("\n--- Next Fit Memory Allocation ---\n")
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
next_fit(partitions, processes)
