def fifo_page_replacement(pages, frames_count):
    frames = []
    page_faults = 0
    print("\n--- FIFO Page Replacement ---\n")
    print("{:<10} {:<30} {:<10}".format("Page", "Frames", "Page Fault"))
    for page in pages:
        if page not in frames:
            if len(frames) < frames_count:
                frames.append(page)
            else:
                frames.pop(0)
                frames.append(page)
            page_faults += 1
            fault_status = "Yes"
        else:
            fault_status = "No"
        print("{:<10} {:<30} {:<10}".format(page, str(frames), fault_status))
    print("\nTotal Page Faults:", page_faults)


pages = list(map(int, input("Enter page reference string (space separated): ").split()))
frames_count = int(input("Enter number of frames: "))
fifo_page_replacement(pages, frames_count)
