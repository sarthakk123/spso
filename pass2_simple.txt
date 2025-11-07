# PASS-II of Two Pass Assembler (Simple Version)
# Amit Kolpe - 2025

symbol_table = {
    "X": 214,
    "L1": 202,
    "NEXT": 207,
    "BACK": 202
}

literal_table = {
    1: 205,  # ='5'
    2: 206,  # ='2'
    3: 210,  # ='1'
    4: 211,  # ='2'
    5: 215   # ='4'
}

intermediate_code = [
    "(AD,01) (C,200)",
    "(IS,04) 1 (L,1)",
    "(IS,05) 1 (S,1)",
    "(IS,04) 2 (L,2)",
    "(AD,03) (S,2)+3",
    "(AD,05)",
    "(L,1)",
    "(L,2)",
    "(IS,01) 1 (L,3)",
    "(IS,02) 2 (L,4)",
    "(IS,07) 1 (S,4)",
    "(AD,05)",
    "(L,3)",
    "(L,4)",
    "(AD,04) (S,2)",
    "(IS,03) 3 (L,5)",
    "(IS,00)",
    "(DL,02) (C,1)",
    "(AD,02)"
]

# Operation Code Table (for reference)
IS = {0: "STOP", 1: "ADD", 2: "SUB", 3: "MULT", 4: "MOVER", 5: "MOVEM", 6: "COMP", 7: "BC", 8: "DIV", 9: "READ", 10: "PRINT"}
REG = {1: "AREG", 2: "BREG", 3: "CREG"}

lc = 200  # Location Counter starts from 200

print("\n--- PASS-II OUTPUT (Machine Code) ---\n")
print("LC\tMachine Code")

for line in intermediate_code:
    line = line.strip()

    # AD statements (Assembler Directives)
    if "(AD" in line:
        if "01" in line:  # START
            lc = 200
        elif "02" in line:  # END
            break
        elif "03" in line:  # ORIGIN
            parts = line.split()
            expr = parts[-1].replace("(S,", "").replace(")", "")
            sym, offset = expr.split("+")
            new_lc = symbol_table["L1"] + int(offset)
            lc = new_lc
        elif "04" in line:  # EQU
            pass
        elif "05" in line:  # LTORG
            for i in literal_table:
                print(f"{lc}\t00 00 {literal_table[i]}")
                lc += 1
        continue

    # DL (Declarative)
    if "(DL" in line:
        if "(C," in line:
            const = line.split("(C,")[1].replace(")", "")
            print(f"{lc}\t00 00 {const}")
            lc += 1
        continue

    # Imperative Statements (IS)
    if "(IS" in line:
        parts = line.replace("(", "").replace(")", "").replace(",", " ").split()
        opcode = int(parts[1])
        reg = 0
        addr = 0

        # Get register
        for p in parts:
            if p.isdigit() and int(p) in REG:
                reg = int(p)
        
        # Get symbol or literal
        if "(S," in line:
            sym_index = int(line.split("(S,")[1].split(")")[0])
            sym_name = list(symbol_table.keys())[sym_index - 1]
            addr = symbol_table[sym_name]
        elif "(L," in line:
            lit_index = int(line.split("(L,")[1].split(")")[0])
            addr = literal_table[lit_index]
        elif "(C," in line:
            addr = int(line.split("(C,")[1].split(")")[0])

        print(f"{lc}\t{opcode:02} {reg:02} {addr}")
        lc += 1

print("\n--- END OF MACHINE CODE ---")
