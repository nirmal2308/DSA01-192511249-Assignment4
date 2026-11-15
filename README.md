# DSA01 - Assignment 4: Designing Reusable Class Hierarchies

**Course:** Object Oriented Programming with C++ / Data Structures and Algorithms (CO4)  
**Student Name:** Nirmal S  
**Register Number:** 192511249  
**Nature of Assignment:** Individual  
**Total Marks:** 50 (Bloom's Level: K5 – Evaluate)  
**GitHub Repository Link:** https://github.com/nirmal2308/DSA01-192511249-Assignment4  

---

## 1. System and Compiler Information

- **Operating System:** Linux (Ubuntu 24.04 LTS x86_64 via WSL)
- **Compiler:** `g++` (Ubuntu 15.2.0-16ubuntu1) 15.2.0
- **C++ Standard:** C++17 (`-std=c++17`)
- **Sanitizers & Flags:** `-std=c++17 -Wall -Wextra -fsanitize=address`

---

## 2. Directory Structure

```text
DSA01-192511249-Assignment4/
├── .gitignore
├── README.md
├── REPORT.md
├── VIVA_NOTES.md
├── Task01/
│   ├── task01.cpp
│   └── output.txt
├── Task02/
│   ├── task02.cpp
│   └── output.txt
├── Task03/
│   ├── task03.cpp
│   └── output.txt
├── Task04/
│   ├── task04.cpp
│   └── output.txt
├── Task05/
│   ├── task05.cpp
│   └── output.txt
└── Task06/
    ├── task06.cpp
    └── output.txt
```

---

## 3. Compilation and Execution Commands

Run the following commands from repository root:

### Task 01: Class Design Review - Payroll Hierarchy
```bash
g++ -std=c++17 -Wall -Wextra Task01/task01.cpp -o Task01/task01
./Task01/task01
```

### Task 02: University People Management & Diamond Resolution
```bash
g++ -std=c++17 -Wall -Wextra Task02/task02.cpp -o Task02/task02
./Task02/task02
```

### Task 03: Constructors in Derived Classes and Composition
```bash
g++ -std=c++17 -Wall -Wextra Task03/task03.cpp -o Task03/task03
./Task03/task03
```

### Task 04: Pointers to Derived Classes - Forensics & Repair
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task04/task04.cpp -o Task04/task04
./Task04/task04
```

### Task 05: Pointers to Objects and the `this` Pointer
```bash
g++ -std=c++17 -Wall -Wextra Task05/task05.cpp -o Task05/task05
./Task05/task05
```

### Task 06: Design Challenge - Vehicle Rental Fleet
```bash
g++ -std=c++17 -Wall -Wextra Task06/task06.cpp -o Task06/task06
./Task06/task06
```

---

## 4. Summary of Tasks and Key Findings

| Task | Topic | Key Findings & Evaluation |
| :--- | :--- | :--- |
| **Task 01** | Class Design Review | Diagnosed 5 defects in duplicated payroll classes; refactored to an abstract `Staff` base class with polymorphic `pay()`, enabling unified single-loop processing. |
| **Task 02** | Diamond Problem Resolution | Solved ambiguity in `TeachingAssistant` inheriting from `Student` and `Staff` using `virtual public Person`; confirmed single `Person` construction. |
| **Task 03** | Construction & Destruction Order | Verified that base classes construct first, then member objects, then derived class bodies (with inverse destruction order); integrated `Address` composition. |
| **Task 04** | Polymorphic Pointer Forensics | Identified early binding bugs, virtual destructor absence, and object slicing; repaired using `virtual` overrides and `std::unique_ptr` without memory leaks. |
| **Task 05** | Pointers to Objects & `this` | Chained method calls in shopping `Cart` via `return *this;`, resolved parameter shadow using `this->`, and searched an array of pointers. |
| **Task 06** | Design Challenge | Designed extensible Vehicle Rental Fleet obeying the Open-Closed Principle, proving that a new `ElectricBus` can be added without modifying existing code. |
