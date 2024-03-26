
Services
1.      CustomerID: A unique ID that identifies each customer.

Quarter: The fiscal quarter that the data has been derived from (e.g. Q3).
4.      Tenure in Months: Indicates the total amount of months that the customer has been with the company by the end of the quarter specified above.
5.      Offer: Identifies the last marketing offer that the customer accepted, if applicable. Values include None, A, B, C, D, and E. Coupon
6.      Phone Service: Indicates if the customer subscribes to home phone service with the company: Yes, No
7.      Internet Service: Indicates if the customer subscribes to Internet service with the company: No, DSL, Fiber Optic, Cable.
Streaming TV: Indicates if the customer uses their Internet service to stream television programing from a third party provider: Yes, No. The company does not charge an additional fee for this service.
Streaming Movies: Indicates if the customer uses their Internet service to stream movies from a third party provider: Yes, No. The company does not charge an additional fee for this service.
Streaming Music: Indicates if the customer uses their Internet service to stream music from a third party provider: Yes, No. The company does not charge an additional fee for this service.r

8.      Avg Monthly GB Download: Indicates the customer’s average download volume in gigabytes, calculated to the end of the quarter specified above.
9.      Contract: Indicates the customer’s current contract type: Month-to-Month, One_Year, Two_Year.
10.   Payment Method: Indicates how the customer pays their bill: 
Bank_Withdrawal,
 Credit_Card,
 Mailed_Check
11.   Monthly Charge: Indicates the customer’s current total monthly charge for all their services from the company.
12.   Total Charges: driven Indicates the customer’s total charges, calculated to the end of the quarter specified above. – calculated using the monthly charge

Status
 CustomerID: A unique ID that identifies each customer. 
Quarter: The fiscal quarter that the data has been derived from (e.g. Q3). which quartile in year is this 

Satisfaction Score: A customer’s overall satisfaction rating of the company from 1 (Very Unsatisfied) to 5 (Very Satisfied). ---- التصنيف للخدمة

Customer Status: Indicates the status of the customer at the end of the quarter: Churned, Stayed, or Joined ----- العميل موجود و لا مشي

Churn Category: 
A high-level category for the customer’s reason for churning: Attitude, Competitor, Dissatisfaction, Other, Price. When they leave the company, all customers are asked about their reasons for leaving. Directly related to Churn Reason. why left the servece: Attitude,
 Competitor, 
Dissatisfaction,
 Other, 
Price

Questions and queries:
1.      Is there a relationship between Gender and Churn? DONE
2.  Which MartialStatus has the highest Churn rate?DONE
Demographics.@attr
Status.customer_status
--stored procedure to check if there is a correlation between any column in the demographics and the churn
GO
CREATE OR ALTER PROC GET_CORR_DEMO_CHURN__(@attr VARCHAR(20))
AS
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    SET @sql = N'
    SELECT COUNT(' + QUOTENAME(@attr) + ') AS [Count] , ' + QUOTENAME(@attr) + ' , s.Customer_Status
    FROM Demographics d
    JOIN Status s ON d.CustomerID = s.CustomerID
    GROUP BY s.Customer_Status, ' + QUOTENAME(@attr) + ';';
    EXEC sp_executesql @sql;
	RETURN
END
GO
EXEC GET_CORR_DEMO_CHURN__'Married'
EXEC GET_CORR_DEMO_CHURN__'Senior_Citizen'
EXEC GET_CORR_DEMO_CHURN__'Gender'
3.    Which Gender has more consumption?DONE
Demographics.gender
Services.Gigabyte_Downloads
SELECT gender, AVG(Avg_Monthly_GB_Download) [Average Monthly GB Download for all Gender] 
FROM Demographics d, Service s
WHERE d.CustomerID = s.CustomerID
GROUP BY Gender
4.	Which city has higher tenure and consumption?DONE
Demographics.zip_code
Customer_location.zip_code, Customer_location.city, Customer_location.state, Customer_location.country
Services.tenure
Services.Gigabyte_Downloads
CREATE OR ALTER VIEW city_tenure_consumption(city, state, country, average_tenure, average_consumption)
AS
	SELECT l.city, l.state, l.country, AVG(s.Tenure_in_Months) AS average_tenure, AVG(s.Avg_Monthly_GB_Download) AS average_consumption 
	FROM Location l, Demographics d, Service s
	WHERE l.Zip_Code = d.Zip_Code AND d.CustomerID = s.CustomerID
	GROUP BY l.city, l.state, l.country

GO
SELECT * FROM city_tenure_consumption
ORDER BY average_tenure DESC, average_consumption DESC

SELECT city, MAX(average_tenure) [Highest Tenure], MAX(average_consumption) [Highest Consumption] FROM city_tenure_consumption
GROUP BY city

SELECT TOP(5) city, average_tenure, average_consumption FROM city_tenure_consumption
ORDER BY average_tenure DESC, average_consumption DESC
5.	Does the Customer with the Highest SatisfactionScore have high consumption?
	Status.Satisfaction_Score
	Services.Gigabyte_Downloads
	Status.customerID, Services.customerID
—done





6.	What is the average tenure of customers based on their contract type? &&&&&
	AVG(Services.tenure)
	Services.contract
done



7.	How does the average monthly charge vary among different payment methods? &&&&&
	AVG(Services.Monthly_Charge)
	Services.Payment Method
done
8.	Which quarter experienced the highest rate churned rate?
Services.Quarter
Status.Satisfaction_Score
Status.Customer_Status
done
9.	Is there a relationship between the number of dependents (or age) and churn?
10.	How does the churn rate differ among different age groups?
Status.Customer_Status
Demographics.number_of_dependents or Demographics.Age
CREATE OR ALTER PROC GET_CORR_DEMOntile_CHURN__(@attr VARCHAR(20))
AS
BEGIN
	DECLARE @sql1 NVARCHAR(MAX); 

    SET @sql1 = N'
    SELECT ' + QUOTENAME(@attr) + ', s.Customer_Status,
	NTILE(3) OVER (PARTITION BY Customer_Status ORDER BY '+ QUOTENAME(@attr) +') AS nt
    FROM Demographics d
    JOIN Status s ON d.CustomerID = s.CustomerID';

    EXEC sp_executesql @sql1;

	RETURN
END

GO
DECLARE @cont_nt TABLE
(
	attr INT,
	Customer_Status VARCHAR(20),
	nt INT
)
INSERT INTO @cont_nt
EXEC GET_CORR_DEMOntile_CHURN__ 'Age'
SELECT AVG(attr) AS [Average Age], nt, COUNT(Customer_Status) [Count Churned] FROM @cont_nt
WHERE Customer_Status='Churned'
GROUP BY nt

SELECT AVG(attr) AS [Average Age], nt, COUNT(Customer_Status) [Count Stayed] FROM @cont_nt
WHERE Customer_Status='Stayed'
GROUP BY nt

– We can do the same with number of dependents instead of age


11.	What is the distribution of satisfaction scores across different contract types?
Status.Satisfaction_Score
Services.contract
12.	Do customers who stream movies have higher tenure compared to those who don't? &&&&&
Services.Streaming_Movies
Services.Tenure
13.	Is there a difference in average monthly charges between senior citizens and non-senior citizens? 
Demographics.Senior_Citizen
Services.Monthly_Charge
14.	Which state has the highest churn rate?
Customer_Location.State
Status.Customer_Status
CREATE VIEW highest_churn_rate 
WITH ENCRYPTION
AS
SELECT top 1
  s.Customer_Status,
  COUNT(*) AS total_customers,
  SUM(CASE WHEN s.Customer_Status = 'Churned' THEN 1 ELSE 0 END) AS churned_customers,
  (SUM(CASE WHEN s.Customer_Status = 'Churned' THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS churn_rate
FROM
  Status s
JOIN
  Demographics d ON s.CustomerID = d.CustomerID
JOIN
  Location l ON d.Zip_Code = l.Zip_Code
GROUP BY
  s.Customer_Status
ORDER BY
  churn_rate DESC;
SELECT * FROM highest_churn_rate


15.	What is the average tenure for customers who use streaming TV versus those who don't? &&&&&DONE
Services.tenure
Services.streamingTV
—
  CREATE PROCEDURE pp
AS
BEGIN
  SELECT
    Streaming_TV,
    AVG(Tenure_in_Months) AS average_tenure
  FROM
    Service
  GROUP BY
    Streaming_TV;
END;
Exec pp



16.	How does the churn rate vary among different internet service types? 
SELECT
  S.Internet_Service,
  COUNT(*) AS total_customers,
  SUM(CASE WHEN Status.Customer_Status = 'Churned' THEN 1 ELSE 0 END) AS churned_customers
FROM
Service s
JOIN
  Status ON s.CustomerID = Status.CustomerID
GROUP BY
  S.Internet_Service




17.	What is the average monthly download volume for each internet service type? &&&&&
Services.Internet_Service
Services.GB_download
CREATE VIEW AverageDownloadView AS 
SELECT DISTINCT
  Internet_Service,
  AVG(Avg_Monthly_GB_Download) OVER (PARTITION BY Internet_Service) AS Avg_Monthly_GB_Download
FROM
  service;

  SELECT * FROM AverageDownloadView;







17.	Is there a correlation between satisfaction scores and churn categories?--------
Status.Satisfaction_Score
Status.Churn_Category
18.	How does the churn rate change based on the type of offers provided?
Status.Customer_Status
Services.Offer
From this query I noted that Offer (A) is one of the offers that caused customers to leave the company.
Create or alter view VrateOffers
As
select st.Customer_Status , sr.Offer
from Status st,Service sr
where sr.CustomerID=st.CustomerID and Customer_Status='Churned'
           select * from VrateOffers



19.	Which payment method is most preferred among customers with high satisfaction scores?
Services.Payment_Method
Status.Satisfaction_Score
From this query I noted that (Mailed_Check) is one of the Payment Methods that  preferred among customers with high satisfaction scores.
           create or alter view Vpayscore
as
select sr.Payment_Method,st.Satisfaction_Score 
from Service sr,Status st
where sr.CustomerID=st.CustomerID and Satisfaction_Score in ('4','5')
select * from Vpayscore
20.	Is there a difference in tenure between customers who churned and those who stayed?
Status.Customer_Status
Services.Tenure
From this query I noted that there is no  much difference in this case, there are the same in tenure.
           create or alter view Vtenure 
as
select sr.Tenure_in_Months, st.Customer_Status
from Service sr,Status st
where sr.CustomerID=st.CustomerID and Tenure_in_Months in ('24','18')
select * from Vtenure
21.	What is the average total charge for customers in different cities?done
Services.Total_Charges
Location.City
Location.zip_code
Demographics.zip_code
22.	Do customers with a higher number of dependents have higher satisfaction scores?done
Demographics.Number_of_Dependents
Status.Satisfaction_Score
23.	What is the percentage distribution of churn categories among churned customers?done
COUNT(Status.Churn_Category)
Order them descendingly
Calculate percentage
24.	Is there a correlation between satisfaction scores and the presence of streaming music services?
	Status.Satisfaction_Score
	Services.Streaming_Music

Create an audit table if the customer changed his/her location
--Audit Table for updating customer location
CREATE TABLE loc_history
(
	CustomerID INT,
	UserName VARCHAR(50),
	ModifiedDate DATE,
	Zip_Old INT,
	Zip_New INT
)
GO
CREATE OR ALTER TRIGGER T_loc
ON Demographics
AFTER UPDATE 
AS
	IF UPDATE(Zip_code)
	BEGIN
		DECLARE @old_zip INT , @new_zip INT, @cust_id INT

		DECLARE loc_cursor CURSOR FOR --I used cursor because multiple rows may be updated at the same time
		SELECT Zip_Code, CustomerID FROM deleted

		OPEN loc_cursor
		FETCH NEXT FROM loc_cursor INTO @old_zip, @cust_id
		
		WHILE @@FETCH_STATUS = 0
		BEGIN
			SELECT @new_zip=Zip_Code FROM inserted WHERE CustomerID=@cust_id

			INSERT INTO loc_history VALUES (@cust_id, SUSER_NAME(), GETDATE(), @old_zip, @new_zip)
			
			FETCH NEXT FROM loc_cursor INTO @old_zip, @cust_id
		END

		CLOSE loc_cursor
		DEALLOCATE loc_cursor				
	END




Triggers:
Prevent Delete From Demographics
Prevent Delete From Service
Prevent Delete From Service

Rules:
1.	Offer_rule:
Make rule on offer table, values in the column is just in this options ('None', 'A', 'B', 'C', 'D', 'E', 'Coupon').



2.	Phone_Service_Rule:
Rue that the values in column is ('Yes', 'No')

