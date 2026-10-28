# Chapter 4: Relational Database Normalization & Decomposition

---

## 1. Introduction to Database Normalization

* **Definition:** Normalization is a systematic refinement process of analyzing relational schemas based on their functional dependencies (FDs) and primary keys.
* **Core Philosophy:** "Every table must represent a single theme or concept (one idea per table)." If a table combines multiple enterprise concepts, it must be systematically decomposed into smaller, well-structured relations.
* **Primary Objectives:**
  1. **Eliminate Data Redundancy:** Minimize duplicate data storage across disk pages.
  2. **Eliminate Inconsistent Dependencies:** Ensure logical dependencies reflect real-world business constraints.
  3. **Prevent Update Anomalies:** Completely eradicate Insertion, Modification, and Deletion anomalies.

```text
               +-------------------------------------------------------+
               |                       1 NF                            |
               |                (Atomic Cell Values)                   |
               |   +-----------------------------------------------+   |
               |   |                   2 NF                        |   |
               |   |        (No Partial Dependencies)              |   |
               |   |   +---------------------------------------+   |   |
               |   |   |               3 NF                    |   |   |
               |   |   |     (No Transitive Dependencies)      |   |   |
               |   |   |   +-------------------------------+   |   |   |
               |   |   |   |             BCNF              |   |   |   |
               |   |   |   |   (Determinant is Super Key)  |   |   |   |
               |   |   |   |   +-----------------------+   |   |   |   |
               |   |   |   |   |         4 NF          |   |   |   |   |
               |   |   |   |   |       (No MVD)        |   |   |   |   |
               |   |   |   |   |   +---------------+   |   |   |   |   |
               |   |   |   |   |   |     5 NF      |   |   |   |   |   |
               |   |   |   |   |   |    (PJNF)     |   |   |   |   |   |
               |   |   |   |   |   +---------------+   |   |   |   |   |
               |   |   |   |   +-----------------------+   |   |   |   |
               |   |   |   +-------------------------------+   |   |   |
               |   |   +---------------------------------------+   |   |
               |   +-----------------------------------------------+   |
               +-------------------------------------------------------+
```

---

## 2. First Normal Form (1NF)

### 2.1 Formal Definition
A relation schema $R$ is in **First Normal Form (1NF)** if and only if the domain of each attribute contains only **atomic (indivisible) values**, and each attribute value in any tuple is a single scalar value from the domain.

### 2.2 Conditions for 1NF
1. **Atomic Values:** Cells must not contain multivalued attributes (arrays, sets, comma-separated lists) or composite attributes.
2. **Unique Column Names:** Each attribute within the relation must have a distinct, unique identifier.
3. **Primary Key Requirement:** Every tuple in the relation must be uniquely identifiable by a designated primary key.
4. **Duplicate Elimination:** No two identical tuples can exist in a relation instance.

---

## 3. Second Normal Form (2NF)

### 3.1 Prerequisite Definitions
* **Prime Attribute:** An attribute that is a constituent element of **at least one candidate key**.
* **Non-Prime Attribute:** An attribute that does **not** belong to any candidate key.
* **Full Functional Dependency:** A functional dependency $X \rightarrow Y$ where removing any attribute $A$ from $X$ causes the dependency to no longer hold:
  $$\forall A \in X, \ (X - \{A\}) \not\rightarrow Y$$
* **Partial Dependency:** A functional dependency where a non-prime attribute is functionally determined by a **proper subset of a composite candidate key**:
  $$\text{Proper Subset of Candidate Key} \longrightarrow \text{Non-Prime Attribute}$$

### 3.2 Formal Definition of 2NF
A relation schema $R$ is in **Second Normal Form (2NF)** if and only if:
1. It is already in **1NF**.
2. **No Partial Dependencies exist** in $R$ (i.e., every non-prime attribute is **fully functionally dependent** on every candidate key).

$$\text{Forbidden in 2NF: } \text{Proper Subset of CK (Prime)} \longrightarrow \text{Non-Prime Attribute}$$

### 3.3 2NF Decomposition Example
Consider relation $R(A, B, C)$ with $FD = \{B \rightarrow C\}$ and candidate key $AB$:
* Prime Attributes: $\{A, B\}$
* Non-Prime Attribute: $\{C\}$
* $B \rightarrow C$ is a **Partial Dependency** because $B \subset AB$ determines non-prime $C$.

```text
       Original Relation R(A, B, C) [Violates 2NF]
       +---+---+---+
       | A | B | C |
       +===+===+===+
       | a | 1 | X |
       | b | 2 | Y |
       | a | 3 | Z |  <-- Redundancy: B=3 maps to C=Z repeatedly
       | c | 3 | Z |  <--
       | d | 3 | Z |  <--
       +---+---+---+
```

**Decomposition into 2NF:**
* $R_1(\underline{A, B})$ with Primary Key $AB$
* $R_2(\underline{B}, C)$ with Primary Key $B$

---

## 4. Third Normal Form (3NF)

### 4.1 Transitive Dependency
* **Definition:** A dependency where a non-prime attribute determines another non-prime attribute (indirect determination via an intermediate non-key attribute):
  $$\text{Candidate Key} \longrightarrow X \longrightarrow Y \quad (\text{where } X \text{ and } Y \text{ are Non-Prime})$$
  $$\text{Forbidden Pattern: } \text{Non-Prime} \longrightarrow \text{Non-Prime}$$

### 4.2 Formal Definition of 3NF
* **Textbook Definition:** A relation schema $R$ is in **3NF** if it is in 2NF and contains **no transitive dependencies**.
* **Direct / Mathematical Definition:**  
  A relation schema $R$ is in **3NF** if and only if for every non-trivial functional dependency $\alpha \rightarrow \beta$ holding on $R$, at least **one** of the following conditions is satisfied:
  1. $\mathbf{\alpha}$ **is a Super Key (SK)** of $R$, **OR**
  2. $\mathbf{\beta}$ **is a Prime Attribute** of $R$.

### 4.3 3NF Decomposition Example
Consider $R(A, B, C)$ with candidate key $A$ and $FDs = \{A \rightarrow B, \ B \rightarrow C\}$:
* Prime Attribute: $\{A\}$; Non-Prime Attributes: $\{B, C\}$.
* In $B \rightarrow C$: $B$ is not a super key, and $C$ is not a prime attribute. This is a **Transitive Dependency** and violates 3NF.
* **Decomposition into 3NF:**
  * $R_1(\underline{A}, B)$ with Primary Key $A$
  * $R_2(\underline{B}, C)$ with Primary Key $B$

---

## 5. Boyce-Codd Normal Form (BCNF)

### 5.1 Formal Definition
A relation schema $R$ is in **Boyce-Codd Normal Form (BCNF)** if and only if for every non-trivial functional dependency $\alpha \rightarrow \beta$ holding on $R$:
$$\mathbf{\alpha} \text{ must be a Super Key (SK) of } R$$

* BCNF is a stricter, stronger version of 3NF.
* It eliminates the alternative allowance that 3NF provides ($\beta$ being a prime attribute).
* Any relation in BCNF is guaranteed to be in 3NF, 2NF, and 1NF.

### 5.2 Classic 3NF vs. BCNF Violation Example
Consider relation $R(A, B, C)$ with $FDs = \{AB \rightarrow C, \ C \rightarrow B\}$:
1. **Candidate Keys:** $AB$ and $AC$.
2. **Prime Attributes:** $\{A, B, C\}$ (All attributes are prime).
3. **Evaluating $C \rightarrow B$:**
   * In 3NF: Since $B$ is a prime attribute, the dependency **satisfies 3NF**.
   * In BCNF: $C$ is not a super key ($C^+ = \{B, C\} \neq ABC$), so it **violates BCNF**.
4. **BCNF Decomposition:**
   * $R_1(\underline{C}, B)$ with Primary Key $C$
   * $R_2(\underline{A, C})$ with Primary Key $AC$

---

## 6. Critical Normalization Theorems

1. **Binary Relation Theorem:**  
   * Any relation schema consisting of **exactly two attributes** ($R(A, B)$) is **always in BCNF**.
2. **All-Prime Relation Theorem:**  
   * If a relation schema $R$ consists **exclusively of prime attributes** (no non-prime attributes exist), then $R$ is **always guaranteed to be in 3NF**, but it may or may not be in BCNF.

---

## 7. Multivalued Dependency (MVD) & Fourth Normal Form (4NF)

### 7.1 Multivalued Dependency (MVD)
* **Notation:** $\alpha \twoheadrightarrow \beta$ ("$\alpha$ multidetermines $\beta$").
* **Definition:** In a relation $R(\alpha, \beta, \gamma)$, an MVD $\alpha \twoheadrightarrow \beta$ holds if the set of values of $\beta$ associated with a given value of $\alpha$ is completely independent of the values of $\gamma$.
* **MVD Redundancy:** When two or more independent multivalued properties are stored in a single relation, every value of the first attribute must be paired with every value of the second attribute (Cartesian Product effect), creating massive redundancy even when the relation is in BCNF.

### 7.2 Trivial vs. Non-Trivial MVD
An MVD $X \twoheadrightarrow Y$ in relation $R$ is **Trivial** if:
1. $Y \subseteq X$, **OR**
2. $X \cup Y = R$ (the attributes together constitute the entire relation schema).

Otherwise, it is **Non-Trivial**.

### 7.3 Formal Definition of 4NF
A relation schema $R$ is in **Fourth Normal Form (4NF)** if and only if:
1. It is already in **BCNF**.
2. For every non-trivial multivalued dependency $X \twoheadrightarrow Y$ holding on $R$, $\mathbf{X}$ **must be a Super Key** of $R$.

### 7.4 4NF Decomposition Example
Consider relation $\text{STUDENT}(\text{S\_Name}, \text{Club\_Name}, \text{Phone\_No})$:
* A student can be in multiple clubs and have multiple phone numbers independently.
* MVDs: $\text{S\_Name} \twoheadrightarrow \text{Club\_Name}$ and $\text{S\_Name} \twoheadrightarrow \text{Phone\_No}$.
* Candidate Key: $\{\text{S\_Name}, \text{Club\_Name}, \text{Phone\_No}\}$.
* $\text{S\_Name}$ is not a super key, violating 4NF.
* **Decomposition into 4NF:**
  * $R_1(\underline{\text{S\_Name}, \text{Club\_Name}})$
  * $R_2(\underline{\text{S\_Name}, \text{Phone\_No}})$

---

## 8. Fifth Normal Form (5NF / Project-Join Normal Form - PJNF)

### 8.1 Concept of Join Dependency ($\Join$)
* A Join Dependency $\Join(R_1, R_2, \dots, R_n)$ holds on relation $R$ if $R$ can be losslessly reconstructed by joining its projections $R_1, R_2, \dots, R_n$.
* A join dependency is a generalization of multivalued dependencies.

### 8.2 Formal Definition of 5NF
A relation schema $R$ is in **Fifth Normal Form (5NF)** if and only if:
1. It is already in **4NF**.
2. Every non-trivial join dependency $\Join(R_1, R_2, \dots, R_n)$ on $R$ is implied by the candidate keys of $R$.
3. Stated simply: **$R$ cannot be further non-loss decomposed into smaller relations.**

---

## 9. Properties of Relational Decompositions

When decomposing relation $R$ into sub-relations $\{R_1, R_2, \dots, R_n\}$, two mathematical properties govern the quality of the decomposition.

### 9.1 Lossless-Join Decomposition (Non-Additive Join)
* **Definition:** A decomposition is **Lossless** if taking the natural join ($\bowtie$) of all decomposed sub-relations reconstructs the exact original relation without producing extra (spurious) tuples or losing records:
  $$R_1 \bowtie R_2 \bowtie \dots \bowtie R_n = R$$
* **Lossy Decomposition:** A decomposition where natural join generates fake/spurious tuples ($R_1 \bowtie R_2 \supset R$). Information is lost because the true pairings cannot be distinguished from spurious pairings.
* **Strict Mandate:** **Lossless join property is mandatory and must be preserved at all costs.**

#### Three Conditions for Binary Lossless Decomposition ($R \rightarrow R_1, R_2$):
1. **Attribute Preservation:** $\text{Att}(R_1) \cup \text{Att}(R_2) = \text{Att}(R)$
2. **Non-Empty Intersection:** $\text{Att}(R_1) \cap \text{Att}(R_2) \neq \emptyset$
3. **Common Key Condition (Crucial):** The common attribute set must be a **Super Key** in at least one of the decomposed relations:
   $$\left(\text{Att}(R_1) \cap \text{Att}(R_2)\right) \rightarrow R_1 \quad \text{OR} \quad \left(\text{Att}(R_1) \cap \text{Att}(R_2)\right) \rightarrow R_2$$

### 9.2 Dependency Preserving Decomposition
* **Definition:** Let $F$ be the set of FDs on $R$, and let $F_i$ be the projection of $F$ onto $R_i$. The decomposition is **Dependency Preserving** if and only if:
  $$\left(F_1 \cup F_2 \cup \dots \cup F_n\right)^+ = F^+$$
* **Significance:** Allows the database to verify integrity constraints locally within individual tables without executing expensive cross-table joins during updates.
* **Trade-off:**
  * **3NF** decomposition can **always** achieve both Lossless Join and Dependency Preservation simultaneously.
  * **BCNF** decomposition guarantees Lossless Join, but **may not preserve dependencies**.

---

## 10. Master Solved Numericals

### Problem 1: 2NF Decomposition
**Statement:** Universal schema $R(A, B, C, D, E, F, G, H, I, J)$ with $F = \{AB \rightarrow C, \ A \rightarrow DE, \ B \rightarrow F, \ F \rightarrow GH, \ D \rightarrow IJ\}$.  
1. Determine all Candidate Keys.  
2. Decompose $R$ into 2NF.

**Step 1: Finding Candidate Keys**
* $A$ and $B$ never appear on the RHS of any FD. Hence, $\{A, B\}$ must be present in every candidate key.
* Compute $(AB)^+$:
  $$(AB)^+ = \{A, B\} \xrightarrow{AB \rightarrow C} \{A, B, C\} \xrightarrow{A \rightarrow DE} \{A, B, C, D, E\} \xrightarrow{B \rightarrow F} \{A, B, C, D, E, F\}$$
  $$\xrightarrow{F \rightarrow GH} \{A, B, C, D, E, F, G, H\} \xrightarrow{D \rightarrow IJ} \{A, B, C, D, E, F, G, H, I, J\} = R$$
* Candidate Key = $\mathbf{AB}$.
  * Prime Attributes = $\{A, B\}$
  * Non-Prime Attributes = $\{C, D, E, F, G, H, I, J\}$

**Step 2: Identifying Partial Dependencies**
* $A \rightarrow DE$ (Partial: $A \subset AB$ determines non-prime $D, E$).
* $B \rightarrow F$ (Partial: $B \subset AB$ determines non-prime $F$).

**Step 3: 2NF Decomposition**
* Sub-relation for $A$: Compute $A^+ = \{A, D, E, I, J\} \implies \mathbf{R_1(\underline{A}, D, E, I, J)}$ with Key: $A$.
* Sub-relation for $B$: Compute $B^+ = \{B, F, G, H\} \implies \mathbf{R_2(\underline{B}, F, G, H)}$ with Key: $B$.
* Sub-relation for remaining key attributes: $\mathbf{R_3(\underline{A, B}, C)}$ with Key: $AB$.
* **Final 2NF Schema:** $\{R_1(\underline{A}, D, E, I, J), \ R_2(\underline{B}, F, G, H), \ R_3(\underline{A, B}, C)\}$.

---

### Problem 2: Finding Highest Normal Form
**Statement:** Find the highest normal form of relation $R(A, B, C, D, E)$ with $F = \{A \rightarrow B, \ BC \rightarrow E, \ ED \rightarrow A\}$.

**Solution:**
1. Attributes $C$ and $D$ never appear on any RHS. Every key must contain $\{C, D\}$.
2. Closures:
   * $(ACD)^+ = \{A, C, D\} \rightarrow \{A, B, C, D, E\} = R \implies \mathbf{ACD}$ is a CK.
   * $(BCD)^+ = \{B, C, D\} \rightarrow \{B, C, D, E, A\} = R \implies \mathbf{BCD}$ is a CK.
   * $(ECD)^+ = \{E, C, D\} \rightarrow \{E, C, D, A, B\} = R \implies \mathbf{ECD}$ is a CK.
3. Candidate Keys: $\{ACD, BCD, ECD\}$.
4. Prime Attributes: $\{A, B, C, D, E\}$ (All attributes are prime; Non-prime = $\emptyset$).
5. **Checking Normal Forms:**
   * 1NF: Satisfied (atomic domains).
   * 2NF: Satisfied (no non-prime attributes exist $\implies$ no partial dependency possible).
   * 3NF: Satisfied (since all attributes are prime, the RHS of every FD is always prime).
   * BCNF: In $A \rightarrow B$, $A^+ = \{A, B\} \neq R$. $A$ is **not a Super Key**. BCNF is violated.
6. **Verdict:** The relation is in **3NF**.

---

### Problem 3: Dependency Preservation Verification
**Statement:** Universal schema $X(P, Q, R, S)$ with $F = \{QR \rightarrow S, \ R \rightarrow P, \ S \rightarrow Q\}$ is decomposed into $Y(P, R)$ and $Z(Q, R, S)$. Test for dependency preservation.

**Solution:**
1. **Project FDs onto sub-relations:**
   * For $Y(P, R)$: $R \rightarrow P$ has both attributes in $Y \implies \mathbf{F_1 = \{R \rightarrow P\}}$.
   * For $Z(Q, R, S)$: $QR \rightarrow S$ and $S \rightarrow Q$ have all attributes in $Z \implies \mathbf{F_2 = \{QR \rightarrow S, \ S \rightarrow Q\}}$.
2. **Union of projected FDs:**
   $$F' = F_1 \cup F_2 = \{R \rightarrow P, \ QR \rightarrow S, \ S \rightarrow Q\}$$
3. **Coverage Check:**
   * $QR \rightarrow S \in F'$ (Directly preserved).
   * $R \rightarrow P \in F'$ (Directly preserved).
   * $S \rightarrow Q \in F'$ (Directly preserved).
4. **Verdict:** Since $(F_1 \cup F_2)^+ = F^+$, the decomposition is **strictly Dependency Preserving**.
5. **Bonus Lossless Check:**
   * $Y \cap Z = \{R\}$.
   * $R^+ = \{R, P\} = \text{Att}(Y) \implies R$ is a Super Key of $Y$.
   * Therefore, the decomposition is **both Lossless and Dependency Preserving**.

---

## 11. Normal Forms Summary Comparison

```text
+-------------+------------------------------------+------------------------------------------+
| Normal Form | Disallowed Structural Dependency   | Exact Mathematical Test for X -> Y       |
+-------------+------------------------------------+------------------------------------------+
| 1 NF        | Multivalued / Composite attributes | Every attribute value must be atomic.    |
| 2 NF        | Partial Dependency                 | X cannot be a proper subset of any CK    |
|             |                                    | if Y is a Non-Prime Attribute.           |
| 3 NF        | Transitive Dependency              | Either X is a Super Key OR               |
|             |                                    | Y is a Prime Attribute.                  |
| BCNF        | Non-Superkey Determinant           | X must strictly be a Super Key.          |
| 4 NF        | Multivalued Dependency (MVD)       | For non-trivial X ->-> Y,                |
|             |                                    | X must be a Super Key.                   |
| 5 NF        | Non-Trivial Join Dependency (PJNF) | All non-trivial join dependencies are    |
|             |                                    | implied by candidate keys.               |
+-------------+------------------------------------+------------------------------------------+
```

---

## 12. Bihar Engineering University (BEU) Exam Section

### 12.1 Frequently Asked University Questions

#### Short Answer Questions (5 Marks)
1. Differentiate between Partial Dependency and Transitive Dependency with examples.
2. State the conditions for a binary decomposition to be Lossless.
3. State the two critical Normalization Theorems regarding 2-attribute relations and all-prime relations.
4. Define Multivalued Dependency (MVD) and state the definition of 4NF.

#### Long Answer Questions (7 to 14 Marks)
1. **Explain the progressive hierarchy of Normal Forms from 1NF to BCNF with formal mathematical definitions and counter-examples.** *(Frequently Asked - 14 Marks)*
2. **What is Lossless Join Decomposition? Explain how spurious tuples are created in a lossy decomposition using a concrete table example.** *(Frequently Asked - 14 Marks)*
3. **What is Dependency Preservation? Can every relation schema be decomposed into BCNF with dependency preservation? Explain with a counter-example.** *(7/14 Marks)*
4. **Consider relation $R(A, B, C, D, E, F, G, H, I, J)$ with $F = \{AB \rightarrow C, A \rightarrow DE, B \rightarrow F, F \rightarrow GH, D \rightarrow IJ\}$. Decompose $R$ into Second Normal Form.** *(Frequently Asked - 14 Marks)*

---

### 12.2 Model Exam Answer: Lossless vs. Lossy (BEU Presentation Standard)

**Question:** Define Lossless Join Decomposition. Prove with a relation instance how a lossy decomposition produces spurious tuples. *(7 Marks)*

**Model Answer:**

**1. Definition:**  
A decomposition of relation $R$ into sub-relations $R_1$ and $R_2$ is **Lossless** if taking their natural join reconstructs the original relation exactly:
$$R_1 \bowtie R_2 = R$$
If the natural join produces extra, invalid tuples ($R_1 \bowtie R_2 \supset R$), the decomposition is **Lossy**.

**2. Demonstration of Lossy Decomposition:**  
Let relation $R(A, B, C)$ have the following instance:
```text
  Relation R
+---+---+---+
| A | B | C |
+===+===+===+
| 1 | a | p |
| 2 | b | q |
| 3 | a | r |
+---+---+---+
```
Decompose $R$ on attribute $B$ into $R_1(A, B)$ and $R_2(B, C)$:
```text
     Table R1              Table R2
   +---+---+             +---+---+
   | A | B |             | B | C |
   +===+===+             +===+===+
   | 1 | a |             | a | p |
   | 2 | b |             | b | q |
   | 3 | a |             | a | r |
   +---+---+             +---+---+
```
Compute the Natural Join $R_1 \bowtie_B R_2$:
* $(1, a) \bowtie (a, p) \implies (1, a, p)$ (Original)
* $(1, a) \bowtie (a, r) \implies \mathbf{(1, a, r)}$ (**Spurious Tuple**)
* $(2, b) \bowtie (b, q) \implies (2, b, q)$ (Original)
* $(3, a) \bowtie (a, p) \implies \mathbf{(3, a, p)}$ (**Spurious Tuple**)
* $(3, a) \bowtie (a, r) \implies (3, a, r)$ (Original)

**3. Conclusion:**  
$R_1 \bowtie R_2$ yields 5 tuples instead of the original 3. The tuples $(1, a, r)$ and $(3, a, p)$ are spurious. Hence, the decomposition is **Lossy** because the common attribute $B$ is not a super key in either $R_1$ or $R_2$.

---

### 12.3 Important Multiple-Choice Questions (MCQs)

1. **A relation $R$ is in 2NF if it is in 1NF and:**
   * (A) Every determinant is a candidate key
   * (B) No non-prime attribute is partially dependent on any candidate key
   * (C) There are no transitive dependencies
   * (D) All attributes are prime  
   * **Correct Answer:** **(B) No non-prime attribute is partially dependent on any candidate key**

2. **In 3NF, for every non-trivial functional dependency $X \rightarrow Y$:**
   * (A) $X$ must be a candidate key
   * (B) $Y$ must be a non-prime attribute
   * (C) Either $X$ is a super key or $Y$ is a prime attribute
   * (D) $X$ and $Y$ must both be prime  
   * **Correct Answer:** **(C) Either $X$ is a super key or $Y$ is a prime attribute**

3. **Which normal form strictly requires that the left-hand side of every non-trivial functional dependency must be a Super Key?**
   * (A) 2NF
   * (B) 3NF
   * (C) BCNF
   * (D) 4NF  
   * **Correct Answer:** **(C) BCNF**

4. **Any relational schema with only two attributes is always in:**
   * (A) 1NF only
   * (B) 2NF only
   * (C) 3NF only
   * (D) BCNF  
   * **Correct Answer:** **(D) BCNF**

5. **Fourth Normal Form (4NF) is designed to handle anomalies caused by:**
   * (A) Partial dependencies
   * (B) Transitive dependencies
   * (C) Multivalued dependencies
   * (D) Join dependencies  
   * **Correct Answer:** **(C) Multivalued dependencies**

6. **If decomposition of $R$ into $R_1$ and $R_2$ produces $R_1 \bowtie R_2 \supset R$, the decomposition is:**
   * (A) Lossless
   * (B) Lossy
   * (C) Dependency preserving
   * (D) Non-additive  
   * **Correct Answer:** **(B) Lossy**

7. **Which property can always be guaranteed during 3NF decomposition, but may be sacrificed during BCNF decomposition?**
   * (A) Lossless Join
   * (B) Atomicity
   * (C) Dependency Preservation
   * (D) Minimality  
   * **Correct Answer:** **(C) Dependency Preservation**