# case_study_utkarsh_sql
Loan Management System in Banking Database


use utkarsh;
CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    PhoneNumber VARCHAR(15),
    Address VARCHAR(255)
);

Loan Product Table
CREATE TABLE Loan_Product (
    LoanProductID INT PRIMARY KEY,
    ProductName VARCHAR(50),
    MaxAmount DECIMAL(15, 2),
    MaxTenure INT,  -- In months
    InterestRate DECIMAL(5, 2)  -- Annual interest rate
);


Loan Table
CREATE TABLE Loan (
    LoanID INT PRIMARY KEY,
    CustomerID INT,
    LoanProductID INT,
    LoanAmount DECIMAL(15, 2),
    InterestRate DECIMAL(5, 2),  -- Annual interest rate
    TenureMonths INT,  -- Loan tenure in months
    StartDate DATE,
    LoanStatus VARCHAR(20),  -- E.g., 'Active', 'Closed'
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID),
    FOREIGN KEY (LoanProductID) REFERENCES Loan_Product(LoanProductID)
);

Repayment Table
CREATE TABLE Repayment (
    RepaymentID INT PRIMARY KEY,
    LoanID INT,
    RepaymentDate DATE,
    PrincipalPaid DECIMAL(15, 2),
    InterestPaid DECIMAL(15, 2),
    TotalRepayment DECIMAL(15, 2),
    FOREIGN KEY (LoanID) REFERENCES Loan(LoanID)
);

transiction Table
CREATE TABLE Transaction (
    TransactionID INT PRIMARY KEY,
    LoanID INT,
    TransactionDate DATE,
    TransactionAmount DECIMAL(15, 2),
    TransactionType VARCHAR(20),  -- E.g., 'Disbursement', 'Repayment'
    FOREIGN KEY (LoanID) REFERENCES Loan(LoanID)
);


Repayment 
EMI Calculation
SELECT LoanID, 
       ROUND((LoanAmount * (InterestRate / 1200) * POWER(1 + (InterestRate / 1200), TenureMonths)) / 
             (POWER(1 + (InterestRate / 1200), TenureMonths) - 1), 2) AS EMI
FROM Loan
WHERE LoanID = 12345;  -- Replace with actual LoanID

INSERT INTO Repayment (LoanID, RepaymentDate, PrincipalPaid, InterestPaid, TotalRepayment)
VALUES (12345, '2025-01-15', 5000.00, 2000.00, 7000.00);  -- Example repayment

-- Update Loan balance
UPDATE Loan
SET LoanAmount = LoanAmount - 5000.00  -- Principal paid
WHERE LoanID = 12345;


Transiction
INSERT INTO Transaction (LoanID, TransactionDate, TransactionAmount, TransactionType)
VALUES (12345, '2025-01-10', 100000.00, 'Disbursement');  -- Loan disbursement example


Overall Loan Balance
SELECT LoanID, 
       LoanAmount - COALESCE(SUM(PrincipalPaid), 0) AS OutstandingBalance
FROM Loan
LEFT JOIN Repayment ON Loan.LoanID = Repayment.LoanID
WHERE Loan.LoanID = 12345  -- Replace with the actual LoanID
GROUP BY Loan.LoanID, LoanAmount;


Loan Statement Query
SELECT Repayment.RepaymentDate, 
       Repayment.PrincipalPaid, 
       Repayment.InterestPaid, 
       Repayment.TotalRepayment, 
       Loan.LoanAmount - COALESCE(SUM(Repayment.PrincipalPaid), 0) AS OutstandingBalance
FROM Loan
INNER JOIN Repayment ON Loan.LoanID = Repayment.LoanID
WHERE Loan.LoanID = 12345  -- Replace with actual LoanID
GROUP BY Repayment.RepaymentDate, Repayment.PrincipalPaid, Repayment.InterestPaid, Repayment.TotalRepayment, Loan.LoanAmount;


Loan Closure
UPDATE Loan
SET LoanStatus = 'Closed'
WHERE LoanID = 12345  -- Replace with actual LoanID
AND LoanAmount = (SELECT COALESCE(SUM(PrincipalPaid), 0) FROM Repayment WHERE LoanID = 12345);


Updating Interest
UPDATE Loan
SET InterestRate = 7.50  -- New interest rate
WHERE LoanProductID = 1;  -- Replace with actual LoanProductID
