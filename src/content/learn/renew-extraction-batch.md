This section describes the renew extraction batch job.
## 1.1 Description
This document describes how Gemini calculates the amount to bill for all renew policies including the policy which the premium status is waived by batch. This is a daily batch and will take into consideration the premium due. The system creates a record for the calculated amount.
## 1.2 Prerequisites
Eligibility criteria for premium renew policies:

- Policy status=in force
- Benefit premium status is regular or premium waived.
- Payment frequency=installment.
- Extract policies with system date>=due date-leading time (defined in rate table-configured in LIMO).

The policy is not eligible if:

- The policy has been frozen by Claims or Custom Service items.
- The next due date of a policy does not meet the extraction criteria.
- The policy has been extracted before.
- The payment method is unit deduction ride.
- The policy is in premium holiday.

## 1.3 Procedures
***Step 01.*** On a daily basis, system select policies which are due for billing to run renew extraction batch.  
***Step 02.*** System computes the renewal premium amount to bill and update related information.  

## 1.4 Business Rules
**System computes the renewal premium amount to bill:**
 
- If premium status is regular or premium waived:
    - Calculate the next renewal premium due as specified by the product.
        - If the benefit fulfills any following conditions, need to recalculate the premium 
            - Calculation method of benefit < > Non-scheduled premium or using Premium to calculate the Sum Assured and Benefit = non-level premium and Calculation Method = Using Sum Assured to calculate premium.
            - Calculation method of benefit < > Non-scheduled premium or using Premium to calculate the Sum Assured and Calculation of premium amount is defined as that based on remaining term and Calculation Method = Using Sum Assured to calculate premium.
            - Calculation method of benefit < > Non-scheduled premium or using Premium to calculate the Sum Assured and Benefit = Level premium and non-guarantee premium and Calculation Method = Using Sum Assured to calculate premium.
            - When the benefit is with loading, system to recalculate the extra premium of the benefit.
            - There is a pre-contract information
            - Benefit is a waiver product and Calculation Method = Using Sum Assured to calculate premium
            - For waiver of premium / payer benefit, Premium, Sum Assured waived, Annual Amount waived and Initial Sum Assured need to be recalculated.
        - If benefit does not fulfill the above rule, system is to use the next premium due amount recorded at benefit level as the next premium due amount.
        - For ILP policy, system should check whether there is recurring top-up. If there is, system will generate the recurring top-up record together with the premium due amount.
        - If there are more than one premium due between the next due date and the system date, the premium due record should be generated separately for each premium due amount.

**System saves information:**

- If premium status is premium waived:
    - Create the waive fee record
        - Create the premium due record as
            - Due date=current due date at benefit level
            - Record status=settled 
            - Collection method is insurer payment
            - Amount equal to the amount calculated at Step 2.1.1.
        - Create the payment record as
            - Collection method=insurer payment
            - Amount=the amount calculated at Step 2.1.1.
            - Money come in date=system date
            - Status=used 
    - Move due date and update related information.
    - If benefit next due date > = waiver end date and next due date < benefit premium expiry date, to update benefit status to regular.
- If premium status is not premium waived:
    - System creates the relevant billing records.
## 1.5 User Cases
## # Case 1: Collection and extraction
- Preconditions:
    - Main Benefit=GEM0168;
    - Policy status=inforce;
    - Premium status=Regular;
    - Policy commencement date=2022-10-17;
    - Leading time=30 days
- Process date & expected results:  

| Process Date | Execution Step             | Expected Results      |
|--------------|:---------------------------|:----------------------|
| 2022-10-17   | Collect inforcing premium  | Policy status=Inforce |
| 2023-09-20   | Run renew extraction batch | Bill record created   |

## # Case 2: With extra premium added during NB underwriting
- Preconditions:
    - Main Benefit=GEM0168 (Conditionally accepted with extra premium);
    - Policy status=inforce;
    - Premium status=Regular;
    - Policy commencement date=2022-10-17;
    - Leading time=30 days
- Process date & expected results:  

| Process Date | Execution Step                                                                      | Expected Results                                                               |
|--------------|:------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------|
| 2022-10-17   | Conditionally accept the policy and add 2-year extra premium during NB underwriting | policy has 2 years of extra premium                                            |
| 2022-10-17   | Collect inforcing premium                                                           | Policy status=Inforce                                                          |
| 2023-09-20   | Run renew extraction batch                                                          | Bill record created. <br></br>Extra premium is also included in the bill       |
| 2023-09-20   | Collect renew premium                                                               | money collected into renew suspense                                            |
| 2023-09-20   | Run renew confirm batch                                                             | Policy next due date moved to the next year                                    |
| 2024-09-20   | Run renew extraction batch                                                          | Renew bill record created. <br></br>Extra premium is also included in the bill |
| 2024-09-20   | Collect renew premium                                                               | money collected into renew suspense                                            |
| 2024-09-20   | Run renew confirm batch                                                             | Policy next due date moved to the next year                                    |
| 2025-09-20   | Run renew extraction batch                                                          | Renew bill record created. <br></br>No more extra premium from this year on    |

## # Case 3: With extra premium added during CS underwriting
- Preconditions:
    - Main Benefit=GEM0168 (change smoker status with extra premium);
    - Policy status=inforce;
    - Premium status=Regular;
    - Policy commencement date=2022-10-17;
    - Leading time=30 days
- Process date & expected results:  

| Process Date | Execution Step                                                             | Expected Results                                                         |
|--------------|:---------------------------------------------------------------------------|:-------------------------------------------------------------------------|
| 2022-10-17   | Collect inforcing premium                                                  | Policy status=Inforce                                                    |
| 2022-10-17   | CS: Change Smoker Status;<br></br>Add extra premium during CS underwriting | Added extra premium to the policy                                        |
| 2023-09-20   | Run renew extraction batch                                                 | Bill record created. <br></br>Extra premium is also included in the bill |

## # Case 3: UL Recurring Top Up
- Preconditions:
    - Main Benefit=UL recurring to up;
    - Policy status=inforce;
    - Premium status=Regular;
    - Policy commencement date=2022-10-17;
    - Leading time=30 days
- Process date & expected results:  

| Process Date | Execution Step             | Expected Results                                                       |
|--------------|:---------------------------|:-----------------------------------------------------------------------|
| 2022-10-17   | Collect inforcing premium  | Policy status=Inforce                                                  |
| 2023-09-20   | Run renew extraction batch | Bill record created. <br></br>Recurring top up is included in the bill |

## # Case 4: ILP + UDR
- Preconditions:
    - Benefit=ILP+UDR
    - Policy status=inforce;
    - Premium status=Regular;
    - Policy commencement date=2022-10-17;
    - Leading time=30 days
- Process date & expected results:  

| Process Date | Execution Step             | Expected Results                                                               |
|--------------|:---------------------------|:-------------------------------------------------------------------------------|
| 2022-10-17   | Collect inforcing premium  | Policy status=Inforce                                                          |
| 2023-09-20   | Run renew extraction batch | Create the bill record <br></br>Only create bill for main product, not for UDR |

## # Case 5: Policy with multi due dates
- Preconditions:
    - Benefit=GEM0170+GEMA17
    - Payment Frequency=Yearly+Monthly
    - Policy status=inforce;
    - Premium status=Regular;
    - Policy commencement date=2022-10-17;
    - Leading time=30 days
- Process date & expected results:  

| Process Date | Execution Step             | Expected Results                                                                 |
|--------------|:---------------------------|:---------------------------------------------------------------------------------|
| 2022-10-17   | Collect inforcing premium  | Policy status=Inforce                                                            |
| 2023-11-17   | Run renew extraction batch | Create the bill record <br></br> Only generated bill for GEMA17, not for GEM0170 |
