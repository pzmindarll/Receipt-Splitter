import json
from decimal import Decimal, ROUND_HALF_UP, getcontext

# ── Increase precision if you like (not strictly required)
getcontext().prec = 9

# ── Config
DATA_FILE = 'receipt.json'
TIP_RATE  = Decimal('0.20')     # 20%
TAX_RATE  = Decimal('0.0888')   # ≈ 8.88%

def load_items():
    with open(DATA_FILE) as f:
        return json.load(f)

def save_items(items):
    with open(DATA_FILE, 'w') as f:
        json.dump(items, f, indent=2)

def fmt(value):
    """Format anything (float, int, or Decimal) as $XX.XX."""
    d = Decimal(str(value))
    return f"${d.quantize(Decimal('0.01'), ROUND_HALF_UP)}"

def main():
    # ── 1) Who are you?
    name = input("Enter your name: ").strip()
    if not name:
        print("Name is required.")
        return

    # ── 2) How many in total? (for communal split)
    guests_str = input("Total number of participants (for communal items): ").strip() or "1"
    try:
        guests = int(guests_str)
    except ValueError:
        print("Invalid number of participants.")
        return

    # ── 3) Load data & compute communal share
    items = load_items()
    communal = [it for it in items if it.get('communal')]
    communal_total = sum(Decimal(str(it['price'])) for it in communal)
    communal_share = (communal_total / Decimal(guests)) if guests else Decimal('0.00')

    # ── 4) Show communal items
    print("\n🔹 Communal items (split evenly):")
    for it in communal:
        print(f" • {it['name']}  {fmt(it['price'])}")
    print(f"→ Each person chips in: {fmt(communal_share)}\n")

    # ── 5) Personal items still unclaimed
    available = [
        it for it in items
        if it.get('claimed_by') is None and not it.get('communal')
    ]
    if not available:
        print("All personal items have been claimed. Enjoy!")
        return

    print("🔹 Available personal items:")
    for idx, it in enumerate(available, 1):
        print(f" {idx}. {it['name']} — {fmt(it['price'])}")

    # ── 6) Pick them
    pick = input("\nNumbers of items you grabbed (e.g. 1,3,7): ").strip()
    try:
        indices = {int(x) - 1 for x in pick.split(',') if x.strip()}
    except ValueError:
        print("Invalid input.")
        return

    chosen = [available[i] for i in indices if 0 <= i < len(available)]

    # ── 7) Tally your personal subtotal
    subtotal = sum(Decimal(str(it['price'])) for it in chosen)

    # ── 8) Compute tax, tip, and total on (personal + communal)
    base  = subtotal + communal_share
    tax   = base * TAX_RATE
    tip   = base * TIP_RATE
    total = base + tax + tip

    # ── 9) Print your bill
    print("\n— Your bill —")
    print(f" Personal subtotal: {fmt(subtotal)}")
    print(f" (+ communal):     {fmt(communal_share)}")
    print(f" Tax  ({(TAX_RATE*100).quantize(Decimal('0.01'))}%): {fmt(tax)}")
    print(f" Tip  ({(TIP_RATE*100).quantize(Decimal('1'))}%):  {fmt(tip)}")
    print(f" Total:            {fmt(total)}\n")

    # ── 10) Mark those items as claimed
    for c in chosen:
        for orig in items:
            if orig is c:
                orig['claimed_by'] = name

    save_items(items)
    print("✅ Done—your picks are recorded. Next run will skip them.")

if __name__ == '__main__':
    main()
