# Chapter 2: Entity-Relationship (ER) Model & Relational Mapping

---

## 1. Introduction to the ER Model

* **Foundational History:** The Entity-Relationship (ER) Model was introduced by **Dr. Peter Chen** in **1976**.
* **Conceptual Abstraction:** It is a high-level conceptual data model that views the enterprise as a collection of basic real-world objects, called **Entities**, and the relationships among these objects.
* **Core Objective:** Serves as a standard, universal visual blueprint enabling database designers, programmers, and non-technical business stakeholders to model enterprise data requirements independently of any underlying physical DBMS software.

---

## 2. Entities & Entity Sets

### 2.1 Entity
* **Definition:** An entity is a "thing" or "object" in the real world that is distinct and distinguishable from all other objects based on its attributes.
* **Classification by Nature:**
  1. **Tangible Entity:** Entities that possess a physical existence in the real world (e.g., a specific student, car, teacher, pen, or computer).
  2. **Intangible Entity:** Entities that exist conceptually or logically without physical form (e.g., a bank account, university course offering, transaction, or flight reservation).

### 2.2 Entity Set
* **Definition:** A collection or set of all entities of the same type that share the same descriptive properties or attributes.
  * *Example:* The set of all persons who are students at a university forms the `STUDENT` entity set.

### 2.3 ER Model vs. Relational Model Terminology Mapping
```
+-----------------------------------------------------------------------------------------+
| ER Model Construct        | Relational Model Equivalent | Graphical Symbol in ER Model  |
+-----------------------------------------------------------------------------------------+
| Entity Set                | Table / Relation            | Single Rectangle Box          |
| Entity Instance           | Tuple / Row / Record        | A single row in a table       |
| Attribute                 | Column / Field / Header     | Ellipse / Oval Shape          |
| Attribute Value (Domain)  | Cell Data Entry             | Stored atomic data value      |
| Relationship Set          | Foreign Key / Join Table    | Diamond / Rhombus             |
+-----------------------------------------------------------------------------------------+
```

---

## 3. Comprehensive Taxonomy of Attributes

An **Attribute** represents an elementary descriptive characteristic or property possessed by members of an entity set. The set of all permitted values for an attribute is termed its **Domain**.

```
                                  ATTRIBUTES
                                      |
         +----------------------------+----------------------------+
         |                            |                            |
   Value Count                  Sub-structure                  Derivation
   +-- Single-Valued            +-- Simple (Atomic)            +-- Stored
   +-- Multivalued              +-- Composite                  +-- Derived
```

### 3.1 Single-Valued vs. Multivalued Attributes
* **Single-Valued Attribute:** Holds exactly one scalar value for a specific entity at any given time (e.g., `Aadhar_No`, `Date_of_Birth`, `Roll_No`).
  * *Symbol:* Single Ellipse.
* **Multivalued Attribute:** An entity may possess more than one value for this property at the same time (e.g., `Phone_Number` where a student owns 2 numbers, `Email_Address`, `Technical_Skills`).
  * *Symbol:* **Double Ellipse**.
  * *Relational Storage Problem & Standard Solution:*
    * Storing comma-separated lists in one column violates **First Normal Form (1NF)** atomicity.
    * Adding arbitrary columns (`Phone_1`, `Phone_2`) creates wasteful NULL fields and limits flexibility.
    * **Standard Relational Mapping:** Create a separate, dedicated table containing the Primary Key of the owner entity (as a Foreign Key) along with the multivalued attribute value:
      $$\text{Table: } \text{STUDENT\_PHONE}(\underline{\text{Student\_ID}}, \underline{\text{Phone\_Number}})$$
      Here, $(\text{Student\_ID} + \text{Phone\_Number})$ forms a Composite Primary Key.

### 3.2 Simple vs. Composite Attributes
* **Simple (Atomic) Attribute:** Cannot be partitioned further into smaller, meaningful sub-components (e.g., `Age`, `Salary`, `Roll_No`).
* **Composite Attribute:** Can be divided into smaller sub-attributes with independent semantic meaning (e.g., `Name` $\rightarrow$ `First_Name`, `Middle_Name`, `Last_Name`; `Address` $\rightarrow$ `Street`, `City`, `State`, `Pincode`).
  * *Relational Mapping:* In a relational database table, composite attributes are decomposed; each individual simple sub-component becomes a distinct table column.

### 3.3 Stored vs. Derived Attributes
* **Stored Attribute:** Permanently stored and maintained in physical database storage (e.g., `Date_of_Birth`).
* **Derived Attribute:** Not physically stored; its value is dynamically calculated on-the-fly from another stored attribute or system clock (e.g., `Age` is derived via: $\text{Current\_Date} - \text{Date\_of\_Birth}$).
  * *Symbol:* **Dashed / Dotted Ellipse**.

### 3.4 Key Attribute
* An attribute whose values uniquely identify each distinct entity within an entity set (e.g., `Student_ID`, `Employee_SSN`).
  * *Symbol:* Ellipse containing text with a **Solid Underline** (`__Student_ID__`).

### 3.5 Descriptive Attribute
* An attribute that belongs to a **Relationship Set** rather than to any single participating entity set. It records attributes of the association itself (e.g., in `EMPLOYEE Works_In DEPARTMENT`, the attribute `Since` or `Start_Date` describes the relationship).
  * *Symbol:* Ellipse connected directly to the Relationship Diamond.

```
                         +-----------------------+
                         |       [STUDENT]       |  <--- Entity Set
                         +-----------+-----------+
                                     |
    +-----------------+--------------+---------------+------------------+
    |                 |                              |                  |
( Stu_Id )    (( Stu_Phone ))                  ( Stu_Name )       ( - - - - - )
( ------ )            |                              |            (    Age    )
    ^                 v                              v            ( - - - - - )
    |          Double Ellipse                  Single Ellipse           ^
Key Attribute  (Multivalued)                      (Simple)              |
(Underline)                                                      Dashed Ellipse
                                                                    (Derived)
```

---

## 4. Relationship Sets & Degree of Relationships

A **Relationship** is an association among several entities. A **Relationship Set** is a mathematical set of relationships of the same type.

### 4.1 Essential Components of a Relationship Set
1. **Name:** A descriptive verb/action label (e.g., `Teaches`, `Advises`, `Works_For`).
2. **Degree:** The number of participating entity sets involved in the relationship.
3. **Structural Constraints:** Cardinality ratios and participation constraints.

### 4.2 Classification by Degree
1. **Unary Relationship (Degree 1 / Recursive / Self-Referential):**
   * The same entity set participates more than once in different distinct roles.
   * *Example:* In an `EMPLOYEE` table, one employee acts in the role of `Supervisor` while another acts as `Supervisee`.
2. **Binary Relationship (Degree 2):**
   * Associates two distinct entity sets. This is the most common relationship in real-world database design.
   * *Example:* `STUDENT` enrolled in `COURSE`; `TEACHER` teaches `SUBJECT`.
3. **Ternary Relationship (Degree 3):**
   * Involves three entity sets simultaneously in a single relationship diamond.
   * *Example:* `TEACHER`, `SUBJECT`, and `COURSE` linked via an assignment relationship `CST` (which teacher teaches which subject in which academic course).
4. **Quaternary Relationship (Degree 4):**
   * Associates four entity sets simultaneously.
   * *Example:* `BUYER`, `SOLICITOR`, `FINANCIAL_INSTITUTION`, and `BID` linked via `ARRANGES`.
5. **N-ary Relationship (Degree N):**
   * A general relationship associating $N$ entity sets. Complex N-ary relationships are usually converted into multiple binary relationships during implementation.

```
(a) Unary (Recursive)                    (b) Binary (Most Common)
    +-------------------+                    +----------+          +---------+
    |     EMPLOYEE      |                    | TEACHER  |----------| SUBJECT |
    +---+-----------+---+                    +----+-----+  teaches +----+----+
        |           |                             |                     |
        | supervisor| supervisee                  +---------[<>]--------+
        \     |     /
         \    v    /
          +--[<>]--+
          supervision

(c) Ternary Relationship
                   +----------+
                   |  COURSE  |
                   +----+-----+
                        |
    +----------+        |        +---------+
    | TEACHER  |-------[<>]------| SUBJECT |
    +----------+     (Assigned)  +---------+
```

---

## 5. Structural Constraints: Mapping Cardinality & Participation

### 5.1 Mapping Cardinality (Cardinality Ratios)
Expresses the maximum number of entities to which another entity can be associated via a relationship set.

1. **One-to-One (1:1):**
   * An entity in $A$ is associated with at most one entity in $B$, and an entity in $B$ is associated with at most one entity in $A$.
   * *ER Representation:* Directed arrows ($\rightarrow$) pointing to **both** entity sets ($A \leftarrow \Diamond \rightarrow B$).
   * *Example:* A Department has one Head; a Professor heads at most one Department.
2. **One-to-Many (1:N):**
   * An entity in $A$ is associated with any number of entities in $B$, but an entity in $B$ is associated with at most one entity in $A$.
   * *ER Representation:* Directed arrow ($\rightarrow$) towards the "One" side entity set, plain line to the "Many" side.
   * *Example:* An Instructor advises many students; a student has at most one advisor.
3. **Many-to-One (N:1):**
   * An entity in $A$ is associated with at most one entity in $B$, while an entity in $B$ is associated with many entities in $A$.
4. **Many-to-Many (M:N):**
   * An entity in $A$ is associated with any number of entities in $B$, and vice-versa.
   * *ER Representation:* Plain undirected lines on **both** sides ($A \text{ --- } \Diamond \text{ --- } B$).
   * *Example:* Students enroll in courses (a student takes multiple courses; a course has multiple students).

### 5.2 Participation Constraints
1. **Total Participation (Existence Dependency):**
   * Every entity in the entity set must participate in at least one relationship instance in that relationship set.
   * *ER Representation:* **Double Line ($=$)** connecting the entity rectangle to the relationship diamond.
   * *Example:* Every `LOAN` record must be associated with at least one `CUSTOMER`.
2. **Partial Participation:**
   * Some entities in the entity set may not participate in the relationship.
   * *ER Representation:* **Single Line ($-$)**.
   * *Example:* Not every `EMPLOYEE` manages a `DEPARTMENT`.

---

## 6. Strong vs. Weak Entity Sets

```
+-----------------------------------------------------------------------------------------------+
| Parameter           | Strong Entity Set                 | Weak Entity Set                     |
+-----------------------------------------------------------------------------------------------+
| Primary Key         | Possesses its own unique          | Lacks sufficient attributes to form |
|                     | Primary Key.                      | a Primary Key on its own.           |
+---------------------+-----------------------------------+-------------------------------------+
| Existence           | Independent existence in the DB.  | Dependent on an Identifying / Owner |
|                     |                                   | Strong Entity set.                  |
+---------------------+-----------------------------------+-------------------------------------+
| Key Identifier      | Key attribute (Solid Underline).  | Discriminator / Partial Key         |
|                     |                                   | (Dashed Underline).                 |
+---------------------+-----------------------------------+-------------------------------------+
| Graphical Notation  | Single Rectangle.                 | Double Rectangle.                   |
+---------------------+-----------------------------------+-------------------------------------+
| Identifying Rel.    | Standard Single Diamond.          | Double Diamond.                     |
+---------------------+-----------------------------------+-------------------------------------+
| Participation       | Can be Partial or Total.          | Always Total Participation ($=$)    |
|                     |                                   | with its identifying strong entity. |
+-----------------------------------------------------------------------------------------------+
```

### 6.1 Weak Entity Example
* A `LOAN` entity has a primary key `Loan_Number`.
* A `PAYMENT` entity tracks individual payment installments with attributes `Payment_Number` and `Payment_Amount`.
* `Payment_Number` alone cannot uniquely identify a payment globally because different loans may have payments labeled 1, 2, 3.
* Therefore, `PAYMENT` is a **Weak Entity Set**, `Payment_Number` is its **Discriminator (Partial Key)**, and `LOAN_PAYMENT` is its **Identifying Relationship**.
* In a relational table:
  $$\text{PAYMENT}(\underline{\text{Loan\_Number}}, \underline{\text{Payment\_Number}}, \text{Payment\_Amount}, \text{Date})$$

```text
  +------------------+                    ======================
  |      LOAN        |                    ||     PAYMENT      ||
  | (Strong Entity)  |                    ||  (Weak Entity)   ||
  +--------+---------+                    ==========+===========
           |                                        || (Total)
           |             //==============\\         ||
           +-------------|| LOAN_PAYMENT ||=========+
                         \\==============//
                     (Identifying Relationship)
```

---

## 7. Extended / Enhanced ER (EER) Concepts

When real-world database requirements become complex, Enhanced ER constructs provide higher-level object-oriented abstractions:

```
             +-------------------+
             |      ACCOUNT      |  <------- Superclass (Generalization)
             +---------+---------+
                       |
                     [IS-A]         ^ Generalization (Bottom-Up)
                    /     \         |
                   /       \        v Specialization (Top-Down)
        +---------+         +---------+
        | SAVING  |         | CURRENT | <--- Subclasses (Specialization)
        +---------+         +---------+
```

### 7.1 Generalization (Bottom-Up Approach)
* Multiple lower-level entity sets sharing common properties are synthesized and merged into a single higher-level generalized entity set.
* Hides lower-level differences while highlighting commonalities.
* *Example:* Merging `SAVING_ACCOUNT` and `CURRENT_ACCOUNT` into a generalized `ACCOUNT` entity set.

### 7.2 Specialization (Top-Down Approach)
* The direct inverse of Generalization.
* A higher-level entity set is broken down into one or more specialized lower-level sub-entity sets based on distinctive attributes or behaviors.
* *Example:* Specializing a `PERSON` entity set into `EMPLOYEE` and `CUSTOMER`.

### 7.3 Aggregation
* In standard ER modeling, a relationship cannot be directly connected to another relationship.
* **Aggregation** is an abstraction through which a relationship set, along with its participating entity sets, is treated as a single higher-level composite entity.
* *Example:* An employee works on a job at a specific branch:
  $$\text{Composite Unit: } [\text{EMPLOYEE} \text{ --- } \text{Works\_On} \text{ --- } \text{JOB} / \text{BRANCH}]$$
  A `MANAGER` entity evaluates and oversees this entire activity. Thus, `MANAGER` is linked via a `MANAGES` relationship to the entire aggregated unit.

```text
  +-------------------------------------------------------+
  |  AGGREGATED COMPOSITE ENTITY                          |
  |                                                       |
  |                    +----------+                       |
  |                    |   JOB    |                       |
  |                    +----+-----+                       |
  |                         |                             |
  |   +----------+     +----+-----+     +----------+      |
  |   | EMPLOYEE |-----| Works_on |-----|  BRANCH  |      |
  |   +----------+     +----+-----+     +----------+      |
  +-------------------------+-----------------------------+
                            |
                            |
                      +-----+-----+
                      |  MANAGES  |
                      +-----+-----+
                            |
                      +-----+-----+
                      |  MANAGER  |
                      +-----------+
```

---

## 8. Formal ER-to-Relational Mapping Rules

Converting an ER diagram into Relational Tables follows strict transformation algorithms:

### 8.1 Strong Entity Set
* Map directly into an independent table.
* Include all simple attributes as columns.
* For composite attributes, include only their simple sub-components as columns.
* Set the key attribute as the table's Primary Key (PK).

### 8.2 Weak Entity Set
* Map into a separate table.
* Attributes include all simple attributes of the weak entity plus the Primary Key of the identifying strong entity (as a Foreign Key).
* Primary Key of the weak table = $(\text{Strong Entity PK} + \text{Weak Entity Discriminator})$.

### 8.3 Binary Relationships by Cardinality
1. **One-to-One (1:1):**
   * Do not create a separate table.
   * Add the Primary Key of one table as a Foreign Key (FK) into the other table.
   * Priority: Place the Foreign Key into the relation that has **Total Participation** to prevent NULL values.
2. **One-to-Many (1:N):**
   * Do not create a separate table.
   * Take the Primary Key from the "One" side table and insert it as a Foreign Key (FK) into the "Many" side table.
3. **Many-to-Many (M:N):**
   * **Must create a separate relationship table.**
   * Attributes of the new table = Primary Keys of both participating entity sets (as Foreign Keys) + any descriptive attributes of the relationship.
   * The combination of both Foreign Keys forms the Composite Primary Key of this relationship table.

### 8.4 Multivalued Attribute Mapping
* Create a separate table.
* Table Schema = $(\underline{\text{Owner PK}}, \underline{\text{Multivalued\_Attribute}})$.
* Both attributes together form the Composite Primary Key.

---

## 9. Comprehensive Case Studies

### 9.1 Banking Management System
* **Entities & Attributes:**
  * `BANK` ($\underline{\text{Code}}$, $\text{Name}$, $\text{Address}$)
  * `BRANCH` ($\underline{\text{Branch\_ID}}$, $\text{Name}$, $\text{Address}$)
  * `CUSTOMER` ($\underline{\text{Cust\_ID}}$, $\text{Name}$, $\text{Phone}$, $\text{Address}$)
  * `ACCOUNT` ($\underline{\text{Account\_No}}$, $\text{Acc\_Type}$, $\text{Balance}$)
  * `LOAN` ($\underline{\text{Loan\_ID}}$, $\text{Loan\_Type}$, $\text{Amount}$)
* **Cardinalities:**
  * `BANK` has `BRANCH` ($1 : N$)
  * `BRANCH` offers `LOAN` ($1 : N$)
  * `BRANCH` maintains `ACCOUNT` ($1 : N$)
  * `CUSTOMER` borrows `LOAN` ($N : M$)
  * `CUSTOMER` holds `ACCOUNT` ($N : M$)

### 9.2 University Management System
* **Entities:** `PROFESSOR`, `DEPARTMENT`, `UNIVERSITY`, `COURSE`, `STUDENT`.
* **Key Relationships:**
  * `PROFESSOR` belongs to `DEPARTMENT` ($N : 1$).
  * `DEPARTMENT` offers `COURSE` ($1 : N$).
  * `STUDENT` enrolls in `COURSE` ($M : N$) with descriptive attribute `Grade`.

---

## 10. Advantages & Limitations of the ER Model

### 10.1 Advantages
1. **Simplicity & Intuitive Visualization:** Graphic notations make high-level enterprise architectures easy to communicate across technical and non-technical teams.
2. **Direct Relational Conversion:** ER constructs map smoothly and algorithmically into relational schemas.
3. **Effective Requirements Engineering:** Serves as a clear conceptual checklist during the early database design phase.

### 10.2 Limitations
1. **Semantic Loss of Information:** Certain fine-grained operational logic and check constraints cannot be visually captured in graphical ER notation.
2. **Limited Constraint Modeling:** Complex functional inter-dependencies and temporal constraints cannot be represented.
3. **Visual Overhead:** Becomes cluttered, unreadable, and overly complex for small or highly interconnected systems.

---

## 11. Bihar Engineering University (BEU) Exam Section

### 11.1 Frequently Asked University Questions

#### Short Answer Questions (5 Marks)
1. Differentiate between Single-Valued and Multivalued attributes with an example. How is a multivalued attribute mapped into a relational table?
2. Explain the concept of a Recursive (Unary) Relationship with an ER diagram.
3. What is a Descriptive Attribute in an ER diagram? Where is it placed?
4. Differentiate between Generalization and Specialization.

#### Long Answer Questions (7 to 14 Marks)
1. **Define a Weak Entity Set. Explain how a weak entity set is represented in an ER diagram and converted into relational tables. Illustrate using a Loan-Payment example.** *(Frequently Asked - 14 Marks)*
2. **Explain the step-by-step methodology to convert an ER diagram into Relational Tables. Cover 1:1, 1:N, M:N relationships and multivalued attributes.** *(Frequently Asked - 14 Marks)*
3. **Draw a complete, well-labeled Entity-Relationship diagram for a Banking Enterprise / University Management System. Identify all entities, attributes, primary keys, and relationship cardinalities.** *(Frequently Asked - 14 Marks)*
4. **Write short notes on:**
   * (a) Aggregation in EER modeling
   * (b) Total vs. Partial Participation
   * (c) Structural Constraints of a Relationship *(7/14 Marks)*

---

### 11.2 Model Exam Answer: Strong vs. Weak Entity (BEU Presentation Standard)

**Question:** Define Weak Entity Set. How does it differ from a Strong Entity Set? Explain its representation and relational schema conversion with an example. *(7 Marks)*

**Model Answer:**

**1. Definition:**  
A Weak Entity Set is an entity set that does not possess sufficient attributes to form a primary key on its own. It depends on an **Identifying (Owner) Strong Entity Set** for its existence. Its records are distinguished using a **Partial Key (Discriminator)**.

**2. Comparison Table:**  
*(Include the comparison table from Section 6 of this document).*

**3. Diagrammatic Representation:**
* Weak Entity: **Double Rectangle**.
* Identifying Relationship: **Double Diamond**.
* Partial Key: **Dashed Underline**.
* Participation: **Double Line** (Total participation with owner).

*(Draw the Loan-Payment diagram from Section 6.1).*

**4. Relational Schema Conversion:**  
To convert a weak entity into a relation:
1. Create a table for the weak entity.
2. Include all simple attributes of the weak entity.
3. Add the Primary Key of the identifying strong entity as a Foreign Key.
4. Composite Primary Key = $(\text{Strong PK} + \text{Discriminator})$.

$$\text{LOAN}(\underline{\text{Loan\_No}}, \text{Amount})$$
$$\text{PAYMENT}(\underline{\text{Loan\_No}}, \underline{\text{Payment\_No}}, \text{Date}, \text{Amount})$$

---

### 11.3 Important Multiple-Choice Questions (MCQs)

1. **Who proposed the Entity-Relationship (ER) model in 1976?**
   * (A) E. F. Codd
   * (B) Peter Chen
   * (C) James Martin
   * (D) C. J. Date  
   * **Correct Answer:** **(B) Peter Chen**

2. **In an ER diagram, a Weak Entity Set is represented using a:**
   * (A) Dashed rectangle
   * (B) Double rectangle
   * (C) Inverted triangle
   * (D) Shaded oval  
   * **Correct Answer:** **(B) Double rectangle**

3. **The attribute discriminator of a weak entity set is denoted by:**
   * (A) Solid underline
   * (B) Dashed / dotted underline
   * (C) Double ellipse
   * (D) Bold letters  
   * **Correct Answer:** **(B) Dashed / dotted underline**

4. **In an ER diagram, an attribute connected to a Relationship diamond is called a:**
   * (A) Derived attribute
   * (B) Descriptive attribute
   * (C) Multivalued attribute
   * (D) Key attribute  
   * **Correct Answer:** **(B) Descriptive attribute**

5. **When converting an $M:N$ (Many-to-Many) binary relationship into a relational model, how many new tables are created specifically for the relationship?**
   * (A) Zero (merged using Foreign Keys)
   * (B) Exactly one new table
   * (C) Two new tables
   * (D) Three new tables  
   * **Correct Answer:** **(B) Exactly one new table**

6. **The bottom-up design process of synthesizing multiple lower-level entity sets into a single higher-level entity set is termed:**
   * (A) Specialization
   * (B) Generalization
   * (C) Aggregation
   * (D) Decomposition  
   * **Correct Answer:** **(B) Generalization**

7. **A relationship involving only one entity set playing multiple roles is classified as:**
   * (A) Binary
   * (B) Ternary
   * (C) Unary / Recursive
   * (D) Total  
   * **Correct Answer:** **(C) Unary / Recursive**