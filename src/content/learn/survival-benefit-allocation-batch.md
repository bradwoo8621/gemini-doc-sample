This section describes the survival benefit allocation processing.
## 1.1 Description
This document describes the batch job that allocates the survival benefit.  
The term ‘Survival Benefit Plan’ will be used throughout the use case. It does not represent a plan that is entered by user. Rather, this term is based on existing eBao system structure which supports the survival benefit allocation. 

## 1.2 Prerequisites
- Payment start date of the Survival Benefit is decided;
    - The disbursement method of the survival benefit is confirmed.
- Policy should satisfy below eligibility criteria:
    - Policy is in-force.
    - Frozen status of policy is “N”.
- Policy benefits should satisfy below eligibility criteria:
    - Policy benefit is In-force. 
    - Policy benefit’s premium status is regular, fully paid or premium waived. That is, except premium status equal to reduced paid up, auto paid up or extended premium term assurance or PHD.
    - Policy benefit is entitled to survival benefit. Refer to Product parameter ‘Survival Benefit Code’ in Survival Benefit Type table. If the benefit code is = ‘301’, the benefit is entitled for survival benefit. Else, it is not entitled for survival benefit.
    - If there is an existing survival benefit plan in the system, system verifies that the survival benefit plan captured in the system is a valid plan. Else, system skips this check. Survival benefit plan failing either of the conditions below is considered an invalid plan.
        - The status of the plan is active.
        - The survival benefit payment end date is greater than or equal to the system date.
    - If there is a survival benefit plan created in the system, system verifies the next survival benefit payment date satisfies the criteria below. Else, system verifies the survival benefit payment start date satisfies the criteria below. 
        - The next survival benefit payment date should be less than or equal to the system date plus the days in advance;
        - If Survival Benefit option is option 1, the next survival benefit payment date should be less than or equal to the system date plus the days in advance (7 days).
        - If Survival benefit option is option 2, the next survival benefit payment date should be less than or equal to the system date plus the days in advance (1 day).
## 1.3 Survival Benefit Options
Option 1: To receive SB/CB in cash  
Option 2: To leave SB/CB on deposit with Company

## 1.4 Procedures
***Step 01.*** At a specific time every day, the system starts a batch job of survival benefit allocation.

- For survival benefit option 1, go to Step 2. 
- For survival benefit option 2, go to Step 3. 

***Step 02.*** The system checks the survival benefit due date.  

- If the current system date < survival benefit due date – the number of days in advance, the process ends. 
- If the current system date ≥ survival benefit due date – the number of days in advance, go to Step 4.  
*NOTE*: The number of days in advance for option 1 is 7. 

***Step 03.*** The system checks the survival benefit due date. 

- If the current system date < survival benefit due date – the number of days in advance, the process ends. 
- If the current system date ≥ survival benefit due date – the number of days in advance, go to Step 4.   
*NOTE*: The number of days in advance for options 2 is 1. 

***Step 04.*** If there is no survival benefit plan, the system creates the survival benefit plan and updates the plan details as follows. If there is already a survival benefit plan, the system updates the plan details as follows:

- Updates the latest survival benefit amount: Latest survival benefit amount = new survival benefit amount calculated by product. 
- Updates the survival benefit plan details
If The survival benefit is the last installment, then the system sets the survival benefit plan status to Inactive. The process ends. If the survival benefit is not the last installment, then the system updates the next survival benefit payment date.

***Step 05.*** The system updates the allocated survival benefit into the survival benefit account, and generates the survival benefit letter. 

***Step 06.*** The finance personnel perform payment to the client for option 1.

## 1.5 Business Rules
- Save Information  
    - For selected policy benefits, following information to be updated:
        - If there is no survival benefit plan, system creates the survival benefit plan and proceed to update the plan details as following. If there is already a survival benefit plan, system proceeds to update the plan details as following.
        - Update the latest survival benefit amount: 
            - Latest survival benefit allocation = new survival benefit amount calculated by Product according to below formula:  

[//]: # (`$$newSurvivalBenefitAmount={SA * SB_Pay_Amount}/{SB_Unit_Payment}$$  `)
[//]: # (*NOTE*: The formula is defined in product configuration. SB_Unit_Payment=1000. SB_Pay_Amount is configured in product definition.  )

        - Updates the survival benefit plan details:
            - If the survival benefit is the last installment, system sets the survival benefit plan to ‘Inactive’ status and the ‘save information’ action ends. Else, system proceeds to 3.1.3.2.
            - Update next survival benefit payment date: 
Next Survival Benefit Payment Date is derived from the product parameter policy months in the ‘SB-AI-MB’ table in the product parameter spreadsheet. 
                - For example, for product code ‘GEM0191’, if the benefit commencement date is 01/01/2005, the survival benefit payment start date will be set as the 01/01/2006. 
                - Assuming the survival benefit is on option 2, on 31/12/2005, system allocates the survival benefit and updates the next survival benefit payment date to 01/01/2007.
- Calculate interests  
System calculate compound interests for SB account every year on interest calculation date. The SB account interests are calculated by below formula:
- 
[//]: # ($$Interest={policyAccount.interestPrinciple*rate} – {policyAccount.interestPrinciple}$$  )

| Parameter | Formula              | Description                                                                                                                                                       |
|-----------|:---------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Rate      | rate=(1+r)^(f*d/365) | -                                                                                                                                                                 |
| r         | -                    | r is interestRate, can query in t_service_rate by rateType                                                                                                        |
| f         | f=1/-1               | If current interest calculation date > last interest calculation date, f=1. <br></br>If current interest calculation date < last interest calculation date, f=-1. |
| d         | -                    | d is the number of days after last interest calculation date.                                                                                                     |

## 1.6 User Cases
## # Case 1: Policy Whose SB Pay Option is Option 1
- Preconditions:
    - Main Benefit=GEM0191
    - Survival benefit option=1;
    - Policy status=inforce;
    - Policy commencement date=2022-10-17
- Process date & expected results:  

| Process Date | New SB Amount                  | SB Balance           | SB Next Due Date                   |
|--------------|:-------------------------------|:---------------------|:-----------------------------------|
| 2022-10-17   | -                              | 0                    | 2023-10-17 (SB payment start date) |
| 2023-10-11   | 100,000.00*11.104/1000=1110.40 | 0 (after withdrawal) | 2024-10-17                         |
| 2024-10-11   | 100,000.00*11.104/1000=1110.40 | 0 (after withdrawal) | 2025-10-17                         |
| 2025-10-11   | 100,000.00*11.104/1000=1110.40 | 0 (after withdrawal) | 2026-10-17                         |
| 2026-10-11   | 100,000.00*11.104/1000=1110.40 | 0 (after withdrawal) | -                                  |

## # Case 2: Policy Whose SB Pay Option is Option 2
- Preconditions:
    - Main Benefit=GEM0168
    - Survival benefit option=2;
    - Policy status=inforce;
    - Customer entry age=55;
    - Policy commencement date=2022-10-17 
- Process date & expected results:  

| Process Date | New SB Amount                  | SB Balance                              | SB Next Due Date                   |
|--------------|:-------------------------------|:----------------------------------------|:-----------------------------------|
| 2022-10-17   | -                              | 0                                       | 2023-10-17 (SB payment start date) |
| 2023-10-11   | 100,000.00*11.104/1000=1110.40 | 1110.4                                  | 2024-10-17                         |
| 2024-10-11   | 100,000.00*11.104/1000=1110.40 | 1110.40+11.13(interest)+1110.40=2231.93 | 2025-10-17                         |
| 2025-10-11   | 100,000.00*11.104/1000=1110.40 | 2231.93+22.32(interest)+1110.40=3364.65 | 2026-10-17                         |
| 2026-10-11   | 100,000.00*11.104/1000=1110.40 | 3364.65+33.65(interest)+1110.40=4508.7  | -                                  |
