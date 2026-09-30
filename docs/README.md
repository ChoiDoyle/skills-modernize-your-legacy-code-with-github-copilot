# Student Account COBOL Components

This directory documents the COBOL account-management example in `src/cobol`. The program models a simple student account with a stored balance and interactive credit/debit operations.

## Component Overview

### `main.cob`

`main.cob` is the interactive entry point and menu controller.

Key responsibilities:

- Displays the Account Management System menu.
- Accepts a user choice from 1 through 4.
- Dispatches supported requests to the `Operations` program:
  - `TOTAL ` to view the current balance.
  - `CREDIT` to add funds.
  - `DEBIT ` to remove funds.
- Ends the loop when the user selects option 4.
- Displays an error for any choice outside the supported range.

The program continues showing the menu until its `CONTINUE-FLAG` is changed from `YES` to `NO`.

### `operations.cob`

`operations.cob` contains the account business operations.

Key responsibilities:

- Handles the operation passed by `main.cob`.
- Reads the current balance through `DataProgram` before changing or displaying it.
- Displays the current balance for a `TOTAL ` request.
- Accepts a credit amount, adds it to the balance, and persists the result for a `CREDIT` request.
- Accepts a debit amount, checks available funds, subtracts the amount, and persists the result for a `DEBIT ` request.
- Displays an insufficient-funds message when a debit is greater than the current balance.

### `data.cob`

`data.cob` provides the balance storage boundary used by `operations.cob`.

Key responsibilities:

- Maintains the account balance in `STORAGE-BALANCE`.
- Returns the stored balance when called with `READ`.
- Replaces the stored balance when called with `WRITE`.
- Returns control to the caller with `GOBACK` after each request.

This component uses a simple in-memory COBOL field; it does not persist data to a file or database.

## Student Account Business Rules

- The account starts with a balance of `$1,000.00`.
- A balance inquiry does not change the balance.
- Credits increase the balance by the entered amount.
- Debits decrease the balance by the entered amount only when sufficient funds are available.
- A debit greater than the current balance is rejected and leaves the balance unchanged.
- The balance is represented as an unsigned numeric value with two decimal places (`PIC 9(6)V99`).
- The interactive menu accepts four choices: view balance, credit, debit, or exit.
- Invalid menu choices do not modify the account and return the user to the menu.

## Program Flow

```text
main.cob
  -> Operations(TOTAL | CREDIT | DEBIT)
       -> DataProgram(READ)
       -> [apply business rule]
       -> DataProgram(WRITE, when the balance changes)
```

The operation names are fixed-width six-character values, so `TOTAL ` and `DEBIT ` include a trailing space while `CREDIT` does not.

## Application Data Flow

```mermaid
sequenceDiagram
  actor Student
  participant Main as main.cob
  participant Operations as operations.cob
  participant Data as data.cob

  loop Until the student exits
    Main->>Student: Display account menu
    Student->>Main: Enter menu choice

    alt View balance (1)
      Main->>Operations: CALL Operations(TOTAL)
      Operations->>Data: CALL DataProgram(READ, balance)
      Data-->>Operations: Return stored balance
      Operations-->>Main: Display current balance
      Main-->>Student: Show balance
    else Credit account (2)
      Main->>Operations: CALL Operations(CREDIT)
      Operations-->>Student: Request credit amount
      Student->>Operations: Enter credit amount
      Operations->>Data: CALL DataProgram(READ, balance)
      Data-->>Operations: Return stored balance
      Operations->>Operations: Add credit amount
      Operations->>Data: CALL DataProgram(WRITE, balance)
      Data-->>Operations: Store updated balance
      Operations-->>Main: Display credit confirmation
      Main-->>Student: Show new balance
    else Debit account (3)
      Main->>Operations: CALL Operations(DEBIT)
      Operations-->>Student: Request debit amount
      Student->>Operations: Enter debit amount
      Operations->>Data: CALL DataProgram(READ, balance)
      Data-->>Operations: Return stored balance
      alt Sufficient funds
        Operations->>Operations: Subtract debit amount
        Operations->>Data: CALL DataProgram(WRITE, balance)
        Data-->>Operations: Store updated balance
        Operations-->>Main: Display debit confirmation
        Main-->>Student: Show new balance
      else Insufficient funds
        Operations-->>Main: Display insufficient-funds message
        Main-->>Student: Show rejection
      end
    else Exit (4)
      Main->>Main: Set CONTINUE-FLAG to NO
      Main-->>Student: Display exit message
    else Invalid choice
      Main-->>Student: Display invalid-choice message
    end
  end
```
