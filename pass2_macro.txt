

MNT = {
    "INCR": 0,   
    "DECR": 4   
}

# Macro Definition Table (MDT)
MDT = [
    "INCR &ARG1, &ARG2",
    "MOVER AREG, &ARG1",
    "ADD AREG, &ARG2",
    "MEND",
    "DECR &ARG3, &ARG4",
    "MOVER AREG, &ARG3",
    "SUB AREG, &ARG4",
    "MEND"
]

# Argument List Array (ALA)
ALA = {
    "INCR": ["ARG1", "ARG2"],
    "DECR": ["ARG3", "ARG4"]
}

# Intermediate (main) code with macro calls
INTERMEDIATE_CODE = [
    "START",
    "INCR N1, N2",
    "DECR N3, N4",
    "END"
]

# Function to expand macros
def expand_macro(macro_name, actual_args):
    start_index = MNT[macro_name]
    params = ALA[macro_name]
    output_lines = []

    # Create mapping between formal parameters and actual arguments
    arg_map = {params[i]: actual_args[i] for i in range(len(params))}

    # Traverse MDT lines until MEND
    for i in range(start_index + 1, len(MDT)):
        line = MDT[i]
        if line.strip().upper() == "MEND":
            break
        # Replace &ARGx with actual arguments
        for formal, actual in arg_map.items():
            line = line.replace("&" + formal, actual)
        output_lines.append(line)
    return output_lines


def main():
    expanded_code = []

    for line in INTERMEDIATE_CODE:
        line = line.strip()
        if not line:
            continue

        # If line is a macro call
        tokens = line.split()
        macro_name = tokens[0]

        if macro_name in MNT:
            # Extract arguments separated by commas
            args_part = line[len(macro_name):].strip()
            actual_args = [x.strip() for x in args_part.split(',')]
            expanded_lines = expand_macro(macro_name, actual_args)
            expanded_code.extend(expanded_lines)
        else:
            # Normal assembly line
            expanded_code.append(line)

    # Print final expanded code
    print("\n--- EXPANDED ASSEMBLY CODE (PASS-II OUTPUT) ---\n")
    for line in expanded_code:
        print(line)


if __name__ == "__main__":
    main()
