# ZOMATO ANALYSIS USING MySQL

- USE  ZOMATO_ANALYSIS  ;

- SELECT * FROM COUNTRY_TABLE ;

- SELECT * FROM ZOMATO_TABLE ;


<img width="198" height="87" alt="image" src="https://github.com/user-attachments/assets/658f7370-6156-49d6-b8bb-b59ac8a1df52" />


------------------------------------------------------------------------------------------------------------------

### # No. of Restaurants
select 
count(restaurantid) as No_of_restaurants
from zomato_table;

**RESULT :**

<img width="193" height="75" alt="image" src="https://github.com/user-attachments/assets/39b27ece-0d6a-4a92-8ae8-5f2c479cf99e" />


---------------------------------------------------------------------------

### # No. of Cities
select
count(distinct(city)) as No_of_cities
from zomato_table ;

**RESULT :**

<img width="147" height="86" alt="image" src="https://github.com/user-attachments/assets/ed1810b4-f014-4b51-8007-fdfb64c6f53c" />


----------------------------------------------------------------------------

### # No. of Countries
select
count(distinct(CountryCode)) as No_of_Countries
from zomato_table ;

**RESULT :**

<img width="185" height="75" alt="image" src="https://github.com/user-attachments/assets/1d8d7165-65b2-4909-9ecf-6bdea0162a59" />


-----------------------------------------------------------------------------

### # No. of cuisines
select count(distinct(cuisines)) as No_of_cuisines
from zomato_table ;

**RESULT :**

<img width="167" height="67" alt="image" src="https://github.com/user-attachments/assets/8e5cbb6b-284a-4338-927d-9a04d499dbf7" />


-------------------------------------------------------------------------------


### 1) COUNTRY TABLE 

SELECT DISTINCT(A.COUNTRYCODE) , B.COUNTRY_NAME

FROM zomato_table AS A LEFT JOIN country_table AS B

ON A.CountryCode=B.Country_Code

order by CountryCode asc ;

**RESULT :**


<img width="374" height="362" alt="image" src="https://github.com/user-attachments/assets/7679f2eb-6082-4f05-82ef-f39395ceea5c" />


--------------------------------------------------------------------------------------------------------------------------

### 2) CALENDER TABLE

SELECT

		 distinct(Datekey_Opening) , 
		 
         year(Datekey_Opening) AS YEAR ,
		 
         month(Datekey_Opening) AS MONTH_NO ,
		 
         monthname(Datekey_Opening) AS MONTH_NAME ,
		 
         concat("Q" , quarter(Datekey_Opening) ) AS QUARTER ,
		 
         CONCAT(year(Datekey_Opening),"-",DATE_FORMAT(Datekey_Opening,"%b")) AS YR_MTH ,
		 
         weekday(Datekey_Opening) AS WEEKDAY_NO ,
		 
         dayname(Datekey_Opening) AS WEEKDAY_NAME ,
		 
         CONCAT("FM","-", month(adddate(Datekey_Opening,interval -3 month)) ) AS FINANCIAL_MONTH ,
		 
		 CONCAT("FQ","-", QUARTER(adddate(Datekey_Opening,interval -1 QUARTER)) ) AS FINANCIAL_QUARTER
		 
FROM zomato_table

order by Datekey_Opening ASC ;

**RESULT :**

<img width="1123" height="293" alt="image" src="https://github.com/user-attachments/assets/c570f078-032c-40c5-9a5b-399948920378" />



------------------------------------------------------------------------------------------------------------------------------------------------------

### 3) NUMBER OF RESTAURENTS BASED ON COUNTRY AND CITY 


WITH CITIES AS

     ( 
	 
        SELECT  
		
               A.COUNTRYCODE AS CONTRY_CODE ,
			   
               B.COUNTRY_NAME AS CONTRY_NAME ,
			   
	           A.CITY AS CITY,
			   
	           COUNT(A.RestaurantID) AS NO_RES_CITY
			   
        FROM ZOMATO_TABLE AS A LEFT JOIN country_table AS B
		
        ON A.CountryCode = B.Country_Code
		
        GROUP BY A.COUNTRYCODE , B.Country_Name , A.CITY 
		
        ORDER BY A.COUNTRYCODE , B.Country_Name , A.CITY ASC 
		
	  ) 
	  

SELECT  

       SUM(NO_RES_CITY) OVER () AS TOTAL_NO_RES ,
	   
       CONTRY_CODE ,
	   
       CONTRY_NAME ,
	   
       SUM(NO_RES_CITY) OVER ( partition by CONTRY_CODE , CONTRY_NAME) AS NO_RES_COUNTRY ,
	   
       CITY ,
	   
       NO_RES_CITY 
	   
	   
 FROM CITIES ;

 **RESULT :**

 <img width="777" height="290" alt="image" src="https://github.com/user-attachments/assets/5bfe5a95-ce23-4314-9de9-bc586be72276" />

 

---------------------------------------------------------------------------------------------------------------------------------------------------

### 4) NUMBER OF RESTAURENTS OPENED BASED ON YEAR , QUARTER , MONTH 

SELECT DISTINCT

    Total_Restaurants,
	
    Opening_Year,
	
    Year_Count,
	
    Opening_Quarter,
	
    Quarter_Count,
	
    Opening_Month,
	
    Month_Count
	
    
FROM (

        SELECT
		
               YEAR(Datekey_Opening) AS opening_year,
			   
               QUARTER(Datekey_Opening) AS opening_quarter,
			   
               MONTH(Datekey_Opening) AS opening_month,
			   

               COUNT(RESTAURANTID) OVER ( PARTITION BY YEAR(Datekey_Opening) ) AS year_count,
			   

               COUNT(RESTAURANTID) OVER ( PARTITION BY YEAR(Datekey_Opening), QUARTER(Datekey_Opening) ) AS quarter_count,
			   

               COUNT(RESTAURANTID) OVER ( PARTITION BY YEAR(Datekey_Opening), QUARTER(Datekey_Opening), MONTH(Datekey_Opening) ) AS month_count,
			   
               
			   COUNT(RESTAURANTID) OVER () AS total_restaurants
			   
        
        FROM ZOMATO_TABLE
		
        
     ) AS OPENING_TABLE
	 

ORDER BY opening_year , opening_quarter , opening_month ;

**RESULT :**

<img width="822" height="292" alt="image" src="https://github.com/user-attachments/assets/412a260f-8200-4fde-9388-71ef27710bde" />




------------------------------------------------------------------------------------------------------------------------------------------------------

### 5) Count of Resturants based on Average Ratings				

select

case

	 when rating >= 1 and rating<2  then "1-2" 
	 
     when rating >= 2 and rating<3  then "2-3" 
	 
     when rating >= 3 and rating<4  then "3-4" 
	 
     when rating >= 4 and rating<=5 then "4-5" 
	 
end as Ratings ,

count(restaurantid) as Restaurant_Count

from zomato_table 

group by ratings 

order by ratings asc;

**RESULT :**

<img width="226" height="127" alt="image" src="https://github.com/user-attachments/assets/873b589d-ecef-4e8b-8ad0-1096ec7fcc51" />




---------------------------------------------------------------------------------------------------------------------------------------------------------

### 6) Create buckets based on Average Price of reasonable size and find out how many resturants falls in each buckets										

SELECT

CASE 

     WHEN AVERAGE_COST_FOR_TWO = 0       THEN "N/A"
	 
     WHEN AVERAGE_COST_FOR_TWO > 0       AND  AVERAGE_COST_FOR_TWO <= 200    THEN "1) 0-200" 
	 
     WHEN AVERAGE_COST_FOR_TWO > 200     AND  AVERAGE_COST_FOR_TWO <= 500    THEN "2) 200-500" 
	 
     WHEN AVERAGE_COST_FOR_TWO > 500     AND  AVERAGE_COST_FOR_TWO <= 1000   THEN "3) 500-1K" 
	 
     WHEN AVERAGE_COST_FOR_TWO > 1000    AND  AVERAGE_COST_FOR_TWO <= 5000   THEN "4) 1K-5K" 
	 
     WHEN AVERAGE_COST_FOR_TWO > 5000    AND  AVERAGE_COST_FOR_TWO <= 10000  THEN "5) 5K-10K" 
	 
     WHEN AVERAGE_COST_FOR_TWO > 10000   AND  AVERAGE_COST_FOR_TWO <= 100000 THEN "6) 10K-1 LAKH" 
	 
     WHEN AVERAGE_COST_FOR_TWO > 100000  THEN "7) ABOVE 1 LAKH" 
	 
END AS AVERAGE_COST ,

count(restaurantid) as Restaurant_Count

FROM ZOMATO_TABLE

GROUP BY AVERAGE_COST

ORDER BY AVERAGE_COST ;

**RESULT :**

<img width="302" height="207" alt="image" src="https://github.com/user-attachments/assets/f19eba08-88a9-479f-acb7-1b7c8e92e3a2" />



----------------------------------------------------------------------------------------------------------------------------------------------------------

### 7) Percentage of Resturants based on "Has_Table_booking"		


select   Has_table_booking as Table_Booking ,

count(RestaurantID) as Restaunt_Count ,

concat(round((count(RestaurantID)  / sum(count(RestaurantID)) over() * 100 ),2)," %") as Resturant_Percent

from zomato_table

group by table_booking ;

**RESULT :**

<img width="388" height="82" alt="image" src="https://github.com/user-attachments/assets/2a2f48f0-c919-464e-b327-baa8446efb36" />


------------------------------------------------------------------------------------------------------------------------------------------------------

### 8) Percentage of Resturants based on "Has_Online_Delivery"

select   Has_Online_delivery as Online_Delivery ,

count(RestaurantID) as Restaunt_Count ,

concat(round((count(RestaurantID)  / sum(count(RestaurantID)) over() * 100 ),2)," %") as Restaurant_Percent

from zomato_table

group by online_delivery ;

**RESULT :**

<img width="396" height="81" alt="image" src="https://github.com/user-attachments/assets/6a340008-7abe-4010-bff6-a812ccd3116d" />



------------------------------------------------------------------------------------------------------------------------------------------------------



