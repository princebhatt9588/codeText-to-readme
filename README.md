Customers(CustomerID, Name, Email, Address, AccountBalance)
   |
   |---< Bets(BetID, BetAmount, BetTime, BetType, Outcome, CustomerID, GameID) >--- Games(GameID, GameDate, GameType, Teams, Odds)
   |
Transactions(TransactionID, CustomerID, Amount, TransactionType, Date, PaymentMethodID)
   |
PaymentMethods(PaymentMethodID, MethodName, Fees)
   
ReportSummary(ReportID, Date, TotalRevenue, TotalBets, TotalCustomers)