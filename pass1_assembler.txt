import re
from collections import OrderedDict
import os

# Default assembly program (used if no input.asm found)
SAMPLE_PROGRAM = """
START 200

MOVER AREG, ='5'
MOVEM AREG, X
L1 MOVER BREG, ='2'
ORIGIN L1+3
LTORG

NEXT ADD AREG, ='1'
SUB BREG, ='2'
BC LT, BACK
LTORG
BACK EQU L1

ORIGIN NEXT+5
MULT CREG, ='4'
STOP
X DS 1
END
"""

# Tables for opcodes and registers
IS = {'STOP':0, 'ADD':1, 'SUB':2, 'MULT':3, 'MOVER':4, 'MOVEM':5, 'COMP':6, 'BC':7, 'DIV':8, 'READ':9, 'PRINT':10}
DL = {'DC':1, 'DS':2}
AD = {'START':1, 'END':2, 'ORIGIN':3, 'EQU':4, 'LTORG':5}
REG = {'AREG':1, 'BREG':2, 'CREG':3, 'DREG':4}
COND = {'LT':1, 'LE':2, 'EQ':3, 'GT':4, 'GE':5, 'NE':6}

# Data structures
symbols = OrderedDict()
literals = []
pools = [0]
intermediate = []

def add_symbol(name, addr=None, defined=False, value=None):
    if name not in symbols:
        symbols[name] = {'address': addr, 'defined': defined, 'value': value}
    else:
        if addr is not None:
            symbols[name]['address'] = addr
            symbols[name]['defined'] = True
        if value is not None:
            symbols[name]['value'] = value

def add_literal(value):
    for i, lit in enumerate(literals):
        if lit['lit'] == value:
            return i
    literals.append({'lit': value, 'address': None, 'pool': len(pools) - 1})
    return len(literals) - 1

def assign_literal_addresses(start, lc):
    for i in range(start, len(literals)):
        if literals[i]['address'] is None:
            literals[i]['address'] = lc
            lc += 1
    return lc

def make_token(t):
    if t[0] in ("IS", "AD", "DL"): return f"({t[0]},{t[1]})"
    if t[0] in ("REG", "COND"): return f"({t[0]},{t[1]})"
    if t[0] in ("S", "L", "C"): return f"({t[0]},{t[1]})"
    return str(t)

def evaluate(expr):
    expr = expr.strip()
    if '+' in expr:
        a, b = expr.split('+', 1)
        return (symbols.get(a, {}).get('address', 0) or 0) + int(b)
    if '-' in expr:
        a, b = expr.split('-', 1)
        return (symbols.get(a, {}).get('address', 0) or 0) - int(b)
    if expr.isdigit():
        return int(expr)
    return symbols.get(expr, {}).get('address', 0) or 0

def process_line(line, lc):
    label = None
    parts = re.split(r'\s+', line, maxsplit=1)
    first = parts[0].upper()
    rest = parts[1] if len(parts) > 1 else ''

    if first not in IS and first not in DL and first not in AD:
        label = parts[0]
        add_symbol(label, addr=lc, defined=True)
        line = rest.strip()
        if not line:
            return lc
        first, rest = re.split(r'\s+', line, maxsplit=1)[0].upper(), line.split(' ', 1)[1] if ' ' in line else ''

    # START
    if first == 'START':
        start_val = int(rest.strip()) if rest.strip().isdigit() else 0
        intermediate.append((None, [("AD", AD['START']), ("C", start_val)]))
        return start_val

    # END
    if first == 'END':
        lc = assign_literal_addresses(pools[-1], lc)
        intermediate.append((None, [("AD", AD['END'])]))
        return lc

    # ORIGIN
    if first == 'ORIGIN':
        lc = evaluate(rest)
        intermediate.append((None, [("AD", AD['ORIGIN']), ("C", lc)]))
        return lc

    # EQU
    if first == 'EQU':
        value = evaluate(rest)
        if label:
            add_symbol(label, addr=value, defined=True, value=value)
        intermediate.append((None, [("AD", AD['EQU']), ("C", value)]))
        return lc

    # LTORG
    if first == 'LTORG':
        lc = assign_literal_addresses(pools[-1], lc)
        pools.append(len(literals))
        intermediate.append((None, [("AD", AD['LTORG'])]))
        return lc

    # Imperative statements
    if first in IS:
        code = IS[first]
        tokens = [("IS", code)]
        operands = [x.strip() for x in rest.split(',')] if rest else []

        for op in operands:
            if not op:
                continue
            if op.upper() in REG:
                tokens.append(("REG", REG[op.upper()]))
            elif op.upper() in COND:
                tokens.append(("COND", COND[op.upper()]))
            elif re.match(r"=('.+'|\".+\")", op):
                lit = op[1:]
                idx = add_literal(lit)
                tokens.append(("L", idx + 1))
            elif op.isdigit():
                tokens.append(("C", int(op)))
            else:
                add_symbol(op)
                index = list(symbols.keys()).index(op) + 1
                tokens.append(("S", index))
        intermediate.append((lc, tokens))
        return lc + 1

    # Declarative statements
    if first in DL:
        code = DL[first]
        tokens = [("DL", code)]
        val = rest.strip()
        if val.isdigit():
            val = int(val)
            tokens.append(("C", val))
            intermediate.append((lc, tokens))
            return lc + (val if first == 'DS' else 1)
        else:
            tokens.append(("C", val))
            intermediate.append((lc, tokens))
            return lc + 1

    return lc

def parse_program(lines):
    lc = 0
    for line in lines:
        lc = process_line(line.strip(), lc)
    assign_literal_addresses(pools[-1], lc)

def read_input():
    if os.path.exists('input.asm'):
        with open('input.asm', 'r') as f:
            return [x.strip() for x in f.readlines() if x.strip()]
    return [x.strip() for x in SAMPLE_PROGRAM.strip().splitlines() if x.strip()]

def print_output():
    print("\n--- Intermediate Code (Pass-I) ---")
    for lc, tokens in intermediate:
        print(f"{lc if lc is not None else '   '} ->", ' '.join(make_token(t) for t in tokens))

    print("\n--- Symbol Table ---")
    print("{:<5} {:<12} {:<10} {:<8}".format("Idx", "Symbol", "Address", "Value"))
    for i, (sym, info) in enumerate(symbols.items(), 1):
        addr = info['address'] if info['address'] is not None else "-"
        val = info['value'] if info['value'] is not None else "-"
        print(f"{i:<5} {sym:<12} {addr:<10} {val:<8}")

    print("\n--- Literal Table ---")
    print("{:<5} {:<10} {:<10} {:<6}".format("Idx", "Literal", "Address", "Pool"))
    for i, lit in enumerate(literals, 1):
        addr = lit['address'] if lit['address'] is not None else "-"
        print(f"{i:<5} {lit['lit']:<10} {addr:<10} {lit['pool']+1:<6}")

    print("\n--- Pool Table ---")
    for i, p in enumerate(pools, 1):
        print(f"Pool {i}: literal index start (1-based) = {p+1}")

def main():
    lines = read_input()
    parse_program(lines)
    print_output()

if __name__ == "__main__":
    main()
