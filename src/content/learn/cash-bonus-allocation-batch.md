This section describes the cash bonus allocation processing.
## 1.1 Description
This chapter describes how Gemini processes the cash bonus declaration and allocation Jobs by daily batch. This includes:

- Updating the latest bonus amount at policy benefit level; and   
- Updating the cash bonus account at policy benefit level; and  
- Updating the next bonus due date at policy benefit level. 
## 1.2 Prerequisites
Policy should satisfy below eligibility criteria:

- Policy is in-force.
- Frozen status of policy is “N”.

Policy benefits should satisfy below eligibility criteria:

- Policy benefit is In-force.
- Policy benefit is the cash bonus product.
- Premium status of the policy benefit is regular, fully paid, or premium waived. If not (i.e. reduced paid up, auto paid up, ETA, PHD, stop payment), the policy benefit is not eligible.
- The next premium due date of the policy benefit is greater than or equal to the next bonus due date for premium status regular or premium waived.
- The next bonus due date should be less than or equal to the system date (plus the days in advance);
- The corresponding cash bonus rate can be found.  
## 1.3 Cash Bonus Options
Option 1: To receive SB/CB in cash
Option 2: To use SB/CB to pay premium
Option 3: To leave SB/CB on deposit with Company
Option 4: Option 3 for xx years thereafter Option 2
## 1.4 Procedures
***Step 01.*** At a specific time every day, the system starts a batch job of cash bonus allocation.  

***Step 02.*** The system checks the bonus due date.  

- If the current system date < bonus due date – the number of days in advance, the process ends.  
- If the current system date ≥ bonus due date – the number of days in advance, go to Step 4.  

*NOTE:* The number of days in advance is a parameter written in program for all options. The number of days in advance is 7 for all cash bonus pay options.  

***Step 03.*** The system checks the parameter No. of Completed policy years for CB becoming payable at policy level.  

- If the duration is within No. of Completed policy years for CB becoming payable, no cash bonus will be calculated nor allocated. Batch record generated in CS info. Process ends.  
- If the duration is beyond No. of Completed policy years for CB becoming payable, the system will update latest bonus amount and put cash bonus amount of the year into the cash bonus account of which cash bonus allocated status indicator is N. Then, the system updates the related cash bonus allocated status indicator from N to Y    

***Step 04.*** The system updates the next bonus due date at policy benefit level: Next bonus due date = the original next bonus due date + 1 year.  

***Step 05.*** The system checks cash bonus pay options.

- If the pay option is option 1, system perform withdrawal form cash bonus account. 
- If the pay option is option 2, system checks if the policy has policy loan
    - If yes, use cash bonus to repay policy loan;
    - If no, cash bonus stays in CB account and generate interest year on year.
- If the pay option is option 3, leave cash bonus in CB account and generate interest year on year.

***Step 06.*** System generates the cash bonus annual statement.

***Step 07.*** For Option 1, the finance personnel perform payment to the client.

## 1.5 Business Rules
- Information update
    - For eligible policy benefits, following information to be updated:
        - Update the latest bonus amount:   
        Latest bonus amount = new bonus amount calculated by below formula:  

[//]: # ($$Latest Bonus Amount=\frac{SA*CB_Factor}{CB_Unit_Amount}$$  )

[//]: # (*NOTE*: The formula is defined in product configuration. CB_Unit_Amount=1. CB_Factor is configured in product definition.  )
    - Check the parameter “No. of Completed policy years for CB becoming payable”.
        - If the duration is still not beyond “No. of Completed policy years for CB becoming payable”, no cash bonus will be calculated nor allocated. Batch record generated in CS info. Process ends.
        - If the duration is beyond “No. of Completed policy years for CB becoming payable”, the system will update latest bonus amount and put cash bonus amount of the year into the cash bonus account of which cash bonus allocated status indicator is N. Then, the system updates the related cash bonus allocated status indicator from N to Y   
        *NOTE*: “No. of Completed policy years for CB becoming payable” is a product level parameter
    - Update the next bonus due date at policy benefit level: 
        - Next Bonus Due Date = the original Next Bonus Due Date + 1 year. 
- Calculate interests  
System calculate compound interests for CB bonus account every year on interest calculation date. The cash bonus account interests are calculated by below formula:  

[//]: # ($$Interest={policyAccount.interestPrinciple*rate} – {policyAccount.interestPrinciple}$$  )

| Parameter | Formula              | Description                                                                                                                                                        |
|-----------|:---------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Rate      | rate=(1+r)^(f*d/365) | -                                                                                                                                                                  |
| r         | -                    | r is interestRate, can query in t_service_rate by rateType                                                                                                         |
| f         | f=1/-1               | If current interest calculation date > last interest calculation date, f=1.  <br></br>If current interest calculation date < last interest calculation date, f=-1. |
| d         | -                    | d is the number of days after last interest calculation date.                                                                                                      |

## 1.6 User Cases
## # Case 1: Policy Whose CB Pay Option is Option 1 
- Preconditions:
    - Main Benefit=GEM0168
    - Cash bonus option=1;
    - Policy status=inforce;
    - Customer entry age=55;
    - Policy commencement date=2022-10-17
- Process date & expected results:  

| Process Date | Latest Bonus Amount     | CB Balance           | Bonus Next Due Date |
|--------------|:------------------------|:---------------------|:--------------------|
| 2022-10-17   | -                       | 0                    | 2023-10-17          |
| 2023-10-11   | -                       | 0                    | 2024-10-17          |
| 2024-10-11   | 10,000*0.0031/1=310.00  | 0 (after withdrawal) | 2025-10-17          |
| 2025-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2026-10-17          |
| 2026-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2027-10-17          |

## # Case 2: Policy Whose CB Pay Option is Option 2
- Preconditions:
    - Main Benefit=GEM0168
    - Cash bonus option=2;
    - Policy status=inforce;
    - Customer entry age=55;
    - Policy commencement date=2022-10-17
    - Policy raised APL on 2026-12-26 in non-forfeiture batch running, and APL is not repaid until 2027-10-17. 
- Process date & expected results:  

| Process Date | Latest Bonus Amount                                                                            | CB Balance                                                   | Bonus Next Due Date |
|:-------------|:-----------------------------------------------------------------------------------------------|:-------------------------------------------------------------|:--------------------|
| 2022-10-17   | -                                                                                              | 0                                                            | 2023-10-17          |
| 2023-10-11   | -                                                                                              | 0                                                            | 2024-10-17          |
| 2024-10-11   | 10,000*0.0031/1=310                                                                            | 310                                                          | 2025-10-17          |
| 2025-10-11   | 10,000*0.00535/1=535                                                                           | 310+3.1(interest)+535=848.1                                  | 2026-10-17          |
| 2026-10-11   | 10,000*0.00769/1=769                                                                           | 313.1+535+8.48(interest)+769=1625.58                         | 2027-10-17          |
| 2026-12-02   | -                                                                                              | 0 (CB used to repay APL. APL Balance=9082.2-1625.58=7456.62) | 2027-10-17          |
| 2027-10-11   | 10,000*0.01013/1=1013	0 (CB allocated and used to repay APL. APL Balance=7456.62-1013=6443.62) |                                                              | 2028-10-17          |

## # Case 3: Policy Whose CB Pay Option is Option 3
- Preconditions:
    - Main Benefit=GEM0168
    - Cash bonus option=3;
    - Policy status=inforce;
    - Customer entry age=55;
    - Policy commencement date=2022-10-17
- Process date & expected results:  

| Process Date | Latest Bonus Amount  | CB Balance                           | Bonus Next Due Date |
|:-------------|:---------------------|:-------------------------------------|:--------------------|
| 2022-10-17   | -                    | 0                                    | 2023-10-17          |
| 2023-10-11   | -                    | 0                                    | 2024-10-17          |
| 2024-10-11   | 10,000*0.0031/1=310  | 310                                  | 2025-10-17          |
| 2025-10-11   | 10,000*0.00535/1=535 | 310+3.1(interest)+535=848.1          | 2026-10-17          |
| 2026-10-11   | 10,000*0.00769/1=769 | 313.1+535+8.48(interest)+769=1625.58 | 2027-10-17          |

## # Case 4: Fully-paid Policy
- Preconditions:
    - Main Benefit=GEM0168
    - Cash bonus option=1;
    - Policy status=inforce;
    - Customer entry age=55;
    - Premium status=Fully-paid
    - Policy commencement date=2022-10-17
- Process date & expected results:  

| Process Date | Latest Bonus Amount     | CB Balance           | Bonus Next Due Date |
|--------------|:------------------------|:---------------------|:--------------------|
| 2022-10-17   | -                       | 0                    | 2023-10-17          |
| 2023-10-11   | -                       | 0                    | 2024-10-17          |
| 2024-10-11   | 10,000*0.0031/1=310.00  | 0 (after withdrawal) | 2025-10-17          |
| 2025-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2026-10-17          |
| 2026-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2027-10-17          |

## # Case 5: Premium Waived Policy
- Preconditions:
    - Main Benefit=GEM0168
    - Cash bonus option=1;
    - Policy status=inforce;
    - Customer entry age=55;
    - Premium status=Premium waived
    - Policy commencement date=2022-10-17
- Process date & expected results:  

| Process Date | Latest Bonus Amount     | CB Balance           | Bonus Next Due Date |
|--------------|:------------------------|:---------------------|:--------------------|
| 2022-10-17   | -                       | 0                    | 2023-10-17          |
| 2023-10-11   | -                       | 0                    | 2024-10-17          |
| 2024-10-11   | 10,000*0.0031/1=310.00  | 0 (after withdrawal) | 2025-10-17          |
| 2025-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2026-10-17          |
| 2026-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2027-10-17          |

## # Case 6:Reduced Paid-up Policy
- Preconditions:
    - Main Benefit=GEM0168
    - Cash bonus option=1;
    - Policy status=inforce;
    - Customer entry age=55;
    - Premium status=Reduced Paid-up
    - Policy commencement date=2022-10-17
- Process date & expected results:  

| Process Date | Latest Bonus Amount     | CB Balance           | Bonus Next Due Date |
|--------------|:------------------------|:---------------------|:--------------------|
| 2022-10-17   | -                       | 0                    | 2023-10-17          |
| 2023-10-11   | -                       | 0                    | 2024-10-17          |
| 2024-10-11   | 10,000*0.0031/1=310.00  | 0 (after withdrawal) | 2025-10-17          |
| 2025-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2026-10-17          |
| 2026-10-11   | 10,000*0.00535/1=535.00 | 0 (after withdrawal) | 2027-10-17          |
