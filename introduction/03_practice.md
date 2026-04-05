## Practice (Classes and Objects)

You are tasked with designing a class **Employee** that stores and displays information about employees in a company.
The class must have the following:

---
### Attributes:
* `name (string)` : Stores the name of the employee
* `employeeId (int)` : Stores the unique employee ID
* `department (string)` : Stores the department (e.g., Engineering, HR )
* `salary (float)` : Stores the employee’s salary
* `isActive (boolean)` : Indicates whether the employee is currently active 

---

### Methods:

* `setDetails(String name, int employeeId, String department, float salary, boolean isActive)`
  Initializes all employee attributes.

* `displayDetails()`
  Prints employee details in a clean format 
* `giveRaise(float amount)`
  Increases the salary by the given amount
* `deactivateEmployee()`
  Marks the employee as inactive 

---

### Example 

**Input:**
Name - "Rahul"
Employee ID - 1001
Department - "Engineering"
Salary - 75000
Active - True

**Output:**

```
Name : Rahul
Employee ID : 1001
Department : Engineering
Salary : 75000
Status : Active
```
---

### Explanation:

An `Employee` object is created in the main class.
The `setDetails()` method initializes all attributes.
The `displayDetails()` method prints employee information.
`giveRaise()` and `deactivateEmployee()` simulate real-world corporate operations.

---

## Solution:

```python
# Your code goes here
class Employee:
    def __init__(self):
        self.name = ""
        self.employeeId = 0
        self.department = ""
        self.salary = 0.0
        self.isActive = True
    
    def setDetails(self, name, employeeId, department, salary, isActive):
        self.name = name
        self.employeeId = employeeId
        self.department = department
        self.salary = salary
        self.isActive = isActive

    def giveRaise(self, amount):
        self.salary += amount

    def deactivateEmployee(self):
        self.isActive = False

    def displayDetails(self):
        status = "Active" if self.isActive else "Inactive"
        print(f"Name : {self.name}")
        print(f"Employee ID : {self.employeeId}")
        print(f"Department : {self.department}")
        print(f"Salary : {int(self.salary)}")
        print(f"Status : {status}")
```
