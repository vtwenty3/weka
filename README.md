This report would explain and show the methods used to analyze a raw database. In order to find patterns and relationships in a data which possibly have errors, we should start with preprocessing the whole database, which includes converting the data to use-ful format, converting and changing values, removing spelling errors, fixing outliers and improving the overall data consistency. Tools used: OpenRefine, Weka;

## Data Cealning
This step aims to correct inconsistencies, typos and improve readability and usability of the data. This pre-processing step is crucial for working efficiently with the data.
Firstly, the approach used when encountering missing data was replacing it with key-word “missing” to indicate that there is a problem with the current row but not com-pletely delete the record, which ensures that the rest of the information of the case is available. Changing the value to a mean value one can result in unexpected results, bad decision based on wrong data etc.
After testing with several algorithms, I have decided that the more efficient and simple way is simply to delete the fields which include values that are not easily predictable.
2
“Id” column is not deleted originally, it can be considered as a column that is not provid-ing real value to the database but having an additional id can be helpful in some cases. When dataset is used in Weka “id” column is removed, as this improves the accuracy of the model.

### Steps taken in pre-processing:
1. Adding a header as it was not present in the dataset provided. Renaming a few columns for better readability. See Table 1.
2. Correcting errors, outliers and cleaning the data. Deleting rows which cannot be used properly for the training. See Table 2.
3. Converting the data to nominal values and simplifying it for better training and accuracy. See Data conversion and simplification.
![image](https://user-images.githubusercontent.com/61486268/215581009-94b8f732-5a4f-4c73-b134-e6f798bac63e.png)

### Data conversion and simplification
Column “debth_status” attributes “all paid”, “ no credits/all paid” and “existing_paid” combined with “all paid”.
Column “credit_reason” attributes “used car” and “new car” to car;
attributes “domenestic appliance” and “furniture/equipment” to “furniture”;
attribute “retraining” to “other”;
Column “credit_amount” modified with python code to nominal values using the same logic as employment – “1K<=X<2K” meaning the credit amount is from 1000 (incl.) to 2000 (excl.). Not using raw numbers, because they can be hard to read when visual-izing a tree graph. 1K step until 5K, then 5<=X<10K and X>10K.
Column “age” modified with python code to nominal values using the same logic as employment – Split to below 25, 25 to 30 (example in database 25<=X<30), 30 to 40, 50 to 50, above 50.
Column “job_status” attributes “high qualif/self emp/mgmt.” to highly skilled;
“unemp/unskilled non res” and “unskilled resident” to “unskilled”.
After all the preprocessing is completed the dataset is saved in csv format and converted using Weka to .arff format.

## Data Analytics
### Classification
J48 algorithm is used as it provides high accuracy, tree visualization and a lot of pa-rameters to choose and tweak. This can result in highly accurate model with tree struc-ture which is easy to read, understand and extract rules from.
After much experimenting the parameters of J48 algorithm and highly pre-processed dataset I was able to achieve an accuracy of 99.49%. This was achieved without prun-ing and minimum instances per leaf = 1. This results in highly complex tree with 755 leaves. Unfortunately, this tree cannot be properly visualized and analyzed due to its size and impossible readability.
The complexity of the tree can be easily controlled by two parameters “confidenceFac-tor” and “minNumObj”. For simpler, easier to read tree which can be analyzed and visualized the parameters (settings) changed are:
minNumObj
15
Higher number for less nodes with more instances
confidenceFactor
0.5
Lower numbers introduce more pruning and low accuracy
This results in much simpler, less accurate tree, but it is properly displayed. I have tried to find the balance between complexity and readability, but I could not get a per-fect result.
Rule 1:
If “account_status = no checking” then loan_status = “yes” (387/46)
If the client’s account status is “no checking” then the loan would be most likely given. 387 instances apply to this rule, where 46 are incorrectly classified.
Rule 2:
If “account status = 0<=X<200” and credit amount “X>10K” then “no” (18/3)
If the client’s account status is between 0 and 200 and he wants a credit above 10K then he would be rejected. 3 of 18 cases has been wrongly classified by the model.
Rule 3:
If “account status = 1K<X<2K” and credit amount is “1K<=X<2K” then “yes” (76/28)
If the client’s account status is between 1000 and 2000 and he wants a credit between 1000 and 2000 then its highly likely he would be given this credit.
Rule 4:
If “account status = <0” and “debth_status = delayed previously” then “no” (12/3)
If the client delayed his credit payment, he would not be given a credit.
5
Rule 5:
If “account status = <0” and “debth_status = critical/other” then yes (67/18)
If the clients debth status is critical or other he would be given a credit
Rule 6,7,8:
If “account status = <0” and “debth status = paid” then:
If “savings amount = > 1000” then “yes” (4/0)
If the client has savings above 1000 then he would be given a credit.
If “savings amount = 500<=X<1000” then yes (6/1)
The client has savings between 500 and 1000 he would be given a credit.
If “saving amount < 100” and “job status = highly skilled” then yes (19/7)
The client has savings below 100 but is highly skilled he would be given a credit.

### Association
Algorithm Used - Apriori
Settings used – Rules – 6
Rule 1:
account_status=no checking credit_reason=radio/tv 124 ==> loan_status=yes 117 <conf:(0.94)> lift:(1.34) lev:(0.03) [29] conv:(4.62)
If the client does not have a bank account with the current bank, his credit reason is radio/tv and his loan status is yes he have a 94% chance to get a loan.
Rule 2:
account_status=no checking debt_status=critical/other existing credit 152 ==> loan_status=yes 142 <conf:(0.93)> lift:(1.33) lev:(0.04) [35] conv:(4.12)
If the client has existing credit with other bank but not with the current he is 93% and his loan status is yes, he is 93% likely to get a credit
Rule 3:
account_status=no checking employment=>=7 113 ==> loan_status=yes 105 <conf:(0.93)> lift:(1.32) lev:(0.03) [25] conv:(3.74)
If the client has no current bank account with the current bank, he is employed for more than 7 years and his loan status is yes then his chance for a loan is 93%.
Rule 4:
account_status=no checking gender:family=male single job_status=skilled 149 ==> loan_status=yes 137 <conf:(0.92)> lift:(1.31) lev:(0.03) [32] conv:(3.42)
if the client has no current bank account with the current bank and he is a single male with skilled job status and his loan_status is yes then he is 92% likely to get a loan
6
Rule 5:
account_status=no checking age=30<=X<40 145 ==> loan_status=yes 132 <conf:(0.91)> lift:(1.3) lev:(0.03) [30] conv:(3.09)
If the client does not have a bank account with the current bank, he is between 30 and 40 years old and his loan status is yes, then he is 91% likely to get a loan.
Rule 6:
account_status=no checking credit_amount=1K<=X<2K 126 ==> loan_status=yes 114 <conf:(0.9)> lift:(1.29) lev:(0.03) [25] conv:(2.89)
If the client has no current bank account with the current bank, his credit amount is between 1000 and 2000 and his loan status is yes then he is 90% likely to get a loan.

## Clustering

Algorithm Used - simplekmeans
Settings Used – Default with Clusters set to 6
![image](https://user-images.githubusercontent.com/61486268/215581377-f77f8b01-f941-4ad3-889f-57499ec99ff6.png)

 Clustered Instances:

0 129 ( 13%)
1 87 ( 9%)
2 212 ( 22%)
3 206 ( 21%)
4 255 ( 26%)
5 94 ( 10%)
![image](https://user-images.githubusercontent.com/61486268/215581479-59829088-7573-4f73-bcc2-fad62ce261d9.png)



