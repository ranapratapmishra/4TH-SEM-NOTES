# Chapter 3: Relational Model Basics, Keys & Functional Dependency

---

## 1. Fundamentals of the Relational Model (RDBMS)

* **Historical Inception:** The Relational Database Management System (RDBMS) was conceptualized by **Dr. Edgar Frank Codd (E. F. Codd)** at IBM in **1970**.
* **Foundational Philosophy:** Data is abstracted and stored in structured, two-dimensional tabular representations termed **Relations**, where relationships between data entities are maintained via shared data values rather than physical navigational pointers.

### 1.1 Core Relational Terminologies
* **Table (Relation):** A mathematical set of unordered tuples (rows) representing entity instances and business transactions.
* **Tuple (Row / Record):** A single horizontal instance of data representing a distinct entity within the relation.
* **Attribute (Column / Field):** A named vertical property describing a specific trait of the relation (e.g., `NAME`, `ID`, `CITY`).
* **Domain:** A pool of permissible, atomic values governed by a declared data type from which attribute values are drawn.
  * **Atomicity:** Every domain value is strictly indivisible with respect to relational operators.
* **Degree / Arity:** The total count of columns/attributes comprising a relation schema:
  $$\text{Degree (Arity)} = \text{Total Number of Attributes}$$
* **Cardinality:** The total count of tuples/rows contained in a relation instance at any specific point in time:
  $$\text{Cardinality} = \text{Total Number of Tuples (Rows)}$$

```text
           Degree / Arity = 5 (Total Columns)
   <---------------------------------------------------------->
   +---------+---------+---------+-----------+----------------+
   |  NAME   |   ID    |  CITY   |  COUNTRY  |     HOBBY      |  <-- Attribute Headers
   +=========+=========+=========+===========+================+
   |  NISHA  |    1    |  AGRA   |   INDIA   |    PLAYING     |  <-- Tuple 1
   +---------+---------+---------+-----------+----------------+
   | NIKITA  |    2    |  DELHI  |   INDIA   |    DANCING     |  <-- Tuple 2
   +---------+---------+---------+-----------+----------------+
   |  AJAY   |    3    |  AGRA   |   INDIA   |     CHESS      |  <-- Tuple 3
   +---------+---------+---------+-----------+----------------+
   |  ARPIT  |    4    |  PATNA  |   INDIA   |    READING     |  <-- Tuple 4
   +---------+---------+---------+-----------+----------------+
   ^                                                          ^
   |                                                          |
   +----------------------------------------------------------+
                  Cardinality = 4 (Total Rows)
```

---

## 2. Fundamental Properties of Relational Tables

Every valid relational table must conform to seven mathematical axioms:
1. **Atomic Values:** Every individual cell contains indivisible, scalar values (no multivalued sets, nested tables, or composite fields).
2. **Homogeneous Domain:** All data elements within a given column must be drawn from the identical data type and domain.
3. **Distinct Rows (No Duplicates):** Every row must be distinct and uniquely identifiable by a candidate/primary key.
4. **Unique Relation Names:** No two tables can share the same relation name within the identical database schema.
5. **Unique Attribute Names:** Every column inside a single table must possess a distinct, unique attribute name.
6. **Insignificant Row Sequence:** The physical vertical order of tuples is completely irrelevant; sorting does not alter semantic content.
7. **Insignificant Column Sequence:** The physical horizontal order of attributes is irrelevant; relational operators reference columns by name, not index.

---

## 3. Relational Database Problems: Update Anomalies

When a relational schema is un-normalized and merges multiple enterprise concepts into a single table, it introduces severe **Data Redundancy**, causing **Update Anomalies**.

Consider the un-normalized relation:
$$\text{STUDENT\_BRANCH}(\underline{\text{Roll\_no}}, \text{name}, \text{Age}, \text{Br\_code}, \text{Br\_name}, \text{Br\_hod\_name})$$

```text
+---------+------+-----+---------+---------+-------------+
| Roll_no | name | Age | Br_code | Br_name | Br_hod_name |
+---------+------+-----+---------+---------+-------------+
|    1    |  A   | 19  |   101   |   Cs    |     Abc     |
|    2    |  B   | 18  |   101   |   Cs    |     Abc     |
|    3    |  C   | 20  |   101   |   Cs    |     Abc     |
|    4    |  D   | 20  |   102   |   Ec    |     Pqr     |
+---------+------+-----+---------+---------+-------------+
```

### 3.1 The Three Types of Anomalies
1. **Insertion Anomaly:**
   * **Definition:** An independent piece of information cannot be recorded into a relation without inserting unrelated or dummy information at the same time.
   * **Example:** If the university establishes a new department (`Br_code: 103, Br_name: AI, Br_hod_name: XYZ`), this record cannot be inserted until at least one student enrolls because `Roll_no` is the Primary Key and cannot be `NULL` (Entity Integrity Violation).
2. **Modification (Update) Anomaly:**
   * **Definition:** Updating a single real-world fact requires modifications across multiple redundant rows.
   * **Example:** If the HOD of branch `101` changes from `Abc` to `XYZ`, the change must be executed on all rows containing `Br_code: 101`. Missing even a single row creates conflicting, inconsistent records.
3. **Deletion Anomaly:**
   * **Definition:** The deletion of a specific piece of information unintentionally removes unrelated, valuable enterprise data.
   * **Example:** If student `Roll_no: 4` (`D`) drops out, deleting that row unintentionally wipes out the entire record that branch `102` (`Ec`) exists with HOD `Pqr`.

### 3.2 The Decomposition Solution
To eliminate anomalies, decompose the single relation into two normalized relations linked via a Foreign Key:
* $\text{STUDENT}(\underline{\text{Roll\_no}}, \text{name}, \text{Age}, \text{Br\_code})$
* $\text{BRANCH}(\underline{\text{Br\_code}}, \text{Br\_name}, \text{Br\_hod\_name})$

```text
  STUDENT Table                              BRANCH Table
+---------+------+-----+---------+         +---------+---------+-------------+
| Roll_no | name | Age | Br_code |         | Br_code | Br_name | Br_hod_name |
+=========+======+=====+=========+         +=========+=========+=============+
|    1    |  A   | 19  |   101   |----+    |   101   |   Cs    |     Abc     |
|    2    |  B   | 18  |   101   |    +--->|   102   |   Ec    |     Pqr     |
|    3    |  C   | 20  |   101   |    |    +---------+---------+-------------+
|    4    |  D   | 20  |   102   |----+           (Primary Key)
+---------+------+-----+---------+
  (PK)                 (FK)
```

---

## 4. Theory of Functional Dependency (FD)

Functional Dependency is the formal mathematical tool used to evaluate the quality and correctness of relational database schemas.

### 4.1 Formal Mathematical Definition
Let $R$ be a relational schema, and let $\alpha \subseteq R$ and $\beta \subseteq R$.  
The Functional Dependency:
$$\alpha \rightarrow \beta \quad (\text{"}\alpha\text{ functionally determines }\beta\text{" or "}\beta\text{ is functionally dependent on }\alpha\text{"})$$
holds on relation $R$ if and only if for all possible legal pairs of tuples $t_1, t_2 \in R$:
$$\text{If } t_1[\alpha] = t_2[\alpha], \quad \text{then } t_1[\beta] = t_2[\beta]$$

* **Determinant:** $\alpha$ (the left-hand side attribute set).
* **Dependent:** $\beta$ (the right-hand side attribute set).
* **Axiomatic Rule:** If two rows have identical values for $\alpha$, they **must** have identical values for $\beta$. If $\alpha$ is identical but $\beta$ differs, the dependency is violated.

### 4.2 Tabular Verification of FDs
Consider relation $R(X, Y, Z)$:

```text
+---+---+---+
| X | Y | Z |
+===+===+===+
| 1 | 4 | 2 |  <-- Tuple 1
| 1 | 4 | 3 |  <-- Tuple 2
| 2 | 6 | 3 |  <-- Tuple 3
| 3 | 2 | 2 |  <-- Tuple 4
+---+---+---+
```

* **Testing $X \rightarrow Y$:**
  * For $X=1$, $Y=4$ in both rows.
  * For $X=2$, $Y=6$.
  * For $X=3$, $Y=2$.
  * Every unique $X$ maps to exactly one $Y$. **Result: $X \rightarrow Y$ HOLDS.**
* **Testing $X \rightarrow Z$:**
  * In Tuple 1, $X=1 \implies Z=2$.
  * In Tuple 2, $X=1 \implies Z=3$.
  * The same $X$ value produces two distinct $Z$ values. **Result: $X \rightarrow Z$ FAILS.**

### 4.3 Trivial vs. Non-Trivial Functional Dependencies
1. **Trivial Functional Dependency:**
   * An FD $\alpha \rightarrow \beta$ is **Trivial** if $\beta \subseteq \alpha$ ($\beta$ is a subset of $\alpha$).
   * Trivial FDs convey no new semantic information and always hold true across all database instances.
   * *Examples:* $A \rightarrow A$, $AB \rightarrow A$, $XYZ \rightarrow YZ$.
2. **Non-Trivial Functional Dependency:**
   * An FD $\alpha \rightarrow \beta$ is **Non-Trivial** if $\beta \not\subseteq \alpha$.
   * It is **Completely Non-Trivial** if $\alpha \cap \beta = \emptyset$ (no overlapping attributes, e.g., $A \rightarrow B$).

---

## 5. Armstrong's Axioms & Inference Rules

Developed by **William W. Armstrong** in **1974**, Armstrong's Axioms are inference rules used to logically derive all functional dependencies implied by an initial set of FDs $F$.

### 5.1 The Three Primary Axioms
1. **Reflexivity Rule:**
   $$\text{If } Y \subseteq X, \quad \text{then } X \rightarrow Y$$
2. **Augmentation Rule:**
   $$\text{If } X \rightarrow Y, \quad \text{then } XZ \rightarrow YZ \quad (\forall Z \subseteq R)$$
3. **Transitivity Rule:**
   $$\text{If } X \rightarrow Y \text{ and } Y \rightarrow Z, \quad \text{then } X \rightarrow Z$$

### 5.2 Secondary Derived Inference Rules
Derived directly by applying the primary axioms:
1. **Union Rule:**
   $$\text{If } X \rightarrow Y \text{ and } X \rightarrow Z, \quad \text{then } X \rightarrow YZ$$
2. **Decomposition (Projectivity) Rule:**
   $$\text{If } X \rightarrow YZ, \quad \text{then } X \rightarrow Y \text{ and } X \rightarrow Z$$
   *(Note: Decomposition applies strictly to the RHS, never to the LHS!)*
3. **Pseudo-Transitivity Rule:**
   $$\text{If } X \rightarrow Y \text{ and } WY \rightarrow Z, \quad \text{then } WX \rightarrow Z$$
4. **Composition Rule:**
   $$\text{If } X \rightarrow Y \text{ and } Z \rightarrow W, \quad \text{then } XZ \rightarrow YW$$

### 5.3 Soundness & Completeness
* **Soundness:** The axioms infer **only** those functional dependencies that are logically valid and true in every relation instance satisfying $F$ (no false or incorrect dependencies are generated).
* **Completeness:** Repeated application of the axioms guarantees the generation of **all possible** valid functional dependencies that can be inferred from $F$ (the complete closure $F^+$ is reachable).

---

## 6. Attribute Closure & Equivalence of FD Sets

### 6.1 Attribute Closure ($X^+$)
* **Definition:** The attribute closure of a set of attributes $X$ under a set of functional dependencies $F$ (denoted $X^+$) is the set of all attributes functionally determined by $X$.
* **Closure Computation Algorithm:**
  1. Initialize $X^+ = X$.
  2. Repeatedly search for an FD $Y \rightarrow Z \in F$ such that $Y \subseteq X^+$.
  3. Update $X^+ = X^+ \cup Z$.
  4. Repeat until no new attributes can be appended to $X^+$.

### 6.2 Equivalence of Two FD Sets ($F \equiv G$)
Two sets of functional dependencies $F$ and $G$ are equivalent if and only if their closures are identical:
$$F^+ = G^+ \iff (F \subseteq G^+) \text{ and } (G \subseteq F^+)$$

**Verification Procedure:**
* For every $X \rightarrow Y \in F$, compute $X^+$ using dependencies in $G$. Verify that $Y \subseteq X^+$.
* For every $A \rightarrow B \in G$, compute $A^+$ using dependencies in $F$. Verify that $B \subseteq A^+$.

---

## 7. Canonical Cover (Minimal Cover / Irreducible Set)

A **Canonical Cover** ($F_c$) of an FD set $F$ is a minimal, non-redundant set of functional dependencies equivalent to $F$.

### 7.1 The Three Types of Redundancies to Eliminate
1. **Redundant Dependencies:** An entire FD can be derived from other FDs in the set.
2. **Extraneous Attribute on RHS:** An attribute on the right side of an FD that can be removed without altering closure.
3. **Extraneous Attribute on LHS:** An attribute on the left side of an FD that can be eliminated without losing inferential power.

### 7.2 Step-by-Step Canonical Cover Algorithm
1. **Step 1 (RHS Decomposition):** Split all FDs so that each has a single attribute on its RHS using the decomposition rule:
   $$X \rightarrow YZ \implies X \rightarrow Y, \ X \rightarrow Z$$
2. **Step 2 (Eliminate Redundant FDs):** For each FD $X \rightarrow A \in F$, temporarily remove it and compute $X^+$ using the remaining dependencies:
   * If $A \in X^+$, the FD is redundant $\rightarrow$ **Remove it permanently**.
   * If $A \notin X^+$, the FD is essential $\rightarrow$ **Retain it**.
3. **Step 3 (Eliminate Extraneous LHS Attributes):** For each FD with multiple LHS attributes (e.g., $AB \rightarrow C$):
   * Check if $A$ is extraneous by calculating $B^+$. If $C \in B^+$, reduce to $B \rightarrow C$.
   * Check if $B$ is extraneous by calculating $A^+$. If $C \in A^+$, reduce to $A \rightarrow C$.

---

## 8. Database Keys Hierarchy

```
+-------------------------------------------------------------------------+
|                               SUPER KEYS                                |
|        (Any attribute set whose closure contains all attributes)        |
|                                                                         |
|   +-----------------------------------------------------------------+   |
|   |                        CANDIDATE KEYS                           |   |
|   |             (Minimal Super Keys; no subset is SK)               |   |
|   |                                                                 |   |
|   |   +---------------------------+   +-------------------------+   |   |
|   |   |        PRIMARY KEY        |   |     ALTERNATE KEYS      |   |   |
|   |   |  (Chosen by DBA; exactly  |   |  (Candidate keys not    |   |   |
|   |   |   one; NO NULLs allowed)  |   |   chosen as primary)    |   |   |
|   |   +---------------------------+   +-------------------------+   |   |
|   +-----------------------------------------------------------------+   |
+-------------------------------------------------------------------------+
```

### 8.1 Super Key
* A set of attributes $K$ is a Super Key of relation $R$ if:
  $$K^+ = R$$
* Every relation has at least one super key (in the extreme case, the union of all attributes).

### 8.2 Candidate Key
* A **minimal super key** (no proper subset of a Candidate Key can be a super key).
* Identifies each tuple uniquely without redundant attributes.
* Every table must have **at least one** candidate key.

### 8.3 Primary Key vs. Alternate Key
* **Primary Key (PK):** Exactly one candidate key selected by the DBA as the principal tuple identifier.
  * **Entity Integrity Constraint:** Attributes comprising the primary key **cannot contain NULL values**.
* **Alternate Key:** Any candidate key not chosen as the primary key:
  $$\text{Alternate Keys} = \text{Candidate Keys} - \{\text{Primary Key}\}$$

### 8.4 Foreign Key & Referential Integrity
* **Foreign Key (FK):** An attribute or set of attributes in a relation that references the Primary Key of another table (or the same table) to establish an association.
* **Referential Integrity Rule:**
  * An FK value must either match an existing PK value in the referenced parent table, OR
  * Must be **NULL** (unless constrained by `NOT NULL`).
  * Foreign keys **can accept NULLs** and **can have duplicate values**.
* **Self-Referencing Foreign Key:** When an FK references the primary key of the **same relation** (e.g., a student class representative $CR$ referencing another student's $Roll\_no$).

### 8.5 Composite Key vs. Secondary Key
* **Composite Key:** A key composed of two or more attributes (concatenated key).
* **Secondary Key:** An attribute used primarily for indexing to accelerate search performance. It does not need to be unique.

---

## 9. Prime vs. Non-Prime Attributes

Let relation $R$ have an attribute set $A$, and let $CK$ be the set of all candidate keys:
* **Prime Attribute:** Any attribute that is a constituent member of **at least one candidate key**:
  $$A_i \text{ is Prime} \iff \exists K \in CK \text{ such that } A_i \in K$$
* **Non-Prime Attribute:** An attribute that does **not** belong to any candidate key:
  $$A_j \text{ is Non-Prime} \iff \forall K \in CK, \ A_j \notin K$$

---

## 10. Solved Numerical Problems

### Problem 1: Attribute Closure Computation
Given relation $R(A, B, C, D, E, F, G)$ and FD set:
$$F = \{A \rightarrow B, \ BC \rightarrow DE, \ AEG \rightarrow G\}$$
Compute $(AC)^+$.

**Solution:**
1. Start with initial set: $(AC)^+ = \{A, C\}$.
2. Apply $A \rightarrow B$ (since $A \subseteq (AC)^+$):  
   $$(AC)^+ = \{A, B, C\}$$
3. Apply $BC \rightarrow DE$ (since $\{B, C\} \subseteq (AC)^+$):  
   $$(AC)^+ = \{A, B, C, D, E\}$$
4. Check $AEG \rightarrow G$: $\{A, E, G\} \not\subseteq \{A, B, C, D, E\}$. Cannot fire.
5. **Final Result:** $\mathbf{(AC)^+ = \{A, B, C, D, E\}}$

---

### Problem 2: Canonical Cover Derivation
Given relation $R(A, B, C, D)$ and FD set:
$$F = \{A \rightarrow B, \ C \rightarrow B, \ D \rightarrow ABC, \ AC \rightarrow D\}$$
Find the Canonical Cover $F_c$.

**Solution:**
1. **Step 1: Decompose RHS into single attributes:**
   $$F = \{A \rightarrow B, \ C \rightarrow B, \ D \rightarrow A, \ D \rightarrow B, \ D \rightarrow C, \ AC \rightarrow D\}$$
2. **Step 2: Check for redundant dependencies:**
   * Test $A \rightarrow B$: With $A \rightarrow B$ removed, $A^+ = \{A\} \not\ni B$. (Essential).
   * Test $C \rightarrow B$: With $C \rightarrow B$ removed, $C^+ = \{C\} \not\ni B$. (Essential).
   * Test $D \rightarrow B$: With $D \rightarrow B$ removed, compute $D^+$ using remaining FDs:
     $$D^+ = \{D\} \xrightarrow{D \rightarrow A, D \rightarrow C} \{D, A, C\} \xrightarrow{A \rightarrow B} \{D, A, C, B\}$$
     Since $B \in D^+$, **$D \rightarrow B$ is redundant and removed**.
   * Test $D \rightarrow A$: $D^+ = \{D, C, B\} \not\ni A$. (Essential).
   * Test $D \rightarrow C$: $D^+ = \{D, A, B\} \not\ni C$. (Essential).
   * Test $AC \rightarrow D$: $(AC)^+ = \{A, C, B\} \not\ni D$. (Essential).
   * Remaining FDs: $\{A \rightarrow B, \ C \rightarrow B, \ D \rightarrow A, \ D \rightarrow C, \ AC \rightarrow D\}$.
3. **Step 3: Check for extraneous attributes on LHS of $AC \rightarrow D$:**
   * Test if $A$ is extraneous: Compute $C^+$ without $AC \rightarrow D$: $C^+ = \{C, B\} \not\ni D$. ($A$ is essential).
   * Test if $C$ is extraneous: Compute $A^+$ without $AC \rightarrow D$: $A^+ = \{A, B\} \not\ni D$. ($C$ is essential).
4. **Step 4: Re-combine RHS where LHS is identical:**
   $$\mathbf{F_c = \{A \rightarrow B, \ C \rightarrow B, \ D \rightarrow AC, \ AC \rightarrow D\}}$$

---

### Problem 3: Candidate Keys & Prime/Non-Prime Classification
Given relation $R(A, B, C, D, E)$ and FD set:
$$F = \{A \rightarrow B, \ BC \rightarrow E, \ ED \rightarrow A\}$$
Find all Candidate Keys, Prime Attributes, and Non-Prime Attributes.

**Solution:**
1. Attributes $C$ and $D$ never appear on the RHS of any FD. Therefore, $\{C, D\}$ must be part of every candidate key.
2. Compute closures of combinations with $CD$:
   * $(ACD)^+ = \{A, C, D\} \xrightarrow{A \rightarrow B} \{A, B, C, D\} \xrightarrow{BC \rightarrow E} \{A, B, C, D, E\} = R \implies \mathbf{ACD}$ is a CK.
   * $(BCD)^+ = \{B, C, D\} \xrightarrow{BC \rightarrow E} \{B, C, D, E\} \xrightarrow{ED \rightarrow A} \{A, B, C, D, E\} = R \implies \mathbf{BCD}$ is a CK.
   * $(ECD)^+ = \{E, C, D\} \xrightarrow{ED \rightarrow A} \{E, C, D, A\} \xrightarrow{A \rightarrow B} \{A, B, C, D, E\} = R \implies \mathbf{ECD}$ is a CK.
3. **Candidate Keys:** $\{ACD, \ BCD, \ ECD\}$
4. **Prime Attributes:** $\{A, B, C, D, E\}$ (All attributes are prime).
5. **Non-Prime Attributes:** $\emptyset$ (None).

---

## 11. Bihar Engineering University (BEU) Exam Section

### 11.1 Frequently Asked University Questions

#### Short Answer Questions (5 Marks)
1. Define Degree (Arity) and Cardinality of a relation with an example.
2. Differentiate between a Super Key and a Candidate Key.
3. State and explain the three primary Armstrong Axioms.
4. Explain Insertion, Modification, and Deletion anomalies with an un-normalized table example.

#### Long Answer Questions (7 to 14 Marks)
1. **What is a Functional Dependency? Define Trivial and Non-Trivial FDs. State and prove Armstrong's Axioms. Why are they referred to as sound and complete?** *(Frequently Asked - 14 Marks)*
2. **Explain the Canonical Cover (Minimal Cover) of a functional dependency set. State the complete step-by-step algorithm and find the canonical cover for $F = \{A \rightarrow B, C \rightarrow B, D \rightarrow ABC, AC \rightarrow D\}$.** *(Frequently Asked - 14 Marks)*
3. **Discuss the concept of Referential Integrity. How is it implemented via Foreign Keys? Can a Foreign Key accept NULL or duplicate values? Justify with a schema example.** *(7 Marks)*
4. **State the criteria for equivalence of two sets of functional dependencies. Verify whether $F = \{A \rightarrow C, AC \rightarrow D, E \rightarrow AD, E \rightarrow H\}$ and $G = \{A \rightarrow CD, E \rightarrow AH\}$ are equivalent.** *(7 Marks)*

---

### 11.2 Model Exam Answer: Armstrong's Axioms (BEU Presentation Standard)

**Question:** State and prove Armstrong's Axioms. Why are they called sound and complete? *(7 Marks)*

**Model Answer:**

**1. Definition:**  
Armstrong's Axioms are a set of inference rules developed by William W. Armstrong in 1974 used to deduce all functional dependencies logically implied by an FD set $F$.

**2. Primary Axioms:**
1. **Reflexivity Rule:** If $Y \subseteq X$, then $X \rightarrow Y$.
2. **Augmentation Rule:** If $X \rightarrow Y$, then $XZ \rightarrow YZ$ for any attribute set $Z$.
3. **Transitivity Rule:** If $X \rightarrow Y$ and $Y \rightarrow Z$, then $X \rightarrow Z$.

**3. Mathematical Proof of Transitivity Rule:**
* Let $X \rightarrow Y$ and $Y \rightarrow Z$ hold in relation $R$.
* By definition of FD, for any tuples $t_1, t_2 \in R$:
  $$\text{If } t_1[X] = t_2[X], \quad \text{then } t_1[Y] = t_2[Y]$$
* Since $Y \rightarrow Z$ also holds:
  $$\text{If } t_1[Y] = t_2[Y], \quad \text{then } t_1[Z] = t_2[Z]$$
* Combining both implications:
  $$\text{If } t_1[X] = t_2[X] \implies t_1[Z] = t_2[Z]$$
* Therefore, by definition, $X \rightarrow Z$ holds. Hence proved.

**4. Sound and Complete Nature:**
* **Sound:** They generate *only* functional dependencies that are logically valid in all relation states satisfying $F$. No incorrect FDs are produced.
* **Complete:** Repeated application of these rules generates *all* valid functional dependencies in the closure $F^+$. No valid FD is missed.

---

### 11.3 Important Multiple-Choice Questions (MCQs)

1. **Who proposed the Relational Database Model in 1970?**
   * (A) Peter Chen
   * (B) E. F. Codd
   * (C) William Armstrong
   * (D) C. J. Date  
   * **Correct Answer:** **(B) E. F. Codd**

2. **The total count of attributes in a relational schema is defined as its:**
   * (A) Cardinality
   * (B) Arity / Degree
   * (C) Domain count
   * (D) Order  
   * **Correct Answer:** **(B) Arity / Degree**

3. **An attribute set $K$ is a Candidate Key if:**
   * (A) $K^+ = R$ and no proper subset of $K$ has closure equal to $R$
   * (B) $K$ contains only one attribute
   * (C) $K$ is selected by the DBA and cannot be NULL
   * (D) $K$ contains all non-prime attributes  
   * **Correct Answer:** **(A) $K^+ = R$ and no proper subset of $K$ has closure equal to $R$**

4. **Which rule states that a Primary Key attribute can never take a NULL value?**
   * (A) Referential Integrity Constraint
   * (B) Entity Integrity Constraint
   * (C) Domain Integrity Constraint
   * (D) Key Completeness Rule  
   * **Correct Answer:** **(B) Entity Integrity Constraint**

5. **A functional dependency $X \rightarrow Y$ is trivial if:**
   * (A) $X \subseteq Y$
   * (B) $Y \subseteq X$
   * (C) $X \cap Y = \emptyset$
   * (D) $Y = \emptyset$  
   * **Correct Answer:** **(B) $Y \subseteq X$**

6. **The Armstrong axiom stating that "If $X \rightarrow Y$, then $XZ \rightarrow YZ$" is termed the:**
   * (A) Augmentation Rule
   * (B) Reflexivity Rule
   * (C) Composition Rule
   * (D) Transitivity Rule  
   * **Correct Answer:** **(A) Augmentation Rule**

7. **In a relation where all attributes belong to at least one candidate key, the set of non-prime attributes is:**
   * (A) Equal to the relation schema
   * (B) Empty ($\emptyset$)
   * (C) The primary key
   * (D) Undefined  
   * **Correct Answer:** **(B) Empty ($\emptyset$)**