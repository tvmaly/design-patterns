1. Single Responsibility Principle (SRP)

Summary: A class should have only one reason to change, meaning it should have only a single, well-defined responsibility. This makes the class more focused, easier to understand, and less prone to side effects when changes are made.

Why it's important: It promotes high cohesion and low coupling. When responsibilities are separated, a change in one area of functionality (e.g., how notifications are sent) doesn't require modifying and re-testing an unrelated area (e.g., how account balances are calculated).

Financial Industry Example: An Account class should only be responsible for managing its balance. The logic for sending notifications or generating statements should be handled by separate classes.

# GOOD: Each class has a single responsibility.

class Account:
    """Manages the balance of a financial account. Its only reason to change
    is if the rules of balance management change."""
    def __init__(self, account_number, initial_balance=0):
        self.account_number = account_number
        self.balance = initial_balance

    def deposit(self, amount):
        self.balance += amount
        print(f"Deposited ${amount}. New balance: ${self.balance}")

    def withdraw(self, amount):
        if amount > self.balance:
            print("Insufficient funds.")
            return False
        self.balance -= amount
        print(f"Withdrew ${amount}. New balance: ${self.balance}")
        return True

class NotificationService:
    """Handles sending notifications. Its only reason to change is if the
    notification method (e.g., email, SMS) changes."""
    def send_transaction_alert(self, account_number, message):
        print(f"ALERT for account {account_number}: {message}")

# Client Code
account = Account("ACC123", 1000)
notifier = NotificationService()

if account.withdraw(500):
    notifier.send_transaction_alert(account.account_number, "Withdrawal of $500 processed.")


2. Open/Closed Principle (OCP)

Summary: Software entities (classes, modules, etc.) should be open for extension but closed for modification. You should be able to add new functionality without altering existing, tested code. This is often achieved using abstraction and polymorphism.

Why it's important: It prevents introducing bugs into existing, stable code. It allows for a more flexible and maintainable system where new features can be added as "plugins."

Financial Industry Example: A system for calculating transaction fees. Instead of an if/elif chain that must be modified for every new fee type, we use a strategy pattern. Each fee logic is a separate class, and new fee types can be added without changing the core processor.

from abc import ABC, abstractmethod

# The Abstraction (The "Closed" part)
class FeeStrategy(ABC):
    @abstractmethod
    def calculate_fee(self, amount):
        pass

# The Extensions (The "Open" part)
class WireTransferFee(FeeStrategy):
    def calculate_fee(self, amount):
        return 25.00  # Flat fee

class StockTradeFee(FeeStrategy):
    def calculate_fee(self, amount):
        return max(10.00, amount * 0.005) # 0.5% fee, min $10

# New functionality can be added easily without changing anything above.
class InternationalTradeFee(FeeStrategy):
    def calculate_fee(self, amount):
        return StockTradeFee().calculate_fee(amount) + 20.00 # Base fee + international charge

class TransactionProcessor:
    def process(self, amount, fee_strategy: FeeStrategy):
        fee = fee_strategy.calculate_fee(amount)
        print(f"Processing transaction of ${amount} with a fee of ${fee:.2f}")

# Client Code
processor = TransactionProcessor()
processor.process(5000, WireTransferFee())
processor.process(5000, StockTradeFee())
processor.process(5000, InternationalTradeFee()) # Using the new extension

3. Liskov Substitution Principle (LSP)

Summary: Subtypes must be substitutable for their base types without altering the correctness of the program. In other words, if a function works with a base class, it must also work with any of its subclasses without any surprises.

Why it's important: LSP ensures that inheritance hierarchies are well-designed and predictable. Violating it leads to incorrect behavior and requires adding type-checking if statements, which breaks the Open/Closed Principle.

Financial Industry Example: A FixedDepositAccount should not inherit from a generic Account if it cannot fulfill the withdraw contract. A function expecting to withdraw from any Account would break if given a FixedDepositAccount.

class Account:
    def __init__(self, balance=0):
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount

# This subclass follows LSP because it fulfills the contract
class SavingsAccount(Account):
    pass

# VIOLATION: This subclass breaks the contract of the parent
class FixedDepositAccount(Account):
    def withdraw(self, amount):
        # A fixed deposit cannot be withdrawn from, breaking the "is-a" relationship
        raise TypeError("Cannot withdraw from a Fixed Deposit account.")

def process_account_withdrawal(account: Account, amount_to_withdraw):
    print(f"Attempting to withdraw ${amount_to_withdraw} from a {type(account).__name__}")
    try:
        account.withdraw(amount_to_withdraw)
        print("Withdrawal successful.")
    except (ValueError, TypeError) as e:
        print(f"Withdrawal failed: {e}")

# Client Code
savings = SavingsAccount(1000)
fixed_deposit = FixedDepositAccount(5000)

process_account_withdrawal(savings, 500)       # Behaves as expected
process_account_withdrawal(fixed_deposit, 500) # Throws an unexpected TypeError, violating LSP

A better design would use composition or have separate interfaces like IWithdrawable and IDepositable.
4. Interface Segregation Principle (ISP)

Summary: No client should be forced to depend on methods it does not use. It's better to have many small, specific interfaces ("roles") than one large, general-purpose ("fat") interface.

Why it's important: It prevents "fat" classes that are hard to maintain. It reduces coupling, as clients only need to know about the methods that are relevant to them.

Financial Industry Example: Instead of a single, large IFinancialInstrument interface with methods for trading, paying dividends, and earning interest, we should create smaller, role-based interfaces.

from abc import ABC, abstractmethod

# GOOD: Segregated interfaces based on roles

class ITradable(ABC):
    @abstractmethod
    def buy(self, quantity): pass
    
    @abstractmethod
    def sell(self, quantity): pass

class IDividendPaying(ABC):
    @abstractmethod
    def collect_dividend(self): pass

class ICouponBearing(ABC):
    @abstractmethod
    def collect_coupon(self): pass

# Stock implements only the interfaces it needs
class Stock(ITradable, IDividendPaying):
    def buy(self, quantity): print(f"Buying {quantity} shares of stock.")
    def sell(self, quantity): print(f"Selling {quantity} shares of stock.")
    def collect_dividend(self): print("Collecting dividend payment.")

# Bond implements only the interfaces it needs
class Bond(ITradable, ICouponBearing):
    def buy(self, quantity): print(f"Buying {quantity} bonds.")
    def sell(self, quantity): print(f"Selling {quantity} bonds.")
    def collect_coupon(self): print("Collecting coupon interest.")

# Client Code
my_stock = Stock()
my_stock.buy(100)
my_stock.collect_dividend()
# my_stock.collect_coupon() # This would correctly cause an AttributeError

my_bond = Bond()
my_bond.buy(10)
my_bond.collect_coupon()



5. Dependency Inversion Principle (DIP)

Summary: High-level modules should not depend on low-level modules; both should depend on abstractions. Furthermore, abstractions should not depend on details; details should depend on abstractions. This is often achieved through Dependency Injection.

Why it's important: It decouples high-level policy code from low-level implementation details. This makes the system vastly more flexible and testable, as you can easily swap out low-level components (e.g., replace a real database with a mock for a unit test).

Financial Industry Example: A high-level ReportGenerator should not depend directly on a concrete MySqlDatabase. Instead, it should depend on an abstract IDataSource interface. The specific database implementation can then be "injected" at runtime.


from abc import ABC, abstractmethod

# The Abstraction
class IDataSource(ABC):
    @abstractmethod
    def get_transactions(self, account_id):
        pass

# Low-level modules (the details)
class DatabaseSource(IDataSource):
    def get_transactions(self, account_id):
        # In a real app, this would query a database
        print(f"Fetching transactions for {account_id} from Production Database...")
        return [("BUY", "AAPL", 100), ("SELL", "GOOG", 50)]

class MockDataSource(IDataSource): # A different detail for testing
    def get_transactions(self, account_id):
        print(f"Fetching transactions for {account_id} from Mock Test Data...")
        return [("TEST_BUY", "TSLA", 10)]

# High-level module
class ReportGenerator:
    def __init__(self, data_source: IDataSource):
        # Depends on the abstraction, not the concrete class
        self.data_source = data_source

    def generate_report(self, account_id):
        transactions = self.data_source.get_transactions(account_id)
        print("--- Generating Report ---")
        for tx in transactions:
            print(tx)
        print("--- End of Report ---")

# Client Code
# Injecting the production database
db_source = DatabaseSource()
prod_reporter = ReportGenerator(db_source)
prod_reporter.generate_report("ACC456")

print("\n" + "="*20 + "\n")

# Injecting the mock data source for a test run
mock_source = MockDataSource()
test_reporter = ReportGenerator(mock_source)
test_reporter.generate_report("TEST_ACC")
