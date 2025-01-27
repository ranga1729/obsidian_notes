### Intro
Database normalization is a process of organizing data in a relational database to minimize redundancy and improve data integrity. It follows a set of rules called normal forms, which progressively reduce repetition and inconsistencies in the data. 
#### Purpose of Normalization
- Identify the suitable set of relations that support data requirements of an enterprise.
- To identify,
	the minimal number of attributes,
	attributes with a close logical relationship
	Minimal redundancy.

#### How normalization Supports Database Design 
![[Pasted image 20240622211102.png]]

Aim of the normalization is to *group attributes into relations to minimize data redundancy.*
Potential benefits of achieving this,
1. Minimal number of update operations.
2. Reducing inconsistencies.
3. Reduce the storage cost

#### Update Anomalies
Update anomalies occur in a database when modifying data leads to inconsistencies or inefficiencies. This often happens due to data redundancy. 
Update anomalies can be divide into,
1. Insertion anomalies
2. Deletion anomalies
3. Modification anomalies

Ex: 
```sql
//right way
Employee {EmpId, Ename, DDate, Address, Dnumber}
Department {Dnumber, Dname, DmgId}

//wrong way
Emp_Dept {EmpId, Ename, BDate, Address, Dnumber, Dname, DmgrId}
```

**Insertion Anomalies**
1. When inserting a new employee tuple to `Emp_Dept` table, we must include either the attribute values for the department that the employee works for or nulls.
2. It is difficult to insert a new department that has no employees.

**Deletion anomalies**
1. If we delete from `Emp_Dept` the employee working for a particular department, the information concerning that Department is also get lost from database. 
	No employees in that department means department data is also lost. 

**Modification Anomalies**
1. If we want to modify an attribute related to a particular department in `Emp_Dept` tables, we have to change all employee tuples in that department. 

These anomalies can be overcome by decomposing the original tables into `Employee` and `Department`. 

The process of normalization through decomposition must confirm the existence of the following properties,
1. The lossless join or non additive join property
	Disallows the possibility of generating spurious tuples with respect to the relation schema created after decomposition. 
> [!NOTE] Spurious Tuples
> spurious tuples rer to unwanted or inaccurate records created during the process of joining two tables. 
> These extra rows don't represent valid data within the context of the intended relationship between the tables.

2. The dependency preservation property
	Ensures that each functional dependency is represented in some individual relation resulting after decomposition.

##### 1. Spurious Tuples
Ex:
![[Pasted image 20240622231711.png]]
By joining these tables we get this,
![[Pasted image 20240622231724.png]]In this tables, A one row as a whole doesn't mean anything itself.
`Pname`(product name) and Employee details are in different contexts. 

##### 2. Functional Dependencies
Functional dependencies refers to the relationships between attributes (columns) in a table, specifically how certain attributes determine the values of others.

If $A$ and $B$ are attributes of relation $R$, 
then $B$ is said to be functionally dependent on $A$ if each value of $A$ is associated with exactly one value of $B$.  d
Denoted by $(A \circledcirc B)$ 

When functional dependencies exist, the attribute or group of attributes on the left hand side of the arrow is called the determinant. 
![[Pasted image 20240622232932.png]]
	$F$ is denotes the set of functional dependencies that are specified on relation schema $R$.
	There are other dependencies that can be inferred or deduced from FDs in F.
Ex: 
If each department has one manager, Dept_no uniquely determines Mgr_empid;
$$ Dept\_{no} \ \ \circledcirc Mgr\_{empid} $$
$$ Mgr\_{empid} \ \ \circledcirc Mgr\_{phone}$$
these dependencies together imply that,
$$ Dept\_{no} \ \ \circledcirc  Mgr\_{phone}$$
However it's impossible to specify all possible functional dependencies for a given situation. 

The set of all dependencies that include $F$ as well as all dependencies that can be inferred from F called the **closure of F**. It is done by $F^{+}$. 

Ex:
$$
F = \lbrace Empid \circledcirc \lbrace Ename,Bdate, Address,Dnumber \rbrace , \ Dnumber \circledcirc \lbrace Dname,Mgrid \rbrace \rbrace
$$
Inferred dependencies
$$
Empid \circledcirc \lbrace Dname, Mgrid \rbrace
$$
$$
Dnumber \circledcirc Dname
$$
#### Armstrong Axioms
Let A, B and C be subsets of the attributes of relation R. Armstrong' axioms are,
1. Reflexivity
If $B$ is a subset of $A$, then $(A \circledcirc B)$
2. Augmentation
If $(A \circledcirc B)$, then $(A,C \circledcirc B,C)$ 
3. Transitivity
If $(A \circledcirc B)$ and $(B \circledcirc C)$, then $(A \circledcirc C)$
4. Projectivity
If $(A \circledcirc BC)$, then $(A \circledcirc B)$
5. Union
If $(A \circledcirc B)$ and $(A \circledcirc C)$, then $(A \circledcirc BC)$
