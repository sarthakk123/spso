def optimal_page_replacement(pages, frames_count):
    frames = []
    page_faults = 0
    print("\n--- Optimal Page Replacement ---\n")
    print("{:<10} {:<30} {:<10}".format("Page", "Frames", "Page Fault"))

    for i in range(len(pages)):
        page = pages[i]
        if page not in frames:
            if len(frames) < frames_count:
                frames.append(page)
            else:
                future_use = []
                for f in frames:
                    if f in pages[i+1:]:
                        future_use.append(pages[i+1:].index(f))
                    else:
                        future_use.append(float('inf'))
                to_replace = future_use.index(max(future_use))
                frames[to_replace] = page
            page_faults += 1
            fault_status = "Yes"
        else:
            fault_status = "No"
        print("{:<10} {:<30} {:<10}".format(page, str(frames), fault_status))

    print("\nTotal Page Faults:", page_faults)


pages = list(map(int, input("Enter page reference string (space separated): ").split()))
frames_count = int(input("Enter number of frames: "))
optimal_page_replacement(pages, frames_count)
