This document describes the auto proposal withdrawal batch job.
## 1.1 Description
This document describes how the auto withdraw batch Jobs are processed by monthly batch. This includes:

- Updating proposal status; and 
- perform refund payment if refund is needed for withdrawn proposal.  

*NOTE*: Frequency of batch is configurable.
## 1.2 Prerequisites
Payee information has been entered for the proposal (when user selected PH or LA as payee, system still checks whether there are five specific basic information under this role). 
## 1.3 Procedures
***Step 01.*** System run batch by month to auto withdraw qualified proposals.  

***Step 02.*** System withdraw qualified proposals and update proposal status as Withdrawn.

***Step 03.*** System check if proposal need refund. If need, send proposal to finance module to refund premium and then go to step 4. If not, then process ends and go to step 5.

***Step 04.*** Generate payment requisition for refund of all amount in general suspense.  

***Step 05.*** Generate withdrawal letter.

## 1.4 Business Rules
system select qualified policy to run auto proposal withdrawal batch monthly:

1. System verify that Payee information has been entered for the proposal or not. If yes, go on check 2). If not, the proposal is not selected.  
2. Proposal satisfying the following conditions a. or b. will be auto withdrawn.  
    - **a.** Policy status = underwriting in progress, Conditions i. or ii. must be met:  
        - i. LCA status = Printed (the policy LCA letter has not been replied to); system date - LCA print date > 60 natural days (configurable)  
        - ii.	If a. is not satisfied, conditions are:   
            - query letter = (pending or printed) AND system date - query letter print date > 60 natural days (configurable)] 
            - or [system date - registration date > 183 natural days (configurable)] when no letters generated or LCA letter status is not printed or query letter = replied.
    - **b.** Policy status = waiting for data entry/data entry in progress/waiting for data verification/verification in progress/waiting for underwriting, conditions blow must be met:
        - i. query letter =printed AND system date - query letter print date > 60 natural days(configurable)
        - ii. or [system date - registration date> 183 natural days(configurable) when no query letters generated.
## 1.5 User Cases
## # Case 1: Withdrawal with refund&NB Query letter printed
- Preconditions:
    - Policy commencement date=2022-10-17;
    - Policy Status=Data Entry in Progress;
    - Payee information is entered;
    - Query Letter status=Printed&Print Date=2022-10-17;
    - General Suspense Balance=3000.  

1) On 2023-1-1, system validate current proposal need auto withdraw during the monthly batch;  
2) System update the policy status&benefit status to Withdrawn;  
3) System validate that refund is needed and generate payment requisition for all amount in general suspense;  

## # Case 2: LCA generated and printed
- Preconditions:
    - Policy commencement date=2022-10-17;
    - Payee information is entered;
    - Policy Status=Underwriting in Progress;
    - LCA status=Printed;
    - General Suspense Balance=0.  

1) On 2023-1-1, system validate current proposal need auto withdraw during the monthly batch;  
2) System update the proposal status to Withdrawn;  
3) System validate that refund is not needed. Process ends.

## # Case 3: LCA generated and not printed
- Preconditions:
    - Policy commencement date=2022-10-17;
    - Payee information is entered;
    - Policy Status=Underwriting in Progress;
    - LCA status=to be Printed;
    - General Suspense Balance=0.  

1) On 2023-5-10, system validate current proposal need auto withdraw during the monthly batch;  
2) System update the proposal status to Withdrawn;  
3) System validate that refund is not needed. Process ends.

## # Case 4: UW query letter printed
- Preconditions:
    - Policy commencement date=2022-10-17;
    - Payee information is entered;
    - Policy Status=Underwriting in Progress;
    - UW Query Letter status=Printed;
    - General Suspense Balance=0.  

1) On 2023-1-1, system validate current proposal need auto withdraw during the monthly batch;  
2) System update the proposal status to Withdrawn;  
3) System validate that refund is not needed. Process ends.

## # Case 5: Query letter not printed
- Preconditions:
    - Policy commencement date=2022-10-17;
    - Payee information is entered;
    - Policy Status=Data Entry in Progress;
    - General Suspense Balance=0.  

1) On 2023-5-10, system validate current proposal need auto withdraw during the monthly batch;  
2) System update the proposal status to Withdrawn;  
3) System validate that refund is not needed. Process ends.

## # Case 6: No query letter generated
- Preconditions:
    - Policy commencement date=2022-10-17;
    - Payee information is entered;
    - Policy Status=Verification in Progress;
    - General Suspense Balance=0.  

1) On 2023-5-10, system validate current proposal need auto withdraw during the monthly batch;  
2) System update the proposal status to Withdrawn;  
3) System validate that refund is not needed. Process ends.

