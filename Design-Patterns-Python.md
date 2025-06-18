Creational Patterns

These patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.
1. Abstract Factory

Summary: Provides an interface for creating families of related or dependent objects without specifying their concrete classes. It's like a factory of factories.

Finance Example: Create different types of financial products for retail vs. corporate customers. A RetailFactory produces savings accounts and personal loans, while a CorporateFactory produces business accounts and commercial loans, ensuring products within a family are compatible.

from abc import ABC, abstractmethod

# Abstract Products
class Account(ABC):
    @abstractmethod
    def get_info(self): pass

class Loan(ABC):
    @abstractmethod
    def get_info(self): pass

# Concrete Products for Retail
class SavingsAccount(Account):
    def get_info(self): return "Retail Savings Account"

class PersonalLoan(Loan):
    def get_info(self): return "Retail Personal Loan"

# Concrete Products for Corporate
class BusinessAccount(Account):
    def get_info(self): return "Corporate Business Account"

class CommercialLoan(Loan):
    def get_info(self): return "Corporate Commercial Loan"

# Abstract Factory
class BankingFactory(ABC):
    @abstractmethod
    def create_account(self) -> Account: pass

    @abstractmethod
    def create_loan(self) -> Loan: pass

# Concrete Factories
class RetailBankingFactory(BankingFactory):
    def create_account(self): return SavingsAccount()
    def create_loan(self): return PersonalLoan()

class CorporateBankingFactory(BankingFactory):
    def create_account(self): return BusinessAccount()
    def create_loan(self): return CommercialLoan()

# Client Code
retail_factory = RetailBankingFactory()
savings = retail_factory.create_account()
print(savings.get_info()) # Output: Retail Savings Account

2. Builder

Summary: Separates the construction of a complex object from its representation, so that the same construction process can create different representations. It's useful for creating objects with many optional components or configurations.

Finance Example: Construct a complex financial derivative contract step-by-step. The Director can use the same Builder to create different standard contract types (e.g., a standard call option).

class Derivative:
    def __init__(self):
        self.underlying = None
        self.strike_price = 0
        self.expiration = None

    def __str__(self):
        return f"Derivative on {self.underlying} @ ${self.strike_price} expiring {self.expiration}"

class DerivativeBuilder:
    def __init__(self):
        self.derivative = Derivative()

    def set_underlying(self, asset):
        self.derivative.underlying = asset
        return self

    def set_strike_price(self, price):
        self.derivative.strike_price = price
        return self

    def set_expiration(self, date):
        self.derivative.expiration = date
        return self

    def get_product(self):
        return self.derivative

# Client Code
builder = DerivativeBuilder()
call_option = builder.set_underlying("AAPL").set_strike_price(150).set_expiration("2024-12-20").get_product()
print(call_option) # Output: Derivative on AAPL @ $150 expiring 2024-12-20


3. Factory Method

Summary: Defines an interface for creating an object, but lets subclasses alter the type of objects that will be created. It delegates the instantiation logic to child classes.

Finance Example: A Bank superclass provides a method to open an account, but subclasses like CommunityBank or InvestmentBank decide which specific Account type (CheckingAccount, BrokerageAccount) to create.

from abc import ABC, abstractmethod

class Account(ABC):
    @abstractmethod
    def type(self): pass

class CheckingAccount(Account):
    def type(self): return "Checking Account"

class BrokerageAccount(Account):
    def type(self): return "Brokerage Account"

class Bank(ABC):
    @abstractmethod
    def create_account(self): pass

    def open_account(self):
        account = self.create_account()
        print(f"Created a new {account.type()}")

class InvestmentBank(Bank):
    def create_account(self):
        return BrokerageAccount()

# Client Code
inv_bank = InvestmentBank()
inv_bank.open_account() # Output: Created a new Brokerage Account


4. Prototype

Summary: Creates new objects by copying an existing object, known as a prototype. This is useful when the cost of creating an object from scratch is more expensive than cloning it.

Finance Example: Create new loan agreements by cloning a standard template and then modifying the specific details like borrower and amount, avoiding the need to reconstruct the entire legal document each time.

import copy

class LoanAgreement:
    def __init__(self, borrower, amount, standard_terms):
        self.borrower = borrower
        self.amount = amount
        self.standard_terms = standard_terms

    def clone(self):
        return copy.deepcopy(self)

    def __str__(self):
        return f"Loan for {self.borrower} of ${self.amount}"

# Create a standard prototype
standard_loan_terms = "Standard terms and conditions..."
loan_prototype = LoanAgreement("", 0, standard_loan_terms)

# Client Code
loan_for_alice = loan_prototype.clone()
loan_for_alice.borrower = "Alice"
loan_for_alice.amount = 50000
print(loan_for_alice) # Output: Loan for Alice of $50000

loan_for_bob = loan_prototype.clone()
loan_for_bob.borrower = "Bob"
loan_for_bob.amount = 120000
print(loan_for_bob) # Output: Loan for Bob of $120000


5. Singleton

Summary: Ensures a class has only one instance and provides a global point of access to it. The provided repository includes a thread-safe implementation using a lock (DoubleChecking.py), which is crucial for concurrent applications.

Finance Example: Manage a single connection to a real-time market data feed. All parts of the application that need market data will use the same instance, preventing multiple costly connections.

import threading

class MarketDataFeed:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if not cls._instance:
            with cls._lock:
                if not cls._instance:
                    cls._instance = super().__new__(cls)
                    print("Connecting to market data feed...")
        return cls._instance

    def get_price(self, ticker):
        # In a real app, this would fetch live data
        return 150.25 if ticker == "AAPL" else 3000.50

# Client Code
feed1 = MarketDataFeed()
feed2 = MarketDataFeed()

print(f"feed1 is feed2: {feed1 is feed2}") # Output: True
print(f"Price of AAPL: {feed1.get_price('AAPL')}")


Structural Patterns

These patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

1. Adapter

Summary: Allows objects with incompatible interfaces to collaborate. It acts as a wrapper, translating calls from one interface to another.

Finance Example: Integrate a new, third-party payment gateway into an existing e-commerce system. The Adapter makes the new gateway's API (make_payment) compatible with the system's expected process_payment method.

class PaymentProcessor:
    def process_payment(self, amount):
        print(f"Processing payment of ${amount} via standard system.")

class NewGateway:
    def make_payment(self, value):
        print(f"Making payment of ${value} via New Gateway.")

class GatewayAdapter(PaymentProcessor):
    def __init__(self, gateway: NewGateway):
        self._gateway = gateway

    def process_payment(self, amount):
        # Adapting the call from process_payment(amount) to make_payment(value)
        self._gateway.make_payment(amount)

# Client Code
new_gateway = NewGateway()
adapter = GatewayAdapter(new_gateway)
adapter.process_payment(100) # Output: Making payment of $100 via New Gateway.


2. Bridge

Summary: Decouples an abstraction from its implementation so that the two can vary independently. It's like building a bridge between an abstract class and its implementation details.

Finance Example: Separate financial reports (Abstraction) from their export formats (Implementation). A PortfolioReport can be exported as a PDF or CSV without either class knowing the specifics of the other.


from abc import ABC, abstractmethod

# Implementation Interface
class IExportFormat(ABC):
    @abstractmethod
    def export(self, data): pass

# Concrete Implementations
class PDFFormat(IExportFormat):
    def export(self, data): print(f"Exporting '{data}' to PDF.")

class CSVFormat(IExportFormat):
    def export(self, data): print(f"Exporting '{data}' to CSV.")

# Abstraction
class Report(ABC):
    def __init__(self, exporter: IExportFormat):
        self._exporter = exporter
    @abstractmethod
    def generate(self): pass

# Refined Abstraction
class PortfolioReport(Report):
    def generate(self):
        data = "Portfolio data with stocks and bonds"
        self._exporter.export(data)

# Client Code
pdf_portfolio = PortfolioReport(PDFFormat())
pdf_portfolio.generate() # Output: Exporting 'Portfolio data...' to PDF.

csv_portfolio = PortfolioReport(CSVFormat())
csv_portfolio.generate() # Output: Exporting 'Portfolio data...' to CSV.

3. Composite

Summary: Composes objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.

Finance Example: Model a financial portfolio that can contain both individual assets (like a Stock) and other sub-portfolios (Portfolio). You can calculate the total value of the entire structure uniformly.

from abc import ABC, abstractmethod

class FinancialAsset(ABC):
    @abstractmethod
    def get_value(self): pass

class Stock(FinancialAsset): # Leaf
    def __init__(self, ticker, value):
        self.ticker = ticker
        self.value = value
    def get_value(self): return self.value

class Portfolio(FinancialAsset): # Composite
    def __init__(self, name):
        self.name = name
        self._children = []
    def add(self, asset: FinancialAsset): self._children.append(asset)
    def get_value(self):
        return sum(child.get_value() for child in self._children)

# Client Code
tech_portfolio = Portfolio("Tech Stocks")
tech_portfolio.add(Stock("AAPL", 1000))
tech_portfolio.add(Stock("GOOG", 2000))

main_portfolio = Portfolio("Main Portfolio")
main_portfolio.add(tech_portfolio)
main_portfolio.add(Stock("JPM", 500)) # A non-tech stock

print(f"Value of main portfolio: ${main_portfolio.get_value()}") # Output: 3500

4. Decorator

Summary: Attaches new responsibilities to an object dynamically by placing it inside a special wrapper object. This provides a flexible alternative to subclassing for extending functionality.

Finance Example: Add optional features like overdraft protection or rewards to a basic bank account without creating a complex class hierarchy.

from abc import ABC, abstractmethod

class IAccount(ABC):
    @abstractmethod
    def get_balance(self): pass

class BasicAccount(IAccount):
    def __init__(self, balance=1000):
        self._balance = balance
    def get_balance(self): return self._balance

class AccountDecorator(IAccount):
    def __init__(self, account: IAccount):
        self._account = account
    def get_balance(self): return self._account.get_balance()

class OverdraftProtectionDecorator(AccountDecorator):
    def get_balance(self):
        return self._account.get_balance() + 500 # Adds a credit line

# Client Code
my_account = BasicAccount(100)
print(f"Basic balance: {my_account.get_balance()}") # Output: 100

my_account_with_overdraft = OverdraftProtectionDecorator(my_account)
print(f"Balance with overdraft: {my_account_with_overdraft.get_balance()}") # Output: 600

5. Facade

Summary: Provides a simplified, high-level interface to a complex subsystem of classes. It hides the system's complexity and makes it easier to use.

Finance Example: Create a simple interface for executing a stock trade, which hides the complex underlying subsystems for compliance checks, order routing, and settlement.

# Complex Subsystems
class Compliance:
    def check(self, trade): print(f"Compliance check for {trade} passed.")
class OrderRouter:
    def route(self, trade): print(f"Routing {trade} to exchange.")
class Settlement:
    def settle(self, trade): print(f"Settling {trade}.")

# Facade
class TradingFacade:
    def __init__(self):
        self._compliance = Compliance()
        self._router = OrderRouter()
        self._settlement = Settlement()

    def execute_trade(self, trade_details):
        print(f"--- Executing Trade: {trade_details} ---")
        self._compliance.check(trade_details)
        self._router.route(trade_details)
        self._settlement.settle(trade_details)
        print("--- Trade Complete ---")

# Client Code
facade = TradingFacade()
facade.execute_trade("BUY 100 AAPL")

6. Flyweight

Summary: Lets you fit more objects into the available amount of RAM by sharing common parts of state between multiple objects instead of keeping all of the data in each object.

Finance Example: Manage thousands of Stock objects. The intrinsic, shared state is the company name and ticker symbol. The extrinsic, unique state is the price, which changes.

class StockFlyweight: # The flyweight with intrinsic state
    def __init__(self, ticker, company):
        self.ticker = ticker
        self.company = company

class StockFactory:
    _stocks = {}
    def get_stock(self, ticker, company):
        if ticker not in self._stocks:
            self._stocks[ticker] = StockFlyweight(ticker, company)
        return self._stocks[ticker]

class PortfolioStock: # The context object with extrinsic state
    def __init__(self, flyweight: StockFlyweight, current_price):
        self.flyweight = flyweight
        self.current_price = current_price
    def display(self):
        print(f"{self.flyweight.company} ({self.flyweight.ticker}): ${self.current_price}")

# Client Code
factory = StockFactory()
stock1 = PortfolioStock(factory.get_stock("AAPL", "Apple Inc."), 150.25)
stock2 = PortfolioStock(factory.get_stock("AAPL", "Apple Inc."), 150.25) # Price could be different

stock1.display()
print(f"Objects are the same: {stock1.flyweight is stock2.flyweight}") # Output: True


7. Proxy

Summary: Provides a surrogate or placeholder for another object to control access to it. It can be used for lazy initialization (creating an object only when needed), access control, or remote communication.

Finance Example: A protection proxy that restricts access to sensitive account operations based on user roles. A junior teller can view a balance but cannot approve a large withdrawal.

from abc import ABC, abstractmethod

class IAccount(ABC):
    @abstractmethod
    def withdraw(self, amount): pass
    @abstractmethod
    def get_balance(self): pass

class RealAccount(IAccount):
    def __init__(self, balance=50000): self._balance = balance
    def withdraw(self, amount): self._balance -= amount; print(f"Withdrew ${amount}")
    def get_balance(self): return self._balance

class AccountProtectionProxy(IAccount):
    def __init__(self, real_account: IAccount, user_role: str):
        self._real_account = real_account
        self._user_role = user_role

    def get_balance(self): return self._real_account.get_balance()
    def withdraw(self, amount):
        if self._user_role == "manager" or amount <= 500:
            self._real_account.withdraw(amount)
        else:
            print("Access Denied: Withdrawal amount too high for your role.")

# Client Code
real_account = RealAccount()
junior_proxy = AccountProtectionProxy(real_account, "junior_teller")
manager_proxy = AccountProtectionProxy(real_account, "manager")

junior_proxy.withdraw(1000) # Output: Access Denied...
manager_proxy.withdraw(1000) # Output: Withdrew $1000


Behavioral Patterns

These patterns are concerned with algorithms and the assignment of responsibilities between objects.
1. Chain of Responsibility

Summary: Lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

Finance Example: Process an expense approval request. The request passes from a Clerk to a Manager to a Director, each handling it only if it falls within their approval limit.

from abc import ABC, abstractmethod

class Approver(ABC):
    def __init__(self, successor=None): self._successor = successor
    @abstractmethod
    def process_request(self, amount): pass

class Clerk(Approver):
    def process_request(self, amount):
        if amount <= 100: print(f"Clerk approved ${amount}")
        elif self._successor: self._successor.process_request(amount)

class Manager(Approver):
    def process_request(self, amount):
        if amount <= 1000: print(f"Manager approved ${amount}")
        elif self._successor: self._successor.process_request(amount)

class Director(Approver):
    def process_request(self, amount):
        if amount <= 10000: print(f"Director approved ${amount}")
        else: print(f"Request for ${amount} requires board approval.")

# Client Code: Build the chain
approval_chain = Clerk(Manager(Director()))
approval_chain.process_request(50)    # Clerk handles
approval_chain.process_request(500)   # Manager handles
approval_chain.process_request(5000)  # Director handles


2. Command

Summary: Turns a request into a stand-alone object that contains all information about the request. This lets you parameterize methods with different requests, delay or queue a request's execution, and support undoable operations.

Finance Example: Encapsulate stock buy and sell orders as objects. An Agent (invoker) can take these Order objects and execute them, without knowing the specifics of buying or selling.

from abc import ABC, abstractmethod

class Order(ABC): # Command
    @abstractmethod
    def execute(self): pass

class StockBroker: # Receiver
    def buy(self, ticker, quantity): print(f"Buying {quantity} of {ticker}")
    def sell(self, ticker, quantity): print(f"Selling {quantity} of {ticker}")

class BuyStockOrder(Order): # Concrete Command
    def __init__(self, broker: StockBroker, ticker: str, quantity: int):
        self.broker, self.ticker, self.quantity = broker, ticker, quantity
    def execute(self): self.broker.buy(self.ticker, self.quantity)

class Agent: # Invoker
    def place_order(self, order: Order): order.execute()

# Client Code
broker = StockBroker()
buy_order = BuyStockOrder(broker, "MSFT", 50)
agent = Agent()
agent.place_order(buy_order) # Output: Buying 50 of MSFT


3. Interpreter

Summary: Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language. Best for simple languages.

Finance Example: Create a simple interpreter to evaluate rules for an automated trading strategy, such as price > 100 AND volume > 10000.

from abc import ABC, abstractmethod

class IExpression(ABC):
    @abstractmethod
    def interpret(self, context): pass

class Price(IExpression):
    def interpret(self, context): return context['price']

class And(IExpression):
    def __init__(self, left, right): self.left, self.right = left, right
    def interpret(self, context): return self.left.interpret(context) and self.right.interpret(context)

class GreaterThan(IExpression):
    def __init__(self, expr, value): self.expr, self.value = expr, value
    def interpret(self, context): return self.expr.interpret(context) > self.value

# Market context for a stock
market_context = {'price': 150, 'volume': 20000}

# Rule: price > 100 AND price < 200
rule = And(GreaterThan(Price(), 100), GreaterThan(Price(), 200)) # Note: second part is false
print(f"Rule evaluation: {rule.interpret(market_context)}") # Output: False


4. Iterator

Summary: Provides a way to access the elements of an aggregate object (e.g., a list or collection) sequentially without exposing its underlying representation.

Finance Example: Iterate through a list of transactions in a bank account. The client code can loop through transactions the same way, whether they are stored in a simple list or a more complex data structure.

import collections.abc

class Account:
    def __init__(self):
        # Transactions could be a list, a database cursor, etc.
        self._transactions = [-50, 200, -20, 1000]

    def __iter__(self):
        return iter(self._transactions) # Python's built-in iter makes this easy

# Client Code
my_account = Account()
print("Transactions:")
for tx in my_account:
    print(tx)

5. Mediator

Summary: Defines an object that encapsulates how a set of objects interact. It promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.

Finance Example: A stock exchange acts as a Mediator. Traders (Colleagues) place buy or sell orders with the exchange, which then matches them. Traders don't need to know about each other directly.


from abc import ABC, abstractmethod

class IExchange(ABC): # Mediator
    @abstractmethod
    def match_order(self, order, trader): pass

class Trader: # Colleague
    def __init__(self, name, exchange: IExchange):
        self.name, self.exchange = name, exchange
    def place_order(self, order):
        print(f"{self.name} places order: {order}")
        self.exchange.match_order(order, self)

class NYStockExchange(IExchange): # Concrete Mediator
    def match_order(self, order, trader):
        print(f"NYSE matching order '{order}' for {trader.name}")
        # In a real system, complex matching logic would go here.

# Client Code
nyse = NYStockExchange()
trader1 = Trader("Goldman Sachs", nyse)
trader2 = Trader("Morgan Stanley", nyse)
trader1.place_order("BUY 1000 AAPL")


6. Memento

Summary: Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later. It enables undo/redo functionality.

Finance Example: Save the state of a complex trade entry form. If the user makes a mistake, they can undo their changes back to a previously saved state.

class TradeTicket: # Originator
    def __init__(self, state): self._state = state
    def set_state(self, state): self._state = state
    def get_state(self): return self._state
    def save_to_memento(self): return Memento(self._state)
    def restore_from_memento(self, memento): self._state = memento.get_state()

class Memento:
    def __init__(self, state): self._state = state
    def get_state(self): return self._state

class History: # Caretaker
    def __init__(self): self._mementos = []
    def add(self, memento): self._mementos.append(memento)
    def get(self, index): return self._mementos[index]

# Client Code
history = History()
ticket = TradeTicket({"ticker": "GOOG", "quantity": 100})
history.add(ticket.save_to_memento()) # Save initial state

ticket.set_state({"ticker": "GOOG", "quantity": 150, "limit": 3000}) # User makes a change
print(f"Current state: {ticket.get_state()}")

ticket.restore_from_memento(history.get(0)) # Undo
print(f"Restored state: {ticket.get_state()}")


7. Observer (Pub/Sub)

Summary: Defines a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically. The repository also shows the Publisher-Subscriber variation, which is functionally similar but often uses a central broker.

Finance Example: When a stock's price (Subject) changes, notify all interested observers, such as a trader's dashboard, a risk management system, and an alerting service.

class Stock: # Subject (Publisher)
    def __init__(self, ticker):
        self.ticker = ticker
        self._price = 0
        self._observers = []
    def attach(self, observer): self._observers.append(observer)
    def set_price(self, price):
        self._price = price
        self._notify()
    def _notify(self):
        for observer in self._observers:
            observer.update(self.ticker, self._price)

class Dashboard: # Observer (Subscriber)
    def update(self, ticker, price):
        print(f"Dashboard: {ticker} is now ${price}")

class Alerter: # Observer (Subscriber)
    def update(self, ticker, price):
        if price > 150:
            print(f"ALERT: {ticker} has exceeded $150!")

# Client Code
aapl_stock = Stock("AAPL")
dashboard = Dashboard()
alerter = Alerter()

aapl_stock.attach(dashboard)
aapl_stock.attach(alerter)

aapl_stock.set_price(149.5) # Dashboard is notified
aapl_stock.set_price(151.0) # Both are notified, and alert triggers

8. State

Summary: Allows an object to alter its behavior when its internal state changes. The object appears to change its class.

Finance Example: Model the lifecycle of a loan application. The available actions (approve, reject) and their outcomes depend on the current state (Pending, Approved, Rejected).

from __future__ import annotations
from abc import ABC, abstractmethod

class LoanApplication: # Context
    _state = None
    def __init__(self, state: State): self.transition_to(state)
    def transition_to(self, state: State): self._state = state; self._state.context = self
    def approve(self): self._state.handle_approval()
    def reject(self): self._state.handle_rejection()

class State(ABC): # State interface
    @property
    def context(self) -> LoanApplication: return self._context
    @context.setter
    def context(self, context: LoanApplication) -> None: self._context = context
    @abstractmethod
    def handle_approval(self): pass
    @abstractmethod
    def handle_rejection(self): pass

class PendingState(State): # Concrete State
    def handle_approval(self): print("Loan approved."); self.context.transition_to(ApprovedState())
    def handle_rejection(self): print("Loan rejected."); self.context.transition_to(RejectedState())

class ApprovedState(State):
    def handle_approval(self): print("Loan is already approved.")
    def handle_rejection(self): print("Cannot reject an approved loan.")

class RejectedState(State):
    def handle_approval(self): print("Cannot approve a rejected loan.")
    def handle_rejection(self): print("Loan is already rejected.")

# Client Code
application = LoanApplication(PendingState())
application.approve()   # Transitions to ApprovedState
application.reject()    # Action has no effect in ApprovedState

9. Strategy

Summary: Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

Finance Example: Choose a risk calculation strategy based on the type of asset or market conditions. A Portfolio context can use a SimpleRiskStrategy or a more complex ValueAtRiskStrategy to calculate its risk profile.

from abc import ABC, abstractmethod

class RiskStrategy(ABC):
    @abstractmethod
    def calculate(self, assets): pass

class SimpleRiskStrategy(RiskStrategy):
    def calculate(self, assets):
        print("Calculating risk with Simple moving average.")
        return 0.25 # Simplified result

class ValueAtRiskStrategy(RiskStrategy):
    def calculate(self, assets):
        print("Calculating risk with complex Value-at-Risk model.")
        return 0.40 # Simplified result

class Portfolio:
    def __init__(self, assets, strategy: RiskStrategy):
        self.assets = assets
        self.strategy = strategy
    def get_risk(self):
        return self.strategy.calculate(self.assets)

# Client Code
my_assets = ["AAPL", "GOOG"]
low_risk_portfolio = Portfolio(my_assets, SimpleRiskStrategy())
print(f"Risk: {low_risk_portfolio.get_risk()}")

high_risk_portfolio = Portfolio(my_assets, ValueAtRiskStrategy())
print(f"Risk: {high_risk_portfolio.get_risk()}")

10. Template Method

Summary: Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. It lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

Finance Example: Define a template for processing a financial transaction. The core steps (validate, execute, log) are fixed, but the specific execute logic differs for a StockTrade versus a WireTransfer.

from abc import ABC, abstractmethod

class TransactionProcessor(ABC):
    def process(self): # Template Method
        self.validate()
        self.execute_core_logic()
        self.log_transaction()

    def validate(self): print("Validating transaction...")
    def log_transaction(self): print("Logging transaction...")
    
    @abstractmethod
    def execute_core_logic(self): pass

class StockTradeProcessor(TransactionProcessor):
    def execute_core_logic(self):
        print("Executing stock trade: debiting cash, crediting stock.")

class WireTransferProcessor(TransactionProcessor):
    def execute_core_logic(self):
        print("Executing wire transfer: debiting source, crediting destination.")

# Client Code
stock_trade = StockTradeProcessor()
stock_trade.process()
print("-" * 20)
wire_transfer = WireTransferProcessor()
wire_transfer.process()


11. Visitor

Summary: Lets you define a new operation without changing the classes of the elements on which it operates. It separates an algorithm from an object structure.

Finance Example: Calculate taxes on a portfolio containing different asset types (Stock, Bond). A TaxVisitor can calculate the tax for each asset type differently, and you can easily add a new ValuationVisitor without changing the asset classes.

from abc import ABC, abstractmethod

class IAsset(ABC): # Element
    @abstractmethod
    def accept(self, visitor): pass

class IVisitor(ABC): # Visitor
    @abstractmethod
    def visit_stock(self, stock): pass
    @abstractmethod
    def visit_bond(self, bond): pass

class Stock(IAsset):
    def __init__(self, value, gains): self.value, self.gains = value, gains
    def accept(self, visitor): visitor.visit_stock(self)

class Bond(IAsset):
    def __init__(self, value, interest): self.value, self.interest = value, interest
    def accept(self, visitor): visitor.visit_bond(self)

class TaxCalculatorVisitor(IVisitor): # Concrete Visitor
    def __init__(self): self.total_tax = 0
    def visit_stock(self, stock): self.total_tax += stock.gains * 0.20 # Capital gains tax
    def visit_bond(self, bond): self.total_tax += bond.interest * 0.15 # Interest income tax

# Client Code
portfolio = [Stock(1000, 200), Bond(5000, 250)]
tax_visitor = TaxCalculatorVisitor()
for asset in portfolio:
    asset.accept(tax_visitor)

print(f"Total tax liability: ${tax_visitor.total_tax}") # (200*0.2) + (250*0.15) = 40 + 37.5 = 77.5


Architectural Patterns

These are high-level patterns that describe the overall structure and organization of a software system.
1. MVC (Model-View-Controller)

Summary: Separates an application into three interconnected components: the Model (data and business logic), the View (UI), and the Controller (handles user input and updates the Model). In the classic version found in the repo, the View directly observes the Model.

Finance Example: A simple stock ticker application. The Model holds the stock data, the View displays it, and the Controller handles user requests to fetch new data.

class StockModel:
    def __init__(self): self.data = {"AAPL": 150.0}
    def get_data(self, ticker): return self.data.get(ticker, "N/A")
    def set_data(self, ticker, price): self.data[ticker] = price; self.view.update()

class StockView:
    def update(self): print(f"View Updated: {controller.get_data_for_view('AAPL')}")

class StockController:
    def __init__(self):
        self.model = StockModel()
        self.view = StockView()
        self.model.view = self.view # Classic MVC link

    def get_data_for_view(self, ticker): return self.model.get_data(ticker)
    def update_price(self, ticker, price): self.model.set_data(ticker, price)

# Client Code
controller = StockController()
controller.update_price("AAPL", 152.50) # Output: View Updated: 152.5

2. MVP (Model-View-Presenter)

Summary: A variation of MVC where the Controller is replaced by a Presenter. The Presenter retrieves data from the Model and formats it for the View. The View is more passive and does not interact directly with the Model; it's entirely managed by the Presenter.

Finance Example: A loan calculator. The View has input fields and a display area. The Presenter takes input from the View, asks the Model to perform the calculation, and then passes the result back to the View to be displayed.

class LoanModel:
    def calculate_payment(self, amount, rate, years): return (amount * (1 + rate)) / years

class LoanView:
    def __init__(self): self.presenter = None
    def display_payment(self, payment): print(f"View: Monthly payment is ${payment:.2f}")
    def get_user_input(self): self.presenter.calculate_clicked(50000, 0.05, 5)

class LoanPresenter:
    def __init__(self, model, view):
        self.model, self.view = model, view
        self.view.presenter = self
    def calculate_clicked(self, amount, rate, years):
        result = self.model.calculate_payment(amount, rate, years)
        self.view.display_payment(result)

# Client Code
model, view = LoanModel(), LoanView()
presenter = LoanPresenter(model, view)
view.get_user_input() # User "clicks" calculate

Anti-Patterns and Other Concepts
Polymorphism (Method Overloading Anti-Pattern)

Summary: The file Basics OOPS/polymorphism.py demonstrates a common misunderstanding in Python. Unlike languages like Java or C++, Python does not support method overloading based on different argument signatures. The second definition of a method will simply overwrite the first.

Correct Financial Example: To achieve similar behavior, use default arguments or check the type of arguments.

class TransactionLogger:
    # This is how NOT to do it (as in the repo)
    # def log(self, transaction_id): ...
    # def log(self, transaction_id, message): ... # This overwrites the first one

    # Correct Pythonic way
    def log(self, transaction_id, message=None):
        log_entry = f"ID: {transaction_id}"
        if message:
            log_entry += f" - Message: {message}"
        print(f"Logging: {log_entry}")

logger = TransactionLogger()
logger.log("TX123")
logger.log("TX456", "Payment successful")


Liskov Substitution Principle (LSP) Violation

Summary: The SOLID/3_LiskovSubstitutionPrinciple.py files correctly demonstrate a classic LSP violation. A Square class that inherits from Rectangle and changes both width and height together breaks expectations. If a function works with a Rectangle, it should also work with a Square. In the example, a test that sets height and width independently and expects a certain area will fail for a Square.

Finance Example: A base Account has a withdraw method. An FDAccount (Fixed Deposit) that inherits from it cannot allow withdrawals, thus throwing an exception or doing nothing. This violates LSP because you cannot substitute an FDAccount for an Account everywhere without breaking the program. A better design would be to use composition or more granular interfaces.

class Account:
    def __init__(self, balance): self.balance = balance
    def deposit(self, amount): self.balance += amount
    def withdraw(self, amount): self.balance -= amount

class FixedDepositAccount(Account): # Violates LSP
    def withdraw(self, amount):
        raise Exception("Cannot withdraw from a fixed deposit!")

def process_withdrawal(account: Account, amount):
    print(f"Attempting to withdraw ${amount}")
    try:
        account.withdraw(amount)
        print("Success!")
    except Exception as e:
        print(f"Failed: {e}")

# Client Code
regular_ac = Account(1000)
fd_ac = FixedDepositAccount(5000)

process_withdrawal(regular_ac, 100) # Works fine
process_withdrawal(fd_ac, 100)      # Fails, breaking the contract of the base class


