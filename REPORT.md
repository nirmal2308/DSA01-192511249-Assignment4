# ASSIGNMENT 4 REPORT: DESIGNING REUSABLE CLASS HIERARCHIES

**Course:** Object Oriented Programming with C++ (DSA01 - CO4)  
**Assignment Title:** Analyse and Implement C++ Applications Using Inheritance, Virtual Base Classes, Abstract Classes, Pointers and Polymorphism  
**Student Name:** Nirmal S  
**Register Number:** 192511249  
**Nature of Assignment:** Individual  
**Total Marks:** 50  
**GitHub Repository Link:** https://github.com/nirmal2308/DSA01-192511249-Assignment4  
**Bloom’s Level Assessed:** K5 – Evaluate  
**Compiler Used:** `g++ (Ubuntu 15.2.0-16ubuntu1) 15.2.0`  
**Standard & Compilation Flags:** `g++ -std=c++17 -Wall -Wextra -fsanitize=address`  

---

## EXECUTIVE SUMMARY

Assignment 4 examines the architectural principles required to construct reusable, extensible, and type-safe object-oriented class hierarchies in C++. Key topics evaluated include:
1. Identifying and refactoring duplicated procedural-style classes into unified abstract hierarchies.
2. Resolving the multiple-inheritance Diamond Problem via virtual base classes.
3. Order of construction and destruction across inheritance and member composition boundaries.
4. Forensic investigation of polymorphic pointer defects (early binding, missing virtual destructors, and object slicing).
5. Fluent method chaining, member disambiguation, and object indirection utilizing the `this` pointer.
6. Synthesizing an Open-Closed extensible vehicle rental fleet architecture capable of absorbing new vehicle categories without modifying existing modules.

---

## TASK 1 – CLASS DESIGN REVIEW: DIAGNOSE AND REPAIR DUPLICATED CLASSES (6 MARKS)

### 1.1 Task 1.1: Diagnosis of Five Design Deficiencies
The legacy payroll code maintained three disparate classes (`Professor`, `LabAssistant`, `GuestFaculty`):
1. **Repeated Fields & Logic Duplication:** Both `Professor` and `LabAssistant` maintain identical fields (`name`, `id`, `basic`, `da`) and duplicate the pay calculation rule (`basic + basic * 0.42`).
2. **Breach of Encapsulation (Public Data):** All fields are declared `public`, allowing external functions to bypass business rules and corrupt payroll balances directly.
3. **Inconsistent Naming Conventions:** The same conceptual entities use conflicting identifiers (`name`/`id` in Professor vs `nm`/`code` in GuestFaculty).
4. **Dead Code / Unused Fields:** The `da` field is declared in `Professor` and `LabAssistant` but completely ignored in the calculation in favor of the hardcoded literal `0.42`.
5. **Lack of Common Polymorphic Abstraction:** Without a shared base class, payroll processing cannot be performed in a single loop; separate arrays, loops, and sum variables are required for every staff type.

### 1.2 Task 1.2: Repaired Class Hierarchy & UML
The repaired design in `Task01/task01.cpp` introduces:
- Abstract base class `Staff` with protected attributes (`name`, `id`), a pure virtual function `virtual double pay() const = 0;`, and a virtual destructor `virtual ~Staff() = default;`.
- Intermediate class `RegularStaff : public Staff` encapsulating common permanent employee state (`basic`, constant `DA_RATE = 0.42`).
- Derived classes `Professor`, `LabAssistant`, and `GuestFaculty` overriding `pay()`.

```
               +-----------------------+
               |        Staff          |  <<Abstract>>
               +-----------------------+
               | # name: string        |
               | # id: int             |
               +-----------------------+
               | + virtual pay(): double = 0
               +-----------------------+
                           ^
             +-------------+-------------+
             |                           |
   +--------------------+      +--------------------+
   |    RegularStaff    |      |    GuestFaculty    |
   +--------------------+      +--------------------+
   | # basic: double    |      | - hours: double    |
   | - DA_RATE = 0.42   |      | - hourlyRate: dbl  |
   +--------------------+      +--------------------+
             ^
       +-----+-----+
       |           |
+-------------+ +----------------+
|  Professor  | |  LabAssistant  |
+-------------+ +----------------+
```

### 1.3 Task 1.3: Design Defense
By implementing an abstract `Staff` base class, introducing a new staff category (e.g. `ResearchAssociate` or `VisitingFellow`) requires creating a new derived class overriding `pay()`. The central payroll engine processes all records via `Staff*` without modifying or recompiling any existing employee classes, strictly satisfying the Open-Closed Principle.

### 1.4 Alternative Concept: Type-Code with `switch`
- If implemented with a single `Staff` class containing an enum `StaffType` and a switch-statement:
  - Time is $O(1)$ in both designs.
  - However, every instance must allocate space for *all* potential fields (`basic`, `da`, `research`, `hours`, `rate`), causing severe memory bloat.
  - Adding a new staff category forces modifying the core `switch` statement in every payroll file, creating high regression risk.

---

## TASK 2 – FROM REQUIREMENT TO HIERARCHY (8 MARKS)

### 2.1 Task 2.1: Relationship Classification & Inheritance Types
- `Student is-a Person`: Public Single Inheritance.
- `Staff is-a Person`: Public Single Inheritance.
- `TeachingAssistant is-a Student, is-a Staff`: Multiple & Hybrid Inheritance.
- `ResearchScholar is-a Student`: Multilevel Inheritance (`Person` $\rightarrow$ `Student` $\rightarrow$ `ResearchScholar`).
- `Address has-a in Person`: Composition (`has-a` relationship).

### 2.2 Task 2.2: Resolving the Diamond Problem via Virtual Base Classes
When `TeachingAssistant` inherits from both `Student` and `Staff`, both intermediate classes inherit from `Person`. Without virtual inheritance:
- `TeachingAssistant` contains **two separate sub-objects of `Person`**, duplicating `name`, `id`, and `phone`.
- Member access `ta.name` triggers compiler ambiguity: `request for member 'name' is ambiguous`.
- **Solution:** Declaring `class Student : virtual public Person` and `class Staff : virtual public Person` ensures that only **one shared `Person` sub-object** exists in `TeachingAssistant`.

### 2.3 Task 2.3: Predict the Output Analysis
```cpp
struct Person { Person() { std::cout << "Person "; } };
struct Student : virtual Person { Student() { std::cout << "Student "; } };
struct Staff : virtual Person { Staff() { std::cout << "Staff "; } };
struct TeachingAssistant : Student, Staff {
    TeachingAssistant() { std::cout << "TA "; }
};
```
- **Predicted Output:** `Person Student Staff TA `
- **Actual Output:** `Person Student Staff TA `
- **Explanation:** In virtual inheritance, the most-derived class (`TeachingAssistant`) is uniquely responsible for constructing the virtual base. The virtual base constructor `Person()` is executed exactly once before any non-virtual base classes.

### 2.4 Task 2.4: Access Specifier Visibility Table
| Base Member Visibility | Public Inheritance | Protected Inheritance | Private Inheritance |
| :--- | :--- | :--- | :--- |
| **`public:`** | `public` in derived | `protected` in derived | `private` in derived |
| **`protected:`** | `protected` in derived | `protected` in derived | `private` in derived |
| **`private:`** | Inaccessible directly | Inaccessible directly | Inaccessible directly |

---

## TASK 3 – CONSTRUCTORS IN DERIVED CLASSES AND MEMBER CLASSES (9 MARKS)

### 3.1 Task 3.1 & 3.2: Construction Order Principles
Construction and destruction follow strict architectural rules:
1. Virtual base classes are constructed first in depth-first left-to-right order.
2. Non-virtual direct base classes are constructed next in declaration order.
3. Member objects (composition) are constructed next in their order of declaration in the class.
4. The derived class constructor body executes last.
5. **Destruction occurs in exact reverse order.**

### 3.2 Predict the Output: Vehicle / Engine / Car Verification
```cpp
struct Car : Vehicle { Engine e; ... };
```
- Prediction: `Vehicle Engine Car | ~Car ~Engine ~Vehicle `
- Actual Output: `Vehicle Engine Car | ~Car ~Engine ~Vehicle `
- Verification: Base `Vehicle` constructs first, member `Engine e` constructs second, and `Car` body completes third. At block exit, `~Car` destructs first, `~Engine` second, and `~Vehicle` third.

### 3.3 Task 3.3: Complete the Code Verification
Executing `Task03/task03.cpp` with `Student s("Kavya", 19, 192311001, "Salem", 636001)`:
```text
192311001 Kavya (19) Salem-636001
```
The member initializer list `: Person(n, a), regNo(r), home(c, p)` cleanly initializes both the inherited base and the composed `Address` member.

---

## TASK 4 – POINTERS TO DERIVED CLASSES: FORENSICS (7 MARKS)

### 4.1 Forensic Analysis of Defects
1. **Defect 1 (Early Binding of `rate()`):** `Account::rate()` was declared non-virtual. When invoked via `Account* a`, the compiler binds the call at compile time to the static type (`Account`), returning 3.5% instead of the senior citizen 4.0%.
2. **Defect 2 (Missing Virtual Destructor / Memory Leak):** `~Account()` was non-virtual. Deleting derived instances through `Account*` only invoked `~Account()`; `~SeniorAccount()` was bypassed, leaking the 100-element `double* history` array.
3. **Defect 3 (Object Slicing):** In `Account copy = SeniorAccount();`, copying a derived object into a value-type base variable strips away derived attributes (`history`) and resets the virtual table pointer to `Account`.

### 4.2 Predict the Output Verification
- Prediction: `3.5 4 3.5`
- Actual Output: `3.5 4 3.5`
- Explanation:
  - `p->rate()` is non-virtual $\rightarrow$ early bound to `Account::rate()` = `3.5`.
  - `p->vrate()` is virtual $\rightarrow$ dynamic dispatch to `SeniorAccount::vrate()` = `4.0`.
  - `copy.vrate()` is invoked on a sliced `Account` object $\rightarrow$ calls `Account::vrate()` = `3.5`.

### 4.3 Prevention Rules
1. Any base class with at least one virtual function **must define a public virtual destructor** (`virtual ~Base() = default;`).
2. Always specify `override` on derived virtual functions to catch signature mismatches at compile time.
3. Store polymorphic objects via pointers (`std::unique_ptr<Base>`) or references, never by value.

---

## TASK 5 – POINTERS TO OBJECTS AND THE THIS POINTER (7 MARKS)

### 5.1 Method Chaining via `return *this;`
In `Task05/task05.cpp`, `add()` and `setDiscount()` return `Cart&` (lvalue reference to `*this`). This enables fluent method chaining:
```cpp
c.add("Rice", 60).add("Dal", 120).setDiscount(10);
```
Expected output verified: `324.00`.

### 5.2 Four Ways to Reach an Object
| Indirection Technique | Syntax | Lifetime Control | Nullability Risk |
| :--- | :--- | :--- | :--- |
| **(a) Direct Name** | `c.total()` | Scope-bound (automatic stack lifetime) | None (always valid) |
| **(b) Raw Pointer** | `p->total()` | Controlled by pointee lifetime | High (can be `nullptr` or dangling) |
| **(c) Reference** | `ref.total()` | Alias to existing object | Low (cannot be null, but can dangle) |
| **(d) Heap Pointer** | `heap->total()` | Dynamic (manual `delete` / smart pointer)| High if manual; zero leak with `unique_ptr` |

### 5.3 Nature of the `this` Pointer
- `this` is an implicit `const` pointer (`ClassName* const`) passed as a hidden first argument to all non-static member functions, holding the memory address of the invoking instance.
- `*this` dereferences the pointer to yield the actual object lvalue.

---

## TASK 6 – DESIGN CHALLENGE: VEHICLE RENTAL FLEET (13 MARKS)

### Stage 1 – Requirements & Entities
- Entities: `Bicycle`, `TwoWheeler`, `Car`, `ElectricCar`, and newly added `ElectricBus`.
- Differing Behaviors:
  - Bicycles: daily rate + $1.5\times$ late penalty.
  - Two-Wheelers: base rate + helmet fee (Rs 50) + $1.5\times$ late penalty.
  - Cars: base rate + insurance fee (Rs 200) + $2.0\times$ late penalty.
  - Electric Cars: 10% green discount + battery kWh rate (Rs 15/kWh) + $2.0\times$ late penalty.
  - Electric Bus: commercial rate + battery kWh rate (Rs 20/kWh) + $2.5\times$ late penalty.

### Stage 2 & 3 – Class Hierarchy & Storage Architecture
- Abstract base class `Vehicle` with pure virtual functions `getCategory()` and `calculateCharge(days, lateDays)`.
- Fleet storage: `std::vector<std::unique_ptr<Vehicle>>` provides homogeneous storage of heterogeneous objects with automatic zero-leak memory management.

### Stage 4 & 5 – Implementation & Verification
Executing `Task06/task06.cpp`:
```text
Rented Bicycle [TN-01-B101] successfully.
Rented Two-Wheeler [TN-02-TW202] successfully.
Rented Car [TN-03-C303] successfully.
Rented Electric Car [TN-04-EC404] successfully.
Rented Electric Bus [TN-05-EB505] successfully.

Returned Bicycle (3 days on-time): Total Bill = Rs 450.00
Returned Car (4 days + 2 days late): Total Bill = Rs 17800.00
Returned Electric Car (2 days + 1 day late): Total Bill = Rs 11240.00
Returned NEW Electric Bus (5 days on-time): Total Bill = Rs 47000.00
```
- **Open-Closed Principle Proved:** The new `ElectricBus` was added seamlessly without altering or recompiling `Vehicle`, `Bicycle`, `TwoWheeler`, or `Car`.

---

## CONCLUSION

Assignment 4 confirms the critical role of virtual base classes, virtual destructors, and pure virtual polymorphic interfaces in eliminating duplication, solving multiple inheritance diamond ambiguities, and upholding the Open-Closed Principle.
