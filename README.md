# Hybrid Inventory Manager

A console-based inventory management application where the **data layer is written in C** (structs + binary file storage) and the **menu/UI layer is written in C++** (classes + STL). Data persists across restarts via a binary file (`inventory.dat`).

---

## File Structure

```
inventory_project/
├── include/
│   ├── inventory.h          # C struct + function declarations (extern "C")
│   └── InventoryManager.h   # C++ class declaration
├── src/
│   ├── inventory.c          # C backend: fread/fwrite/fseek binary storage
│   ├── InventoryManager.cpp # C++ UI layer: STL vector, sort, formatted output
│   └── main.cpp             # Entry point: menu loop
├── Makefile
├── CMakeLists.txt
└── README.md
```

---

## Build & Run Steps

### Using Make (recommended)

```bash
# Build
make

# Run
./inventory_manager

# Or build + run in one step
make run

# Clean build artifacts
make clean

# Clean everything (including inventory.dat)
make cleanall
```

### Using CMake

```bash
mkdir build_cmake && cd build_cmake
cmake ..
make
./inventory_manager
```

---

## Menu Options

| Option | Action |
|--------|--------|
| `1`    | Add a new item (prompts for ID, name, quantity, price) |
| `2`    | View an item by ID |
| `3`    | Update an existing item by ID |
| `4`    | Soft-delete an item by ID |
| `5`    | List all active items sorted by ID |
| `5n`   | List all active items sorted by name |
| `6`    | Exit |

---

## Test Cases

### 1. Add 3 items, exit, restart, and verify they persist

```
Choice: 1  → ID=1, Name=Widget, Qty=10, Price=5.99
Choice: 1  → ID=2, Name=Gadget, Qty=3,  Price=19.99
Choice: 1  → ID=3, Name=Doohickey, Qty=7, Price=2.50
Choice: 6  → exit

# Restart the program
Choice: 5  → All 3 items appear  ✓
```

### 2. Update an item; verify after restart

```
Choice: 3  → ID=2, new Qty=50, new Price=24.99
Choice: 6  → exit

# Restart
Choice: 2  → ID=2 shows updated values  ✓
```

### 3. Deleted items do not appear in List or View

```
Choice: 4  → ID=3
Choice: 5  → Only ID=1 and ID=2 appear  ✓
Choice: 2  → ID=3 → "not found (or deleted)"  ✓
```

### 4. Duplicate IDs are rejected

```
Choice: 1  → ID=1, Name=Duplicate ...
             → "[!] Failed: duplicate active ID or file error."  ✓
```

### 5. Invalid inputs are re-asked (no crash)

```
Choice: 1  → ID=-5  → re-ask
             ID=abc  → re-ask
             ID=4 (valid) ...
             Qty=-1  → re-ask
             Qty=0   → accepted  ✓
```

---

## Design Notes

- **C layer** (`inventory.c`): pure C11, uses `fread`/`fwrite`/`fseek` for direct binary record access. Soft deletes set `is_deleted=1`; records are never physically removed so existing IDs are stable.
- **C++ layer** (`InventoryManager.cpp`): C++17, uses `std::vector` to load records and `std::sort` with lambda comparators to sort by ID or name before printing.
- **Linking**: both object files link into one executable; `extern "C"` in the header ensures correct name mangling.
