# passI macro.py
import os
import re
from collections import OrderedDict

SAMPLE = """START
MACRO
INCR &ARG1, &ARG2
ADD AREG, &ARG1
MEND
MACRO
DECR &ARG3, &ARG4
SUB AREG, &ARG3
MOVER CREG, &ARG4
MEND
INCR N1, N2
DECR N3,N4
END
"""

def load_input():
    if os.path.exists("input.mac"):
        with open("input.mac","r") as f:
            return [ln.rstrip() for ln in f.readlines()]
    else:
        return [ln.rstrip() for ln in SAMPLE.splitlines()]

def clean_tokens(s):
    return s.strip()

def parse_parameter_list(param_str):
    parts = [p.strip() for p in param_str.split(',') if p.strip()!='']
    # remove leading & if present
    return [p[1:] if p.startswith('&') else p for p in parts]

def replace_formals_with_pos(line, formal_list):
    # replace occurrences of &NAME by (P,i) where i is 1-based index
    def repl(m):
        name = m.group(0)[1:]  # remove &
        if name in formal_list:
            return f"(P,{formal_list.index(name)+1})"
        return m.group(0)
    return re.sub(r"&[A-Za-z0-9_]+", repl, line)

def main():
    lines = load_input()
    i = 0
    MNT = []   # list of dicts: {'name', 'mdt_index'}
    MDT = []   # list of lines (strings). We'll use 1-based indexing when printing.
    ALA = OrderedDict()  # macro_name -> list of formal parameters in order

    while i < len(lines):
        line = lines[i].strip()
        if line.upper() == "MACRO":
            # next line must be macro header: name and parameters
            i += 1
            header = lines[i].strip()
            # header example: INCR &ARG1, &ARG2
            parts = header.split(None, 1)
            macro_name = parts[0].strip()
            params = []
            if len(parts) > 1:
                params = parse_parameter_list(parts[1])
            # record in MNT
            mdt_index = len(MDT) + 1  # next MDT line index (1-based)
            MNT.append({'name':macro_name, 'mdt_index':mdt_index})
            ALA[macro_name] = params[:]  # copy list
            # now read body until MEND
            i += 1
            while i < len(lines):
                body_line = lines[i].strip()
                if body_line.upper() == "MEND":
                    MDT.append("MEND")
                    break
                # replace formals with positional markers
                replaced = replace_formals_with_pos(body_line, params)
                MDT.append(replaced)
                i += 1
        i += 1

    # Print results
    print("\n--- MNT (Macro Name Table) ---")
    print("{:<5} {:<12} {:<10}".format("Idx","Macro Name","MDT Index"))
    for idx, e in enumerate(MNT, start=1):
        print(f"{idx:<5} {e['name']:<12} {e['mdt_index']:<10}")

    print("\n--- MDT (Macro Definition Table) ---")
    print("{:<5} {}".format("Idx","Definition"))
    for idx, line in enumerate(MDT, start=1):
        print(f"{idx:<5} {line}")

    print("\n--- ALA (Argument List Array) ---")
    for macro, params in ALA.items():
        print(f"\nMacro: {macro}")
        if not params:
            print("  (no arguments)")
        else:
            print("{:<6} {}".format("Pos", "Parameter"))
            for p_idx, name in enumerate(params, start=1):
                print(f"{p_idx:<6} {name}")

if __name__ == "__main__":
    main()
