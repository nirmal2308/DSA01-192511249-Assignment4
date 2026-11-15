# VIVA & CONCEPT GUIDE: REUSABLE CLASS HIERARCHIES
**Student:** Nirmal S (Reg No: 192511249)  
**Perspective:** Python Master to C++ Beginner Bridge  
**Goal:** Crack the viva and confidently explain/modify any part of Assignment 4 on the spot.

---

## 1. PYTHON VS C++ "CHEAT SHEET" (YOUR MENTAL BRIDGE)

| Concept | Python Mental Model | C++ Reality in Assignment 4 | Why Faculty Cares |
| :--- | :--- | :--- | :--- |
| **Inheritance Syntax** | `class Derived(Base):` | `class Derived : public Base` | Access specifier controls base member visibility in derived class. |
| **Diamond Problem** | Solved automatically via C3 Linearization (MRO). | C++ creates two copies of base sub-object by default! Requires `virtual public Base`. | Core of Task 02: University People Management. |
| **Virtual Functions** | All methods in Python are polymorphic/virtual by default. | Methods are non-virtual (early bound) by default! Must declare `virtual` for runtime dynamic dispatch. | Core of Task 04: Why Senior Citizen rate 4% wasn't called. |
| **Virtual Destructor** | Garbage collector cleans up derived attributes. | If deleting derived object via base pointer, base destructor MUST be `virtual`, else derived destructor is bypassed! | Task 04: Preventing memory leaks in polymorphic hierarchies. |
| **Object Slicing** | Variables are references; slicing cannot occur. | Assigning derived object to a value-type base variable physically slices off derived attributes! | Task 04: `Account copy = SeniorAccount()`. |
| **The `this` Pointer** | Explicit `self` passed as first argument. | Implicit `this` pointer passed secretly to non-static member functions. | Task 05: Method chaining via `return *this;`. |
| **Abstract Classes** | Subclassing `abc.ABC` with `@abstractmethod`. | Class with at least one pure virtual function: `virtual void f() = 0;`. | Task 01 & 06: `Staff` and `Vehicle` interfaces. |

---

## 2. TASK-BY-TASK VIVA CHEAT SHEET

### TASK 01: Class Design Review (`Task01/task01.cpp`)
- **What it does:** Refactors duplicated `Professor`, `LabAssistant`, `GuestFaculty` into an abstract `Staff` hierarchy.
- **Viva Answer:** *"The original design suffered from repeated pay calculation logic, public variables, and inability to process staff in a single loop. We unified them under abstract base class `Staff` with pure virtual `pay()`."*

---

### TASK 02: Diamond Problem Resolution (`Task02/task02.cpp`)
- **What it does:** Resolves diamond inheritance in `TeachingAssistant` inheriting from `Student` and `Staff` via `virtual public Person`. Verifies prediction `Person Student Staff TA `.
- **Viva Answer:** *"Without virtual inheritance, TeachingAssistant would contain two Person sub-objects, duplicating name and ID and causing ambiguity. Virtual inheritance guarantees exactly one shared Person sub-object constructed by the most-derived class."*

---

### TASK 03: Construction & Destruction Order (`Task03/task03.cpp`)
- **What it does:** Verifies prediction `Vehicle Engine Car | ~Car ~Engine ~Vehicle ` and derived constructor composition with `Address`.
- **Viva Answer:** *"Base classes are constructed first, then member objects in declaration order, then the derived class constructor body. Destruction occurs in exact reverse order."*

---

### TASK 04: Polymorphic Forensics & Slicing (`Task04/task04.cpp`)
- **What it does:** Investigates non-virtual method dispatch, missing virtual destructor leaks, and object slicing.
- **Viva Answer:** *"Non-virtual methods bind at compile time based on pointer type rather than the actual object. Deleting a derived object via a non-virtual base pointer skips the derived destructor, leaking memory. Object slicing occurs when assigning derived objects by value to base variables."*

---

### TASK 05: The `this` Pointer & Chaining (`Task05/task05.cpp`)
- **What it does:** Chaining in `Cart` via `return *this;`, distinguishes member variables (`this->discount = discount;`).
- **Viva Answer:** *"The `this` pointer holds the address of the current instance. Returning `*this` by reference enables fluent method chaining (`c.add().add().setDiscount()`)."*

---

### TASK 06: Vehicle Rental Fleet (`Task06/task06.cpp`)
- **What it does:** Polymorphic vehicle rental fleet with abstract base `Vehicle` and adds `ElectricBus` without modifying existing classes.
- **Viva Answer:** *"We designed the fleet following the Open-Closed Principle: classes are open for extension but closed for modification. We added the new ElectricBus class without changing a single line of existing code."*
