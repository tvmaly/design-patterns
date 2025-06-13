A list of design patterns in Go with code examples.

This can be used as context with large language models to help with Go related design questions.  

This documentation is licensed under the MIT license.
Copyright Tyson Maly 2025.

### **Creational Patterns**

These patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.

#### 1. Abstract Factory

**Summary:** Provides an interface for creating families of related or dependent objects without specifying their concrete classes. It's like a factory that produces other factories.

**Financial Example:** A factory for creating different types of financial report generators (e.g., PDF or CSV), where each report type has a header and a body.

```go
package main

import "fmt"

// Abstract Products
type IReportHeader interface{ Content() string }
type IReportBody   interface{ Content() string }

// Concrete Products for PDF
type PdfHeader struct{}
func (h PdfHeader) Content() string { return "PDF Header" }
type PdfBody struct{}
func (b PdfBody) Content() string { return "PDF Body: Transaction List" }

// Concrete Products for CSV
type CsvHeader struct{}
func (h CsvHeader) Content() string { return "Ticker,Price,Quantity" }
type CsvBody struct{}
func (b CsvBody) Content() string { return "AAPL,150.00,10\nGOOG,2800.50,5" }

// Abstract Factory
type IReportFactory interface {
	CreateHeader() IReportHeader
	CreateBody() IReportBody
}

// Concrete Factories
type PdfReportFactory struct{}
func (f PdfReportFactory) CreateHeader() IReportHeader { return PdfHeader{} }
func (f PdfReportFactory) CreateBody() IReportBody   { return PdfBody{} }

type CsvReportFactory struct{}
func (f CsvReportFactory) CreateHeader() IReportHeader { return CsvHeader{} }
func (f CsvReportFactory) CreateBody() IReportBody   { return CsvBody{} }

// Client
func main() {
	pdfFactory := PdfReportFactory{}
	fmt.Println(pdfFactory.CreateHeader().Content())
	fmt.Println(pdfFactory.CreateBody().Content())

	fmt.Println("---")

	csvFactory := CsvReportFactory{}
	fmt.Println(csvFactory.CreateHeader().Content())
	fmt.Println(csvFactory.CreateBody().Content())
}

```

#### 2. Builder

**Summary:** Separates the construction of a complex object from its representation, so the same construction process can create different representations. It's useful for creating objects with many optional components or configurations.

**Financial Example:** Building a complex Trade object with multiple optional fields.

```go
package main

import "fmt"

// The complex object (Product)
type Trade struct {
	Ticker    string
	Quantity  int
	Price     float64
	IsBuy     bool
	OrderType string // e.g., "Market", "Limit"
}

// The Builder interface
type TradeBuilder interface {
	SetTicker(string) TradeBuilder
	SetQuantity(int) TradeBuilder
	SetPrice(float64) TradeBuilder
	SetSide(bool) TradeBuilder
	SetOrderType(string) TradeBuilder
	Build() Trade
}

// The Concrete Builder
type concreteTradeBuilder struct {
	trade Trade
}

func NewTradeBuilder() TradeBuilder {
	return &concreteTradeBuilder{trade: Trade{OrderType: "Market", IsBuy: true}} // Default values
}

func (b *concreteTradeBuilder) SetTicker(t string) TradeBuilder    { b.trade.Ticker = t; return b }
func (b *concreteTradeBuilder) SetQuantity(q int) TradeBuilder     { b.trade.Quantity = q; return b }
func (b *concreteTradeBuilder) SetPrice(p float64) TradeBuilder    { b.trade.Price = p; return b }
func (b *concreteTradeBuilder) SetSide(isBuy bool) TradeBuilder    { b.trade.IsBuy = isBuy; return b }
func (b *concreteTradeBuilder) SetOrderType(ot string) TradeBuilder { b.trade.OrderType = ot; return b }
func (b *concreteTradeBuilder) Build() Trade                         { return b.trade }

// Client
func main() {
	builder := NewTradeBuilder()
	trade := builder.SetTicker("MSFT").SetQuantity(100).SetPrice(305.50).Build()
	fmt.Printf("Constructed Trade: %+v\n", trade)
}

```

#### 3. Factory Method

**Summary:** Defines an interface for creating an object, but lets subclasses alter the type of objects that will be created.

**Financial Example:** A base PaymentProcessor that defines a method to create a payment, with concrete subclasses for processing credit cards vs. bank transfers.

```go
package main

import "fmt"

// The Product interface
type Payment interface {
	Process(amount float64)
}

// Concrete Products
type CreditCardPayment struct{}
func (p *CreditCardPayment) Process(amount float64) { fmt.Printf("Processing %.2f via Credit Card.\n", amount) }

type BankTransferPayment struct{}
func (p *BankTransferPayment) Process(amount float64) { fmt.Printf("Processing %.2f via Bank Transfer.\n", amount) }

// The Creator (Factory) interface
type PaymentProcessor interface {
	CreatePayment() Payment
}

// Concrete Creators
type CreditCardProcessor struct{}
func (p *CreditCardProcessor) CreatePayment() Payment { return &CreditCardPayment{} }

type BankTransferProcessor struct{}
func (p *BankTransferProcessor) CreatePayment() Payment { return &BankTransferPayment{} }

// Client
func main() {
	ccProcessor := &CreditCardProcessor{}
	payment1 := ccProcessor.CreatePayment()
	payment1.Process(100.50)

	btProcessor := &BankTransferProcessor{}
	payment2 := btProcessor.CreatePayment()
	payment2.Process(5000.75)
}
```

#### 4. Prototype

**Summary:** Creates new objects by copying an existing object, known as the prototype. This is useful when the cost of creating an object is more expensive than cloning it.

**Financial Example:** Creating new financial instruments (like bonds) that share many common properties by cloning a pre-configured template.

```go
package main

import "fmt"

// The Prototype interface
type FinancialInstrument interface {
	Clone() FinancialInstrument
	GetDetails() string
}

// Concrete Prototype
type Bond struct {
	Issuer    string
	Maturity  string
	Coupon    float64
	FaceValue float64
}

func (b *Bond) Clone() FinancialInstrument {
	// Create a copy of the Bond object
	return &Bond{
		Issuer:    b.Issuer,
		Maturity:  b.Maturity,
		Coupon:    b.Coupon,
		FaceValue: b.FaceValue,
	}
}

func (b *Bond) GetDetails() string {
	return fmt.Sprintf("Issuer: %s, Coupon: %.2f%%", b.Issuer, b.Coupon)
}

// Client
func main() {
	// Create a prototype for a US Treasury Bond
	treasuryBondPrototype := &Bond{
		Issuer:    "US Treasury",
		Maturity:  "10-Year",
		Coupon:    3.5,
		FaceValue: 1000,
	}

	// Clone the prototype to create a new, specific bond instance
	newBond := treasuryBondPrototype.Clone()
	fmt.Println("Cloned Bond Details:", newBond.GetDetails())
}
```

#### 5. Singleton

**Summary:** Ensures a class only has one instance and provides a global point of access to it.

**Financial Example:** A MarketDataService that maintains a single connection to a stock exchange feed to avoid multiple, costly connections.

```go
package main

import (
	"fmt"
	"sync"
)

type MarketDataService struct {
	connectionID int
}

var instance *MarketDataService
var once sync.Once

// GetInstance provides the global access point to the singleton instance.
func GetInstance() *MarketDataService {
	once.Do(func() {
		// This part runs only once
		fmt.Println("Creating MarketDataService instance...")
		instance = &MarketDataService{connectionID: 12345}
	})
	return instance
}

func (s *MarketDataService) GetStockPrice(ticker string) float64 {
	fmt.Printf("Fetching %s price using connection %d\n", ticker, s.connectionID)
	return 150.75 // Dummy price
}

// Client
func main() {
	service1 := GetInstance()
	service1.GetStockPrice("AAPL")

	service2 := GetInstance() // Will not create a new instance
	service2.GetStockPrice("GOOG")

	fmt.Printf("Service 1 and 2 are the same instance: %t\n", service1 == service2)
}
```

### **Structural Patterns**

These patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

#### 1. Adapter

**Summary:** Allows objects with incompatible interfaces to collaborate. It acts as a wrapper that translates one interface into another that a client expects.

**Financial Example:** Adapting an old, third-party LegacyBankAPI to conform to your application's modern PaymentGateway interface.

```go
package main

import "fmt"

// The target interface that the client code uses
type PaymentGateway interface {
	SubmitPayment(amount float64)
}

// The Adaptee (the legacy or incompatible system)
type LegacyBankAPI struct{}
func (l *LegacyBankAPI) SendTransaction(value float32) {
	fmt.Printf("Legacy API: Sent transaction of %.2f\n", value)
}

// The Adapter
type BankAPIAdapter struct {
	LegacyAPI *LegacyBankAPI
}

func (a *BankAPIAdapter) SubmitPayment(amount float64) {
	fmt.Println("Adapter: Converting and forwarding payment.")
	a.LegacyAPI.SendTransaction(float32(amount))
}

// Client
func main() {
	legacyAPI := &LegacyBankAPI{}
	adapter := &BankAPIAdapter{LegacyAPI: legacyAPI}

	// The client code interacts only with the target interface
	adapter.SubmitPayment(125.50)
}
```

#### 2. Bridge

**Summary:** Decouples an abstraction from its implementation so that the two can vary independently. It's like building a bridge between the abstraction and its implementation.

**Financial Example:** Separating the Account type (e.g., Checking, Savings) from the Notification method (e.g., SMS, Email), allowing you to mix and match them.

```go
package main

import "fmt"

// The Implementor interface
type NotificationSender interface {
	Send(message string)
}

// Concrete Implementors
type SMSSender struct{}
func (s *SMSSender) Send(message string) { fmt.Printf("Sending SMS: %s\n", message) }

type EmailSender struct{}
func (e *EmailSender) Send(message string) { fmt.Printf("Sending Email: %s\n", message) }

// The Abstraction
type Account interface {
	Withdraw(amount float64)
}

// Refined Abstraction
type CheckingAccount struct {
	notifier NotificationSender
}
func (a *CheckingAccount) Withdraw(amount float64) {
	msg := fmt.Sprintf("Withdrew %.2f from Checking.", amount)
	a.notifier.Send(msg)
}

// Client
func main() {
	smsNotifier := &SMSSender{}
	emailNotifier := &EmailSender{}

	checkingWithSMS := &CheckingAccount{notifier: smsNotifier}
	checkingWithSMS.Withdraw(100.0)

	checkingWithEmail := &CheckingAccount{notifier: emailNotifier}
	checkingWithEmail.Withdraw(250.0)
}
```

#### 3. Composite

**Summary:** Composes objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.

**Financial Example:** A financial Portfolio can contain individual Stock assets as well as other Portfolios, and you can calculate the total value of the entire structure with a single method call.

```go
package main

import "fmt"

// The Component interface
type Asset interface {
	GetValue() float64
}

// The Leaf
type Stock struct {
	Name  string
	Value float64
}
func (s *Stock) GetValue() float64 { return s.Value }

// The Composite
type Portfolio struct {
	Name   string
	Assets []Asset
}
func (p *Portfolio) GetValue() float64 {
	totalValue := 0.0
	for _, asset := range p.Assets {
		totalValue += asset.GetValue()
	}
	return totalValue
}
func (p *Portfolio) Add(asset Asset) {
	p.Assets = append(p.Assets, asset)
}

// Client
func main() {
	stock1 := &Stock{Name: "AAPL", Value: 15000}
	stock2 := &Stock{Name: "GOOG", Value: 28000}

	techPortfolio := &Portfolio{Name: "Tech Portfolio"}
	techPortfolio.Add(stock1)
	techPortfolio.Add(stock2)

	mainPortfolio := &Portfolio{Name: "Main Portfolio"}
	mainPortfolio.Add(techPortfolio)
	mainPortfolio.Add(&Stock{Name: "TSLA", Value: 12000})

	fmt.Printf("Total value of main portfolio: $%.2f\n", mainPortfolio.GetValue())
}
```

#### 4. Decorator

**Summary:** Attaches new behaviors to objects by placing them inside special wrapper objects that contain the behaviors. This allows for adding responsibilities to an object dynamically without affecting other objects.

**Financial Example:** Starting with a basic Loan and dynamically adding features like Insurance or LateFee calculations.

```go
package main

import "fmt"

// The Component interface
type Loan interface {
	GetMonthlyPayment() float64
}

// The Concrete Component
type BasicLoan struct {
	Principal float64
}
func (l *BasicLoan) GetMonthlyPayment() float64 { return l.Principal / 12.0 }

// The base Decorator
type LoanDecorator struct {
	WrappedLoan Loan
}
func (d *LoanDecorator) GetMonthlyPayment() float64 { return d.WrappedLoan.GetMonthlyPayment() }

// Concrete Decorators
type InsuranceDecorator struct {
	LoanDecorator
	InsuranceFee float64
}
func (d *InsuranceDecorator) GetMonthlyPayment() float64 {
	return d.WrappedLoan.GetMonthlyPayment() + d.InsuranceFee
}

// Client
func main() {
	loan := &BasicLoan{Principal: 1200.0}
	fmt.Printf("Basic loan payment: %.2f\n", loan.GetMonthlyPayment())

	insuredLoan := &InsuranceDecorator{
		LoanDecorator: LoanDecorator{WrappedLoan: loan},
		InsuranceFee:  10.0,
	}
	fmt.Printf("Insured loan payment: %.2f\n", insuredLoan.GetMonthlyPayment())
}
```

#### 5. Facade

**Summary:** Provides a simplified, higher-level interface to a library, a framework, or any other complex set of classes.

**Financial Example:** A MortgageApplication facade that simplifies the complex process of interacting with subsystems for credit checks, income verification, and property appraisal.

```go
package main

import "fmt"

// Complex Subsystems
type CreditCheckSystem struct{}
func (s *CreditCheckSystem) Check(applicantID string) bool {
	fmt.Println("Checking credit score...")
	return true
}

type IncomeVerificationSystem struct{}
func (s *IncomeVerificationSystem) Verify(applicantID string) bool {
	fmt.Println("Verifying income...")
	return true
}

type PropertyAppraisalSystem struct{}
func (s *PropertyAppraisalSystem) Appraise(propertyID string) bool {
	fmt.Println("Appraising property...")
	return true
}

// The Facade
type MortgageApplicationFacade struct {
	creditCheck *CreditCheckSystem
	incomeCheck *IncomeVerificationSystem
	appraisal   *PropertyAppraisalSystem
}

func (f *MortgageApplicationFacade) ApplyForMortgage(applicantID, propertyID string) bool {
	fmt.Println("Facade: Starting mortgage application process...")
	if !f.creditCheck.Check(applicantID) { return false }
	if !f.incomeCheck.Verify(applicantID) { return false }
	if !f.appraisal.Appraise(propertyID) { return false }
	fmt.Println("Facade: Mortgage application approved!")
	return true
}

// Client
func main() {
	facade := &MortgageApplicationFacade{
		creditCheck: &CreditCheckSystem{},
		incomeCheck: &IncomeVerificationSystem{},
		appraisal:   &PropertyAppraisalSystem{},
	}
	facade.ApplyForMortgage("JohnDoe123", "Property456")
}
```

#### 6. Flyweight

**Summary:** Lets you fit more objects into the available amount of RAM by sharing common parts of state between multiple objects instead of keeping all of the data in each object.

**Financial Example:** Sharing immutable Currency objects. The currency code (e.g., "USD") is intrinsic (shared), while the amount is extrinsic (unique to each transaction).

```go
package main

import "fmt"

// The Flyweight object
type Currency struct {
	Code string // Intrinsic state (shared)
}

// The Flyweight Factory
type CurrencyFactory struct {
	currencies map[string]*Currency
}

func NewCurrencyFactory() *CurrencyFactory {
	return &CurrencyFactory{currencies: make(map[string]*Currency)}
}

func (f *CurrencyFactory) GetCurrency(code string) *Currency {
	if currency, exists := f.currencies[code]; exists {
		return currency
	}
	fmt.Printf("Creating new currency object for %s\n", code)
	newCurrency := &Currency{Code: code}
	f.currencies[code] = newCurrency
	return newCurrency
}

// The Context object that uses the flyweight
type MoneyTransaction struct {
	Amount   float64   // Extrinsic state
	Currency *Currency // Flyweight
}

// Client
func main() {
	factory := NewCurrencyFactory()

	tx1 := MoneyTransaction{Amount: 100.0, Currency: factory.GetCurrency("USD")}
	tx2 := MoneyTransaction{Amount: 200.0, Currency: factory.GetCurrency("USD")}
	tx3 := MoneyTransaction{Amount: 50.0, Currency: factory.GetCurrency("EUR")}

	fmt.Printf("Transaction 1: %.2f %s\n", tx1.Amount, tx1.Currency.Code)
	fmt.Printf("Transaction 2: %.2f %s\n", tx2.Amount, tx2.Currency.Code)
	fmt.Printf("Transaction 3: %.2f %s\n", tx3.Amount, tx3.Currency.Code)
	fmt.Printf("Are USD objects the same? %t\n", tx1.Currency == tx2.Currency)
}
```

#### 7. Proxy

**Summary:** Provides a substitute or placeholder for another object. A proxy controls access to the original object, allowing you to perform something either before or after the request gets to the original object.

**Financial Example:** A proxy for a bank account that logs every transaction before it's executed on the real account.

```go
package main

import "fmt"

// The Subject interface
type BankAccount interface {
	Deposit(amount float64)
}

// The Real Subject
type RealBankAccount struct {
	Balance float64
}
func (a *RealBankAccount) Deposit(amount float64) {
	a.Balance += amount
	fmt.Printf("Deposited %.2f. New balance: %.2f\n", amount, a.Balance)
}

// The Proxy
type BankAccountProxy struct {
	realAccount *RealBankAccount
}

func (p *BankAccountProxy) Deposit(amount float64) {
	fmt.Printf("[LOG] Attempting to deposit %.2f\n", amount)
	if p.realAccount == nil {
		p.realAccount = &RealBankAccount{} // Lazy initialization
	}
	p.realAccount.Deposit(amount)
}

// Client
func main() {
	account := &BankAccountProxy{}
	account.Deposit(100.0)
	account.Deposit(50.0)
}
```

### **Behavioral Patterns**

These patterns are concerned with algorithms and the assignment of responsibilities between objects.

#### 1. Chain of Responsibility

**Summary:** Lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

**Financial Example:** An expense approval system where a request is passed up a chain of command (Manager -> Director -> VP) based on the amount.

```go
package main

import "fmt"

// The Handler interface
type Approver interface {
	SetNext(Approver)
	Approve(amount float64)
}

// Base Handler
type BaseApprover struct {
	next Approver
}
func (a *BaseApprover) SetNext(next Approver) { a.next = next }
func (a *BaseApprover) passToNext(amount float64) {
	if a.next != nil {
		a.next.Approve(amount)
	} else {
		fmt.Printf("Expense of %.2f could not be approved.\n", amount)
	}
}

// Concrete Handlers
type Manager struct { BaseApprover }
func (m *Manager) Approve(amount float64) {
	if amount <= 1000 {
		fmt.Printf("Manager approved expense of %.2f\n", amount)
	} else {
		m.passToNext(amount)
	}
}

type Director struct { BaseApprover }
func (d *Director) Approve(amount float64) {
	if amount <= 5000 {
		fmt.Printf("Director approved expense of %.2f\n", amount)
	} else {
		d.passToNext(amount)
	}
}

// Client
func main() {
	manager := &Manager{}
	director := &Director{}
	manager.SetNext(director)

	manager.Approve(500)
	manager.Approve(4500)
	manager.Approve(10000)
}
```

#### 2. Command

**Summary:** Turns a request into a stand-alone object that contains all information about the request. This lets you parameterize methods with different requests, delay or queue a request's execution, and support undoable operations.

**Financial Example:** Encapsulating Buy and Sell stock orders as command objects that can be executed by a broker.

```go
package main

import "fmt"

// The Command interface
type Order interface {
	Execute()
}

// The Receiver
type StockTrade struct{}
func (s *StockTrade) Buy(ticker string, quantity int)  { fmt.Printf("Buying %d of %s\n", quantity, ticker) }
func (s *StockTrade) Sell(ticker string, quantity int) { fmt.Printf("Selling %d of %s\n", quantity, ticker) }

// Concrete Commands
type BuyStockCommand struct {
	stock    *StockTrade
	ticker   string
	quantity int
}
func (c *BuyStockCommand) Execute() { c.stock.Buy(c.ticker, c.quantity) }

type SellStockCommand struct {
	stock    *StockTrade
	ticker   string
	quantity int
}
func (c *SellStockCommand) Execute() { c.stock.Sell(c.ticker, c.quantity) }

// The Invoker
type Broker struct {
	orders []Order
}
func (b *Broker) TakeOrder(order Order) { b.orders = append(b.orders, order) }
func (b *Broker) PlaceOrders() {
	for _, order := range b.orders {
		order.Execute()
	}
	b.orders = []Order{}
}

// Client
func main() {
	stock := &StockTrade{}
	buyCmd := &BuyStockCommand{stock: stock, ticker: "AAPL", quantity: 10}
	sellCmd := &SellStockCommand{stock: stock, ticker: "GOOG", quantity: 5}

	broker := &Broker{}
	broker.TakeOrder(buyCmd)
	broker.TakeOrder(sellCmd)
	broker.PlaceOrders()
}
```

#### 3. Iterator

**Summary:** Lets you traverse elements of a collection without exposing its underlying representation (like list, stack, tree, etc.).

**Financial Example:** An iterator to go through a customer's TransactionHistory without exposing the internal slice.

```go
package main

import "fmt"

type Transaction struct {
	Amount float64
	Type   string
}

// The Aggregate interface
type TransactionCollection interface {
	CreateIterator() Iterator
}

// The Concrete Aggregate
type TransactionHistory struct {
	transactions []Transaction
}
func (h *TransactionHistory) CreateIterator() Iterator {
	return &TransactionIterator{history: h, index: 0}
}

// The Iterator interface
type Iterator interface {
	HasNext() bool
	Next() *Transaction
}

// The Concrete Iterator
type TransactionIterator struct {
	history *TransactionHistory
	index   int
}
func (it *TransactionIterator) HasNext() bool { return it.index < len(it.history.transactions) }
func (it *TransactionIterator) Next() *Transaction {
	if it.HasNext() {
		tx := &it.history.transactions[it.index]
		it.index++
		return tx
	}
	return nil
}

// Client
func main() {
	history := &TransactionHistory{
		transactions: []Transaction{{100, "DEBIT"}, {500, "CREDIT"}},
	}
	iterator := history.CreateIterator()
	for iterator.HasNext() {
		tx := iterator.Next()
		fmt.Printf("Transaction: %.2f (%s)\n", tx.Amount, tx.Type)
	}
}
```

#### 4. Mediator

**Summary:** Reduces chaotic dependencies between objects. The pattern restricts direct communications between the objects and forces them to collaborate only via a mediator object.

**Financial Example:** A TradingSystem acts as a mediator between different Traders, so they don't need to know about each other directly.

```go
package main

import "fmt"

// The Mediator interface
type TradingMediator interface {
	PlaceOrder(order string, trader *Trader)
}

// Colleague
type Trader struct {
	Name     string
	mediator TradingMediator
}
func (t *Trader) PlaceOrder(order string) {
	fmt.Printf("%s is placing an order: %s\n", t.Name, order)
	t.mediator.PlaceOrder(order, t)
}

// Concrete Mediator
type StockExchange struct {
	traders []*Trader
}
func (e *StockExchange) Register(t *Trader) { e.traders = append(e.traders, t) }
func (e *StockExchange) PlaceOrder(order string, originator *Trader) {
	fmt.Printf("Exchange received order '%s' from %s. Notifying other traders.\n", order, originator.Name)
	for _, t := range e.traders {
		if t != originator {
			fmt.Printf("  - Notifying %s\n", t.Name)
		}
	}
}

// Client
func main() {
	exchange := &StockExchange{}
	trader1 := &Trader{Name: "Trader A", mediator: exchange}
	trader2 := &Trader{Name: "Trader B", mediator: exchange}
	exchange.Register(trader1)
	exchange.Register(trader2)

	trader1.PlaceOrder("BUY 100 AAPL")
}
```

#### 5. Memento

**Summary:** Lets you save and restore the previous state of an object without revealing the details of its implementation.

**Financial Example:** Saving the state of a Portfolio before a rebalancing operation, allowing the user to undo the changes if they are not satisfied.

```go
package main

import "fmt"

// The Memento - stores the state
type PortfolioMemento struct {
	state map[string]int
}

// The Originator - the object whose state we want to save
type Portfolio struct {
	holdings map[string]int
}
func (p *Portfolio) AddStock(ticker string, quantity int) { p.holdings[ticker] += quantity }
func (p *Portfolio) CreateMemento() *PortfolioMemento {
	// Create a copy to avoid modification
	stateCopy := make(map[string]int)
	for k, v := range p.holdings {
		stateCopy[k] = v
	}
	return &PortfolioMemento{state: stateCopy}
}
func (p *Portfolio) Restore(m *PortfolioMemento) { p.holdings = m.state }

// The Caretaker - knows when to save and restore
type History struct {
	mementos []*PortfolioMemento
}
func (h *History) Push(m *PortfolioMemento) { h.mementos = append(h.mementos, m) }
func (h *History) Pop() *PortfolioMemento {
	last := h.mementos[len(h.mementos)-1]
	h.mementos = h.mementos[:len(h.mementos)-1]
	return last
}

// Client
func main() {
	portfolio := &Portfolio{holdings: make(map[string]int)}
	history := &History{}

	portfolio.AddStock("AAPL", 100)
	history.Push(portfolio.CreateMemento())
	fmt.Printf("Current Portfolio: %+v\n", portfolio.holdings)

	portfolio.AddStock("GOOG", 50)
	fmt.Printf("After rebalance: %+v\n", portfolio.holdings)

	// Undo the change
	portfolio.Restore(history.Pop())
	fmt.Printf("Restored Portfolio: %+v\n", portfolio.holdings)
}
```

#### 6. Observer

**Summary:** Defines a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.

**Financial Example:** A StockTicker (the subject) notifies multiple PortfolioDisplays (the observers) whenever a stock price changes.

```go
package main

import "fmt"

// The Observer interface
type Investor interface {
	Update(stock string, price float64)
}

// The Subject
type StockTicker struct {
	observers []Investor
	price     map[string]float64
}
func (s *StockTicker) Register(o Investor)   { s.observers = append(s.observers, o) }
func (s *StockTicker) Notify(stock string) {
	for _, o := range s.observers {
		o.Update(stock, s.price[stock])
	}
}
func (s *StockTicker) SetPrice(stock string, price float64) {
	if s.price == nil { s.price = make(map[string]float64) }
	s.price[stock] = price
	fmt.Printf("Ticker: %s price changed to %.2f\n", stock, price)
	s.Notify(stock)
}

// Concrete Observer
type PortfolioDisplay struct {
	name string
}
func (p *PortfolioDisplay) Update(stock string, price float64) {
	fmt.Printf("[%s] Notified: %s is now $%.2f\n", p.name, stock, price)
}

// Client
func main() {
	ticker := &StockTicker{}
	display1 := &PortfolioDisplay{name: "My Portfolio"}
	display2 := &PortfolioDisplay{name: "Alert System"}

	ticker.Register(display1)
	ticker.Register(display2)

	ticker.SetPrice("MSFT", 300.50)
}
```

#### 7. State

**Summary:** Lets an object alter its behavior when its internal state changes. It appears as if the object changed its class.

**Financial Example:** A Loan object can be in different states (Pending, Active, PaidOff), and its behavior (e.g., makePayment) changes depending on the current state.
```go
package main

import "fmt"

type Loan struct {
	currentState State
}
func NewLoan() *Loan {
	l := &Loan{}
	l.currentState = &PendingState{loan: l} // Initial state
	return l
}
func (l *Loan) SetState(s State) { l.currentState = s }
func (l *Loan) Approve()         { l.currentState.Approve() }
func (l *Loan) MakePayment()     { l.currentState.MakePayment() }

// The State interface
type State interface {
	Approve()
	MakePayment()
}

// Concrete States
type PendingState struct{ loan *Loan }
func (s *PendingState) Approve() {
	fmt.Println("Loan approved. Moving to Active state.")
	s.loan.SetState(&ActiveState{loan: s.loan})
}
func (s *PendingState) MakePayment() { fmt.Println("Cannot make payment on a pending loan.") }

type ActiveState struct{ loan *Loan }
func (s *ActiveState) Approve()     { fmt.Println("Loan is already active.") }
func (s *ActiveState) MakePayment() { fmt.Println("Payment received for active loan.") }

// Client
func main() {
	loan := NewLoan()
	loan.MakePayment() // Cannot make payment
	loan.Approve()      // Moves to Active
	loan.MakePayment()  // Can make payment
}
```

#### 8. Strategy

**Summary:** Lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

**Financial Example:** A RiskCalculator can use different strategies (SimpleVolatility, ValueAtRisk) to assess the risk of a portfolio.

```go
package main

import "fmt"

// The Strategy interface
type RiskCalculationStrategy interface {
	Calculate(portfolioValue float64) float64
}

// Concrete Strategies
type SimpleVolatilityStrategy struct{}
func (s *SimpleVolatilityStrategy) Calculate(v float64) float64 { return v * 0.15 } // 15% volatility

type ValueAtRiskStrategy struct{}
func (s *ValueAtRiskStrategy) Calculate(v float64) float64 { return v * 0.05 } // 5% VaR

// The Context
type RiskCalculator struct {
	strategy RiskCalculationStrategy
}
func (c *RiskCalculator) SetStrategy(s RiskCalculationStrategy) { c.strategy = s }
func (c *RiskCalculator) CalculateRisk(portfolioValue float64) {
	risk := c.strategy.Calculate(portfolioValue)
	fmt.Printf("Calculated risk: %.2f\n", risk)
}

// Client
func main() {
	calculator := &RiskCalculator{}
	portfolioValue := 100000.0

	calculator.SetStrategy(&SimpleVolatilityStrategy{})
	calculator.CalculateRisk(portfolioValue)

	calculator.SetStrategy(&ValueAtRiskStrategy{})
	calculator.CalculateRisk(portfolioValue)
}
```

#### 9. Template Method

**Summary:** Defines the skeleton of an algorithm in a superclass but lets subclasses override specific steps of the algorithm without changing its structure.

**Financial Example:** A template for generating financial reports (AuditReport, PerformanceReport) where the overall structure is the same, but data fetching and formatting steps are specific to each report type.

```go
package main

import "fmt"

// The Abstract Class (using an interface and a base struct)
type ReportGenerator interface {
	Generate()
	fetchData() string
	formatData(data string) string
	printReport(formattedData string)
}

type baseReportGenerator struct {
	impl ReportGenerator
}
func (b *baseReportGenerator) Generate() {
	data := b.impl.fetchData()
	formatted := b.impl.formatData(data)
	b.impl.printReport(formatted)
}

// Concrete Classes
type AuditReport struct {
	baseReportGenerator
}
func NewAuditReport() *AuditReport {
	r := &AuditReport{}
	r.baseReportGenerator.impl = r
	return r
}
func (r *AuditReport) fetchData() string { return "Audit Log Data" }
func (r *AuditReport) formatData(d string) string { return fmt.Sprintf("--- AUDIT REPORT ---\n%s", d) }
func (r *AuditReport) printReport(fd string) { fmt.Println(fd) }

type PerformanceReport struct {
	baseReportGenerator
}
func NewPerformanceReport() *PerformanceReport {
	r := &PerformanceReport{}
	r.baseReportGenerator.impl = r
	return r
}
func (r *PerformanceReport) fetchData() string { return "Portfolio Performance Data" }
func (r *PerformanceReport) formatData(d string) string { return fmt.Sprintf("--- PERFORMANCE REPORT ---\n%s", d) }
func (r *PerformanceReport) printReport(fd string) { fmt.Println(fd) }

// Client
func main() {
	audit := NewAuditReport()
	audit.Generate()

	fmt.Println()

	perf := NewPerformanceReport()
	perf.Generate()
}
```

#### 10. Visitor

**Summary:** Lets you separate algorithms from the objects on which they operate. It allows adding new operations to existing object structures without modifying them.

**Financial Example:** A TaxCalculator visitor that can "visit" different types of assets (Stock, Bond, RealEstate) to calculate the specific tax liability for each, without the asset classes needing to know about tax logic.
```go
package main

import "fmt"

// The Visitor interface
type AssetVisitor interface {
	VisitStock(s *Stock)
	VisitBond(b *Bond)
}

// The Element interface
type Asset interface {
	Accept(v AssetVisitor)
}

// Concrete Elements
type Stock struct { Value float64 }
func (s *Stock) Accept(v AssetVisitor) { v.VisitStock(s) }

type Bond struct { Value float64 }
func (b *Bond) Accept(v AssetVisitor) { v.VisitBond(b) }

// Concrete Visitor
type TaxCalculator struct {
	TotalTax float64
}
func (t *TaxCalculator) VisitStock(s *Stock) {
	tax := s.Value * 0.15 // 15% capital gains tax
	fmt.Printf("Stock tax: %.2f\n", tax)
	t.TotalTax += tax
}
func (t *TaxCalculator) VisitBond(b *Bond) {
	tax := b.Value * 0.10 // 10% income tax
	fmt.Printf("Bond tax: %.2f\n", tax)
	t.TotalTax += tax
}

// Client
func main() {
	assets := []Asset{&Stock{Value: 1000}, &Bond{Value: 2000}}
	taxVisitor := &TaxCalculator{}

	for _, asset := range assets {
		asset.Accept(taxVisitor)
	}
	fmt.Printf("Total tax liability: %.2f\n", taxVisitor.TotalTax)
}
```

### **Anti-Patterns**

These are common responses to recurring problems that are usually ineffective and may result in bad consequences.

#### 1. Blob / God Object

**Summary:** An anti-pattern where a single class takes on an excessive number of responsibilities, becoming large, unwieldy, and difficult to maintain. It violates the Single Responsibility Principle.

**Financial Example:** A single Customer object that handles profile information, transaction history, loan applications, and investment management.

```go
package main

import "fmt"

// ANTI-PATTERN: This object does too much.
type GodCustomerObject struct {
	Name    string
	Address string
	Balance float64
}

// Profile-related method
func (c *GodCustomerObject) UpdateAddress(newAddress string) {
	fmt.Println("Updating address...")
	c.Address = newAddress
}

// Transaction-related method
func (c *GodCustomerObject) ProcessTransaction(amount float64) {
	fmt.Println("Processing transaction...")
	c.Balance += amount
}

// Loan-related method
func (c *GodCustomerObject) ApplyForLoan(amount float64) bool {
	fmt.Println("Applying for loan...")
	// Complex loan logic would go here
	return c.Balance > amount*0.1
}

// Client
func main() {
	customer := &GodCustomerObject{Name: "John Doe", Balance: 5000}
	customer.UpdateAddress("123 New St")
	customer.ProcessTransaction(-100)
	customer.ApplyForLoan(10000)
	fmt.Printf("Customer state: %+v\n", customer)
}
```

#### 2. Spaghetti Code

**Summary:** An anti-pattern characterized by code that is tangled, unstructured, and difficult to follow. It often features long methods, excessive use of global variables, and a lack of clear separation of concerns, making it nearly impossible to debug or modify.

**Financial Example:** A single function to process a wire transfer that mixes input validation, database access, and external API calls without any structure.

```go
package main

import "fmt"

// ANTI-PATTERN: A single function with tangled logic.
func ProcessWireTransfer(amount float64, accountFrom, accountTo string) {
	fmt.Println("Starting wire transfer...")

	// 1. Validation logic mixed in
	if amount <= 0 {
		fmt.Println("Error: Amount must be positive.")
		return
	}
	if len(accountFrom) < 5 {
		fmt.Println("Error: Invalid source account.")
		return
	}

	// 2. Database logic mixed in
	fmt.Printf("DB: Checking balance for %s...\n", accountFrom)
	// Pretend we found a balance of 500
	balance := 500.0
	if balance < amount {
		fmt.Println("Error: Insufficient funds.")
		return
	}
	fmt.Printf("DB: Debiting %.2f from %s...\n", amount, accountFrom)
	fmt.Printf("DB: Crediting %.2f to %s...\n", amount, accountTo)

	// 3. External API call mixed in
	fmt.Println("API: Notifying external compliance service...")
	// Pretend API call is successful

	fmt.Println("Wire transfer completed successfully.")
}

// Client
func main() {
	ProcessWireTransfer(100.0, "ACCT12345", "ACCT67890")
}
```


#### 3. Swiss Army Knife

**Summary:** An anti-pattern where a class is overloaded with a wide range of unrelated functionalities, much like a physical Swiss Army knife. While versatile, it becomes bloated and violates the Single Responsibility and Interface Segregation principles.

**Financial Example:** A FinancialUtils class that contains methods for currency conversion, loan calculation, stock valuation, and PDF report generation.

```go
package main

import "fmt"

// ANTI-PATTERN: This class provides too many unrelated services.
type FinancialSwissArmyKnife struct{}

func (u *FinancialSwissArmyKnife) ConvertUSDtoEUR(amount float64) float64 {
	fmt.Println("Converting USD to EUR...")
	return amount * 0.92
}

func (u *FinancialSwissArmyKnife) CalculateMonthlyLoanPayment(principal, rate float64, years int) float64 {
	fmt.Println("Calculating loan payment...")
	// Simplified calculation
	return (principal * (1 + rate)) / float64(years*12)
}

func (u *FinancialSwissArmyKnife) GeneratePDFReport(data string) {
	fmt.Printf("Generating PDF with data: %s\n", data)
}

// Client
func main() {
	utils := &FinancialSwissArmyKnife{}
	eur := utils.ConvertUSDtoEUR(100)
	payment := utils.CalculateMonthlyLoanPayment(100000, 0.05, 30)
	utils.GeneratePDFReport("Annual Summary")

	fmt.Printf("EUR: %.2f, Payment: %.2f\n", eur, payment)
}
```

### **Architectural Patterns**

These patterns describe fundamental structural organization for software systems.

#### 1. Layered (N-Tier)

**Summary:** Organizes a system into horizontal layers, where each layer has a specific responsibility (e.g., Presentation, Business Logic, Data Access). A layer can only communicate with the layer directly below it.

**Financial Example:** A simple banking application with a Presentation Layer (UI), Business Layer (logic), and Data Access Layer (database).

```go
package main

import "fmt"

// --- Data Access Layer ---
type AccountRepository struct{}
func (r *AccountRepository) GetBalance(accountID string) float64 {
	fmt.Printf("[Data] Fetching balance for %s\n", accountID)
	return 1000.0 // Dummy data
}

// --- Business Logic Layer ---
type AccountService struct {
	repo *AccountRepository
}
func (s *AccountService) Withdraw(accountID string, amount float64) bool {
	fmt.Printf("[Business] Processing withdrawal of %.2f\n", amount)
	balance := s.repo.GetBalance(accountID)
	if balance >= amount {
		// In a real app, would update the balance
		return true
	}
	return false
}

// --- Presentation Layer ---
type AccountController struct {
	service *AccountService
}
func (c *AccountController) HandleWithdrawRequest(accountID string, amount float64) {
	fmt.Printf("[Presentation] Received withdrawal request for %.2f\n", amount)
	if c.service.Withdraw(accountID, amount) {
		fmt.Println("[Presentation] Success!")
	} else {
		fmt.Println("[Presentation] Failure: Insufficient funds.")
	}
}

// Client
func main() {
	repo := &AccountRepository{}
	service := &AccountService{repo: repo}
	controller := &AccountController{service: service}

	controller.HandleWithdrawRequest("ACCT123", 500.0)
}
```

#### 2. MVC (Model-View-Controller)

**Summary:** Separates an application into three interconnected components: the Model (data and business logic), the View (UI representation), and the Controller (handles user input and updates the Model).

**Financial Example:** A simple stock portfolio viewer.
```go
package main

import "fmt"

// Model: Holds the application data and business logic.
type Stock struct {
	Ticker string
	Price  float64
}

// View: Displays the data from the model.
type StockView struct{}
func (v *StockView) PrintStockDetails(ticker string, price float64) {
	fmt.Printf("--- Stock Display ---\nTicker: %s\nPrice: $%.2f\n", ticker, price)
}

// Controller: Handles user input and updates the model.
type StockController struct {
	model *Stock
	view  *StockView
}
func (c *StockController) SetStockPrice(price float64) {
	c.model.Price = price
	c.UpdateView()
}
func (c *StockController) UpdateView() {
	c.view.PrintStockDetails(c.model.Ticker, c.model.Price)
}

// Client
func main() {
	model := &Stock{Ticker: "TSLA", Price: 180.00}
	view := &StockView{}
	controller := &StockController{model: model, view: view}

	controller.UpdateView()
	fmt.Println("\nMarket update...")
	controller.SetStockPrice(185.50)
}
```

#### 3. Pipe and Filter

**Summary:** Processes a stream of data in stages, where each stage (Filter) performs a specific transformation and passes its output to the next stage (via a Pipe).

**Financial Example:** A trade processing pipeline that validates, enriches, and then records a trade.

```go
package main

import "fmt"

type TradeData map[string]interface{}

// Filter interface
type Filter interface {
	Process(data TradeData) TradeData
}

// Concrete Filters
type ValidationFilter struct{}
func (f *ValidationFilter) Process(data TradeData) TradeData {
	if _, ok := data["amount"]; !ok {
		fmt.Println("Validation failed: amount missing.")
		return nil // Stop processing
	}
	fmt.Println("Trade validated.")
	return data
}

type EnrichmentFilter struct{}
func (f *EnrichmentFilter) Process(data TradeData) TradeData {
	if data == nil { return nil }
	data["currency"] = "USD"
	fmt.Println("Trade enriched with currency.")
	return data
}

type RecordingFilter struct{}
func (f *RecordingFilter) Process(data TradeData) TradeData {
	if data == nil { return nil }
	fmt.Printf("Trade recorded: %+v\n", data)
	return data
}

// The Pipeline
type TradePipeline struct {
	filters []Filter
}
func (p *TradePipeline) AddFilter(f Filter) { p.filters = append(p.filters, f) }
func (p *TradePipeline) Execute(data TradeData) {
	for _, filter := range p.filters {
		data = filter.Process(data)
		if data == nil {
			break
		}
	}
}

// Client
func main() {
	pipeline := &TradePipeline{}
	pipeline.AddFilter(&ValidationFilter{})
	pipeline.AddFilter(&EnrichmentFilter{})
	pipeline.AddFilter(&RecordingFilter{})

	trade := TradeData{"ticker": "IBM", "amount": 10000.0}
	pipeline.Execute(trade)
}
```

### **Concurrency Patterns**

These patterns deal with multi-threaded programming paradigms.

#### 1. Barrier

**Summary:** A synchronization mechanism that forces a set of concurrent processes or goroutines to wait at a certain point (the barrier) until all of them have arrived before any are allowed to proceed.

**Financial Example:** Simulating a trading day where multiple market data feeds must all be "online" before the trading engine can start processing orders.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func marketFeed(name string, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("%s: Connecting...\n", name)
	time.Sleep(1 * time.Second) // Simulate connection time
	fmt.Printf("%s: Online and waiting at the barrier.\n", name)
}

// Client
func main() {
	var wg sync.WaitGroup
	feeds := []string{"NYSE Feed", "NASDAQ Feed", "FX Feed"}
	wg.Add(len(feeds))

	fmt.Println("Waiting for all market feeds to come online...")

	for _, feedName := range feeds {
		go marketFeed(feedName, &wg)
	}

	// The WaitGroup acts as a barrier. The next line will not execute
	// until all goroutines have called wg.Done().
	wg.Wait()

	fmt.Println("\nAll feeds are online. Starting trading engine.")
	// ... start trading logic here ...
}
```

#### 2. Double-Checked Locking

**Summary:** A concurrency pattern used to reduce the overhead of acquiring a lock by first testing the locking criterion (the "check") without the lock, and only acquiring the lock if the check fails. It's primarily used for lazy initialization of singleton objects in a thread-safe way.

**Financial Example:** A thread-safe singleton for a RiskEngine that is expensive to create.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type RiskEngine struct{}
func (e *RiskEngine) Calculate() { fmt.Println("Calculating portfolio risk...") }

var riskEngineInstance *RiskEngine
var mu sync.Mutex

// GetRiskEngineInstance uses double-checked locking for lazy initialization.
func GetRiskEngineInstance() *RiskEngine {
	if riskEngineInstance == nil { // First check (without lock)
		mu.Lock()
		defer mu.Unlock()
		if riskEngineInstance == nil { // Second check (with lock)
			fmt.Println("Creating expensive RiskEngine...")
			time.Sleep(1 * time.Second) // Simulate expensive creation
			riskEngineInstance = &RiskEngine{}
		}
	}
	return riskEngineInstance
}

// Client
func main() {
	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			engine := GetRiskEngineInstance()
			engine.Calculate()
		}()
	}
	wg.Wait()
}
```

Note: In modern Go, sync.Once is the idiomatic and safer way to achieve this, as shown in the Singleton example.
