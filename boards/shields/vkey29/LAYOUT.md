# VKey29 ZMK Layout Guide

## How the layout works

### 1. **Hardware: `.dtsi` and `.overlay`**

- **`vkey29.dtsi`** (and **`vkey29_left.overlay / vkey29_right.overlay`**) define the **physical matrix**:
  - **5 rows** × **7 columns** = **35 switch positions** per half.
  - Rows are driven (e.g. “num, top, home, bottom, thumb”).
  - Columns are read.
  - No **matrix transform** is used, so keymap bindings follow **raw kscan order**.

### 2. **Keymap: `.keymap`**

- **`config/keymap/vkey29.keymap`** defines **what each key does** (behaviors).
- **Rule:** You must have **exactly one binding per matrix position**, in **keyboard scan order**.
- For `kscan-gpio-matrix` without a transform, that order is **row-major**:
  - Row 0: positions 0–6 (col 0 → col 6)
  - Row 1: positions 7–13
  - Row 2: positions 14–20
  - Row 3: positions 21–27
  - Row 4: positions 28–34

So you need **35 bindings per half** (70 total for split central) in each layer (and the same count for any other layer, or use `&trans` for “same as below”).

### 3. **Binding order (physical layout)**

```
Matrix (5×7) – each cell is one binding index:

       Col0   Col1   Col2   Col3   Col4   Col5   Col6
Row0    0      1      2      3      4      5      6
Row1    7      8      9     10     11     12     13
Row2   14     15     16     17     18     19     20
Row3   21     22     23     24     25     26     27
Row4   28     29     30     31     32     33     34
```

- Match this to **your physical keyboard**: which row/column has which key (e.g. thumb, home row, number row).
- If a position has **no key** on your PCB, use **`&trans`** (transparent) or **`&none`** (no action).

### 4. **Common behaviors**

| Behavior   | Example   | Meaning                    |
|-----------|-----------|----------------------------|
| Key press | `&kp A`   | Sends keycode (e.g. A)     |
| Modifier  | `&kp LSHFT` | Left Shift               |
| Layer     | `&mo 1`   | Hold = layer 1             |
| Transparent | `&trans` | Use key from layer below   |
| No op     | `&none`   | Key does nothing           |

See [ZMK Keymap Behaviors](https://zmk.dev/docs/keymaps) for more.

---

## How to set your layout

1. **Know your physical layout**  
   Which key is at which row/column (e.g. from a schematic or PCB silkscreen). If you have a “VKey29” or “VRanger24” layout diagram, use it.

2. **Edit `config/keymap/vkey29.keymap`**  
   - For split central: **70 entries** per layer (35 left + 35 right).
   - Put the behavior for **position 0** first, then **1**, … then **69** (left 0–34, right 35–69).
   - For a position with no key, use `&trans`.

3. **Optional: matrix transform**  
   If you want a **logical** layout with fewer than 35 keys (e.g. only 24), you can add a **matrix transform** in the `.dtsi` or `.overlay` that maps logical positions to physical `RC(row, col)`. Then the keymap has one binding per **logical** key, and you don’t need a binding for every empty matrix position. See [ZMK Layout Configuration](https://zmk.dev/docs/config/layout).

4. **Build and flash**  
   Build your firmware and flash both halves (if split). Key positions that feel wrong usually mean the binding order doesn’t match your physical row/column layout—adjust the order in `bindings` accordingly.

---

## File roles (summary)

| File                     | Role |
|--------------------------|------|
| `Kconfig.shield`         | Enables the shield (e.g. for build menu). |
| `vkey29.dtsi`            | Base hardware: kscan, I2C, OLED. |
| `vkey29_left.overlay`    | Left half: row/col GPIOs. |
| `vkey29_right.overlay`   | Right half: row/col GPIOs. |
| `config/keymap/vkey29.keymap` | **Your keymap:** one binding per key in kscan order (70 for split central). |

Your layout is defined **only** in the keymap (and optionally by a matrix transform); the `.dtsi`/`.overlay` only define the matrix size and pins.
