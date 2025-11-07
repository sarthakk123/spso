def lru_page_replacement(pages, frames_count):
    frames = []
    page_faults = 0
    recent_use = {}

    print("\n--- LRU Page Replacement ---\n")
    print("{:<10} {:<30} {:<10}".format("Page", "Frames", "Page Fault"))

    for time, page in enumerate(pages):
        if page not in frames:
            if len(frames) < frames_count:
                frames.append(page)
            else:
                lru_page = min(recent_use, key=recent_use.get)
                frames[frames.index(lru_page)] = page
                del recent_use[lru_page]
            page_faults += 1
            fault_status = "Yes"
        else:
            fault_status = "No"
        recent_use[page] = time
        print("{:<10} {:<30} {:<10}".format(page, str(frames), fault_status))

    print("\nTotal Page Faults:", page_faults)


pages = list(map(int, input("Enter page reference string (space separated): ").split()))
frames_count = int(input("Enter number of frames: "))
lru_page_replacement(pages, frames_count)
