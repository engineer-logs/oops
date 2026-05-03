# design a passport renewal system using Passport and Person class.
from datetime import date, datetime

# imports type hint utilities from Python’s typing module
from enum import Enum

# is importing two classes—datetime and date—from Python’s built-in datetime module.
from typing import List, Optional

# Enum is a base class used to create enumerations — a set of named constant values.


# One-to-Many realword example: Person and Passport with historical records .

# A Person can have MANY Passports over time (historical records), but each Passport belongs to exactly ONE Person.



# association example
class PassportStatus(Enum):
    ACTIVE = "active"
    EXPIRED = "expired"
    CANCELLED = "cancelled"
    LOST = "lost"
    REVOKED = "revoked"




class Passport:
    """
    Passport is an INDEPENDENT entity with its own lifecycle.
    It exists as a separate record in the database, not just an attribute.
    
    Real-world analogy: A passport booklet is a physical document 
    with its own identity, not just a number tagged to a person.
    """
    
    # Class-level counter (simulates auto-increment in database)
    _id_counter: int = 0 
    def __init__(self, passport_number: str, country: str,issue_date: date,expiry_date: date):
        Passport._id_counter += 1
        self.__id = Passport._id_counter # Assign a unique ID to each passport instance
        self.__passport_number = passport_number
        self.__country = country
        self.__issue_date = issue_date
        self.__expiry_date = expiry_date
        self.__status = PassportStatus.ACTIVE
        
        self.__owner: Optional[Person] = None  # Association to Person (initially None) 
        # Passport references Person, but Person is created separately
        # This is UNIDIRECTIONAL association (Passport knows Person)
    
    # ----- Getters (encapsulation) -----
    @property
    def id(self) -> int:
        """Database primary key - immutable, never changes"""
        return self.__id
    
    @property
    def passport_number(self) -> str:
        """Passport number - once assigned, never changes for this document"""
        return self.__passport_number
    
    @property
    def country(self) -> str:
        return self.__country
    
    @property
    def issue_date(self) -> date:
        return self.__issue_date
    
    @property
    def expiry_date(self) -> date:
        return self.__expiry_date
    
    @property
    def status(self) -> PassportStatus:
        return self.__status
    
    @property
    def owner(self) -> Optional['Person']:
        """Returns the associated Person (if any)"""
        return self.__owner
    
    @property
    def is_valid(self) -> bool:
        """Determines if the passport is currently valid based on status and expiry date"""
        today = date.today()
        return (self.__status == PassportStatus.ACTIVE and 
                self.__issue_date <= today < self.__expiry_date)
        
     # ----- Business Logic -----
     
    def link_to_person(self, person: 'Person'):
        """
            This creates a unidirectional association (Passport -> Person)
            Person can exist without a Passport, but Passport references Person
            This represents the foreign key relationship in OOP.
        """
        if self.__owner is not None:
            raise ValueError(f"Passport {self.__passport_number} already belongs to someone")
        self.__owner = person
       
    def cancel(self, reason: str = "Renewal") -> None:
        """
        When passport is renewed, the OLD one gets CANCELLED, not deleted.
        This preserves history and prevents identity fraud.
        """
        if self.__status != PassportStatus.ACTIVE:
            raise ValueError(f"Cannot cancel passport with status: {self.__status.value}")
        
        self.__status = PassportStatus.CANCELLED
        print(f"  [DB UPDATE] Passport {self.__passport_number} status → CANCELLED (Reason: {reason})")
        # Note: We UPDATE status, NOT passport_number
        # SQL equivalent: UPDATE passports SET status='cancelled' WHERE id=<id>
    
    def expire(self) -> None:
        """Marks passport as expired"""
        self.__status = PassportStatus.EXPIRED
        print(f"  [DB UPDATE] Passport {self.__passport_number} status → EXPIRED")
    
    def __str__(self) -> str:
        owner_name = self.__owner.name if self.__owner else "Unassigned"
        return (f"Passport(id={self.__id}, #{self.__passport_number}, "
                f"{self.__country}, Status: {self.__status.value}, "
                f"Owner: {owner_name})")
    
    def __repr__(self) -> str:
        return f"<Passport #{self.__passport_number}>"

class Person:
    """
    Person entity that can have MULTIPLE passports over time.
    This shows ONE-TO-MANY association.
    
    Interview talking point: "A person has MANY passports in their lifetime,
    but currently has ONE ACTIVE passport. This is both historical and temporal."
    """
    def __init__(self, name: str, date_of_birth: date, nationality: str):
        self.__name: str = name
        self.__date_of_birth: date = date_of_birth
        self.__nationality: str = nationality
        
        # KEY CONCEPT: Person maintains a collection of all passports
        self.__passports: List[Passport] = []  # One-to-Many association
        
        # Convenient reference to current active passport
        self.__current_passport: Optional[Passport] = None
    
    # ----- Getters (encapsulation) -----
    @property
    def name(self) -> str:
        return self.__name
    @property
    def date_of_birth(self) -> date:
        return self.__date_of_birth
    @property
    def nationality(self) -> str:
        return self.__nationality
    @property
    def passports(self) -> List[Passport]:
        """
        Returns a list of all passports associated with this person
        Encapsulation: Protects internal state from external modification
        Immutability: Strings, ints, tuples don't need copying (they're immutable)
        Shallow vs Deep: .copy() is shallow (new list, same objects), deepcopy() is deep (new everything)
        Performance: There's a cost, but for collections in getters, safety is usually worth it
        Python specific: In Python, everything is a reference, so returning a collection returns a reference to the mutable object.
        """
        return self.__passports.copy()  # Return a copy to prevent external modification
    @property
    def current_passport(self) -> Optional[Passport]:
        """Returns the currently active passport, if any"""
        return self.__current_passport
    
    
    # ----- Business Logic -----
    def issue_new_passport(self, passport_number: str,issue_date: date,expiry_date: date,is_renewal: bool = False) -> Passport:
        """
        Issues a NEW passport. If renewal, cancels the old one.
        - NEVER updates passport_number on existing record
        - ALWAYS creates a NEW Passport object (INSERT in DB)
        - Old passport gets CANCELLED (UPDATE status only)
        """
        # Step 1: Create brand NEW passport (DB: INSERT)
        new_passport = Passport(
            passport_number=passport_number,  # NEW unique number
            country=self.__nationality,
            issue_date=issue_date,
            expiry_date=expiry_date
        )
        print(f"  [DB INSERT] Created new passport: #{passport_number}")
        
        # Step 2: If renewal, cancel the OLD passport
        if is_renewal and self.__current_passport:
            old_passport = self.__current_passport
            print(f"  Renewal detected. Old passport: #{old_passport.passport_number}")
            old_passport.cancel(reason="Renewed")
            # Old passport stays in the list (historical record)
        
        # Step 3: Link new passport to this person (bidirectional association)
        new_passport.link_to_person(self)
        self.__passports.append(new_passport)
        self.__current_passport = new_passport
        print(f"  [DB COMMIT] Passport #{passport_number} is now active for {self.__name}\n")
        return new_passport
    

    def get_passport_history(self) -> List[Passport]:
        """Returns complete passport history (for auditing)"""
        return sorted(self.__passports, key=lambda p: p.issue_date, reverse=True)
    
    def get_active_passports(self) -> List[Passport]:
        """Returns currently active passports (usually just one)"""
        return [p for p in self.__passports if p.is_valid]
    
    def __str__(self) -> str:
        active = f"#{self.__current_passport.passport_number}" if self.__current_passport else "None"
        
        return (f"Person: {self.__name}, Nationality: {self.__nationality}, "
                f"Active Passport: {active}, "
                f"Total Passports: {len(self.__passports)}")
        
        



# ============================================
# DEMONSTRATION - The Real-World Scenario
# ============================================

def simulate_passport_lifecycle():
    """Simulates a person's passport lifecycle over time"""
    
    print("="*70)
    print("PASSPORT LIFECYCLE SIMULATION".center(70))
    print("="*70)
    
    # ----- 1. Person Born -----
    john = Person(
        name="John Doe",
        date_of_birth=date(1990, 5, 15),
        nationality="USA"
    )
    print(f"\n1️⃣  New person created: {john}")
    
    # ----- 2. First Passport Issued (2020) -----
    print("\n" + "🔹 SCENARIO 1: First passport issued".center(50))
    print("-"*50)
    
    first_passport = john.issue_new_passport(
        passport_number="US123456789",  # First number
        issue_date=date(2020, 1, 15),
        expiry_date=date(2030, 1, 14),
        is_renewal=False
    )
    
    print(f"After first passport:")
    print(f"  {john}")
    print(f"  Active passport number: {john.current_passport.passport_number}")
    print(f"  Total passports in history: {len(john.passports)}")
    
    # ----- 3. Passport Renewal (2025 - Before Expiry) -----
    print("\n" + "🔹 SCENARIO 2: Passport Renewal (THE KEY SCENARIO)".center(70))
    print("-"*70)
    
    print("Before renewal:")
    print(f"  Old passport number: {john.current_passport.passport_number}")
    print(f"  Old passport status: {john.current_passport.status.value}")
    print(f"  Old passport DB id: {john.current_passport.id}")
    
    renewed_passport = john.issue_new_passport(
        passport_number="US987654321",  # COMPLETELY DIFFERENT NUMBER!
        issue_date=date(2025, 6, 1),
        expiry_date=date(2035, 5, 31),
        is_renewal=True  # This triggers cancellation of old passport
    )
    
    print("After renewal:")
    print(f"  NEW passport number: {john.current_passport.passport_number}")
    print(f"  NEW passport DB id: {john.current_passport.id}")
    print(f"  NEW passport status: {john.current_passport.status.value}")
    print(f"  Total passports in history: {len(john.passports)}")
    
    # Let's check the old passport
    old_passport = john.passports[0]  # First in list
    print(f"\n  OLD passport still exists in history:")
    print(f"    Number: {old_passport.passport_number}")
    print(f"    Status: {old_passport.status.value}")
    print(f"    Still valid? {old_passport.is_valid}")
    
    # ----- 4. Verify Complete History -----
    print("\n" + "🔹 PASSPORT HISTORY (Audit Trail)".center(50))
    print("-"*50)
    
    for i, passport in enumerate(john.get_passport_history(), 1):
        print(f"  {i}. {passport}")
    
    # ----- 5. Database Analogy -----
    print("\n" + "🔹 DATABASE REPRESENTATION".center(50))
    print("-"*50)
    
    print("""
    Passports Table:
    ┌────┬──────────────────┬──────────┬────────────┬────────────┬───────────┐
    │ id │ passport_number  │ country  │ issue_date │ expiry_date│ status    │
    ├────┼──────────────────┼──────────┼────────────┼────────────┼───────────┤
    │ 1  │ US123456789      │ USA      │ 2020-01-15 │ 2030-01-14 │ CANCELLED │ ← UPDATED status only
    │ 2  │ US987654321      │ USA      │ 2025-06-01 │ 2035-05-31 │ ACTIVE    │ ← NEW entry
    └────┴──────────────────┴──────────┴────────────┴────────────┴───────────┘
    
    Person_Passports Table (Junction/Association):
    ┌───────────┬─────────────┬──────────────┐
    │ person_id │ passport_id │ is_current   │
    ├───────────┼─────────────┼──────────────┤
    │ 1         │ 1           │ False        │
    │ 1         │ 2           │ True         │
    └───────────┴─────────────┴──────────────┘
    
    KEY POINTS:
    ✓ Old passport #US123456789 was NOT deleted - status changed to CANCELLED
    ✓ New passport #US987654321 was INSERTED with new unique number
    ✓ Passport number was NEVER updated - it's part of the identity
    ✓ Complete history is preserved for audit trail
    ✓ Person can access both current and historical passports
    """)


# ============================================
# ADDITIONAL SCENARIOS
# ============================================

def demonstrate_advanced_scenarios():
    """Shows other real-world scenarios"""
    
    print("\n" + "="*70)
    print("ADVANCED SCENARIOS".center(70))
    print("="*70)
    
    # Scenario: Lost Passport
    print("\n🔹 LOST PASSPORT SCENARIO")
    print("-"*30)
    
    alice = Person("Alice Smith", date(1985, 3, 20), "UK")
    pass1 = alice.issue_new_passport("UK111111", date(2020, 1, 1), date(2030, 1, 1))
    
    pass1.cancel("Lost")
    print(f"  Old passport status: {pass1.status.value}")
    print(f"  Active passport: {alice.current_passport}")
    
    # Issue replacement
    pass2 = alice.issue_new_passport("UK222222", date(2023, 6, 1), date(2033, 6, 1))
    print(f"  New replacement passport: #{pass2.passport_number}")
    print(f"  Total passports for Alice: {len(alice.passports)}")
    
    # Demonstrating association types
    print("\n🔹 ASSOCIATION TYPES IN THIS DESIGN")
    print("-"*40)
    print("""
    1. ONE-TO-MANY: Person → Passport
       A person can have many passports over time.
       Implemented via List[Passport] in Person class.
    
    2. BIDIRECTIONAL NAVIGATION:
       From Person: person.current_passport → Passport
       From Passport: passport.owner → Person
       This allows navigation in both directions.
    
    3. LIFECYCLE ASSOCIATION:
       Passport life: Created → Active → Expired/Cancelled
       Never deleted - status changes only.
    """)


# ============================================
# RUN THE DEMONSTRATION
# ============================================

if __name__ == "__main__":
    simulate_passport_lifecycle()
    demonstrate_advanced_scenarios()
    
    # Interview Summary
    print("\n" + "="*70)
    print("INTERVIEW SUMMARY: Why passport number changes on renewal".center(70))
    print("="*70)
    print("""
    🎯 ANSWER TO "Does backend update or create new entry?"
    
    ❌ DOES NOT: UPDATE passport_number on existing record
       - Passport number is part of the document's identity
       - Changing it would break foreign key relationships
       - Would lose historical tracking
    
    ✅ DOES: Create NEW passport record (INSERT)
       1. Generate new passport with unique number
       2. Insert as new row in Passports table
       3. Cancel old passport (UPDATE status, NOT number)
       4. Update Person's current_passport reference
    
    🔒 WHY THIS APPROACH?
       - Security: Prevents number compromise extending to old documents
       - Audit trail: Complete history of all issued documents
       - Compliance: Meets ICAO (International Civil Aviation Organization) standards
       - Data integrity: Each passport number uniquely identifies one document
    
    🏗️ OOP PATTERNS USED:
       - Encapsulation: Private attributes with controlled access
       - Association: Objects reference each other (not inheritance)
       - State Pattern: Passport status transitions (Active→Cancelled)
       - Immutability: Passport number never changes once set
       - Temporal Data: History maintained through status, not deletion
    """)


# ============================================
# QUICK DEMO FOR ANSWERING YOUR SPECIFIC QUESTION
# ============================================

def quick_answer_demo():
    """Focused demo answering the original question"""
    
    person = Person("Demo User", date(1990, 1, 1), "India")
    
    # First passport
    pass1 = person.issue_new_passport("IN123456789", date(2020, 1, 1), date(2030, 1, 1))
    print(f"First passport DB id: {pass1.id}, Number: {pass1.passport_number}")
    
    # Renewal - Notice it's a DIFFERENT number!
    pass2 = person.issue_new_passport("IN998877665", date(2025, 1, 1), date(2035, 1, 1), 
                                      is_renewal=True)
    print(f"New passport DB id: {pass2.id}, Number: {pass2.passport_number}")
    
    print(f"\nOld passport status: {pass1.status.value}")
    print(f"Old passport number: {pass1.passport_number}")
    print(f"New passport number: {pass2.passport_number}")
    print(f"Are they different? {pass1.passport_number != pass2.passport_number}")
    print(f"Different DB records? {pass1.id != pass2.id}")
    
    # KEY POINT: pass1 still exists, just cancelled
    # pass2 is a completely new object with new ID and new number

# Uncomment to run just the quick demo
# quick_answer_demo()

'''
Q: "When a passport is renewed, does the backend update the passport number or create a new entry?"

A: "It creates a new entry. Here's why:

Domain Reality: A passport is a physical document, not just an attribute. When renewed, you get a new booklet with a new number. The old booklet is cancelled but remains in records.

Database Implementation:

New INSERT into passports table with new passport_number

UPDATE old passport's status to 'CANCELLED' (never UPDATE the number)

Old record preserved for audit trail

OOP Implementation (as shown above):

Each passport is an independent object with its own identity

Person maintains a collection (One-to-Many association)

Issuing a new passport creates a new Passport instance

The old passport's cancel() method changes only its status

Why not UPDATE?

Passport number is the natural key - changing it breaks identity

Need to track history for security/compliance

Same person might need old passport for records

This follows the principle that immutable identifiers shouldn't be updated - instead, we model the real-world lifecycle of the document."

This implementation and explanation perfectly demonstrates association in OOP while answering your backend question about data management
'''


