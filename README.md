# Cleaning-NFL-Data
--Chapter 8 solutions from class
--View the first 1000 rows of the newly imported table

SELECT TOP 1000 *
FROM Career_Stats_Receiving;

--Are there any players in the receiving table that are not in the basics table

SELECT DISTINCT [Player ID]
FROM Career_Stats_Receiving
WHERE [Player ID] NOT IN
(SELECT DISTINCT [Player ID] FROM Basic_Stats);

--Appears that all players in the Receiving table are also in the Basic stats table

--Let's create a copy of our Receiving table to do our analysis
SELECT *
INTO Career_Stats_Receiving_Analysis
FROM Career_Stats_Receiving;

--We will use this copy to for Analysis so that we don't corrput the original data
--Look at the data in the new table that was made
SELECT TOP 1000 *
FROM Career_Stats_Receiving_Analysis;

--Are there any empty values in player ID or name?
SELECT [player ID] 
FROM Career_Stats_Receiving_Analysis
WHERE [player ID] = ' ';

--They query returned no rows with playerID having empty data. How about Name? 

SELECT PlayerName 
FROM Career_Stats_Receiving_Analysis
WHERE PlayerName = ' ';

--All rows have data for player name as well! 
--Explore the Games Played column

SELECT [Games Played], Count(*) AS [# of games played]
FROM Career_Stats_Receiving_Analysis
GROUP BY [Games Played]
ORDER BY [Games Played];

-- a number of records have zero games played
-- the range of games played is 0 to 17
--are the 0 games played and 17 games played values reasonable?

--Explore 0 games played

SELECT *
FROM Career_Stats_Receiving_Analysis
WHERE [Games Played]= '0';
-- 628 rows but they dont have other stats either so that is a good sign
SELECT *
FROM Career_Stats_Receiving_Analysis
WHERE PlayerName LIKE 'Basnight, M%' OR PlayerName LIKE 'Taylor, Tr%';

-- explore two specific players to see if they had activity in other seasons 
-- One guy played two seasons, the other played 11. 

--explore 17 games played
SELECT *
FROM Career_Stats_Receiving_Analysis
WHERE [Games Played]='17';

--Interesting that each player has atleast one TD but no receiving yards of receptions
--Explore the 1930 season

SELECT *
FROM Career_Stats_Receiving_Analysis
WHERE Season = '1930'
Order BY Team;

-- Games played range from 1 to 17. Did some research and 17 is a valid number. Only two teams played 17 games in 
-- a 11 team league. 

SELECT DISTINCT Team 
FROM Career_Stats_Receiving_Analysis
WHERE Season = '1930';

--Missing team: Providence Steam Roller 10/11
--Explore receptions column

SELECT receptions, COUNT(*) AS [# of records]
FROM Career_Stats_Receiving_Analysis
GROUP BY Receptions
ORDER BY [# of records] DESC;

-- a number of records (4404) have '--' as values. this data value comes from the original data set
-- It appears these values are not known, thus, I would like to change these values to NULL
-- other numbers apppear reasonable. I will change the data to numeric data type. Because receptions
-- will always be whole number, I will use integer data type
  
 ALTER TABLE [dbo].[Career_Stats_Receiving_Analysis]
 ADD [Clean Receptions] int NULL; 

 --The NULL statement used does two things 1) allows NULL's in the new column
 -- 2)sets all the values in each row to NULL

 SELECT *
 FROM [dbo].[Career_Stats_Receiving_Analysis];

 Select [clean receptions]
 FROM [dbo].[Career_Stats_Receiving_Analysis];

 --Use CTRL + SHIFT + R to refresh tables 

 -- Update table to clean reception data and place in new column
-- first query is for existing numeric values

UPDATE Career_Stats_Receiving_Analysis
SET [Clean Receptions] = CAST(Receptions AS INT)
WHERE Receptions != '--';
-- Took the values in the Receptions column and transfered them over to the new column as INT Data type

SELECT receptions, COUNT(*) AS [# of records]
FROM Career_Stats_Receiving_Analysis
GROUP BY Receptions
ORDER BY Receptions;

SELECT [Clean Receptions], COUNT(*) '# of records'
FROM Career_Stats_Receiving_Analysis
GROUP BY [Clean Receptions]
ORDER BY [Clean Receptions] DESC;

-- there is no need for the following query because all rows in "Clean Receptions"
-- were assigned NULL value when the column was added 

UPDATE Career_Stats_Receiving_Analysis
SET [Clean Receptions] = NULL
WHERE Receptions = '--';

--Let's explore the Longest Reception column

SELECT [Longest Reception], Count(*) AS [# of records]
FROM [dbo].[Career_Stats_Receiving_Analysis]
GROUP BY [Longest Reception]
ORDER BY [Longest Reception] desc; 

--A number of records have the letter 't' after the number of yards

SELECT [Longest Reception], Count(*) AS [# of records]
FROM [dbo].[Career_Stats_Receiving_Analysis]
GROUP BY [Longest Reception]
ORDER BY [# of records] desc; 

--A number of records have '--' as the longest reception. I will explore more

SELECT *
from [dbo].[Career_Stats_Receiving_Analysis]
WHERE [Longest Reception] = '--';

--	I observe a problem in the data. While many records with '--' in the longest reception
--	column also have '--' in the other columns (e.g., receptions, receiving yards, etc.), some
--	columns have values in these other columns. For example, Stu Voight played 14 games in 1974
--	with 32 receptions and 268 receiving yards; however, the longest reception column has '--'
--	I could not find a means of deriving longest reception from the other columns. For now,
--	I am ignoring this anonomly for now. 

--	Because 8627 represents about 48% of the data, I may not be
--	able to use the 'longest reception' column in statistical analysis. 

--	I decided to replace all '--' values with NULL


--	Several values have the letter 'T' at the end of the character string. I could not find any
--	information on the NFL website about the 'T'. I am guessing the 'T' indicates a touchdown.
--	I will handle this in the clean data set by adding two columns. One column to hold the 
--	numeric values for the longest reception and one to hold a "flag" for whether the
--	player scored a touchdown with their longest reception

ALTER TABLE [dbo].[Career_Stats_Receiving_Analysis]
ADD [Clean Longest Reception] int NULL;

-- created column and added to the table, included data type and that it accepts NULL values

ALTER TABLE [dbo].[Career_Stats_Receiving_Analysis]
ADD [Clean Longest Reception was TD] bit NULL;

--Similar to last operation

UPDATE [dbo].[Career_Stats_Receiving_Analysis]
SET [Clean Longest Reception] = Cast([Longest Reception] AS int)
WHERE [Longest Reception] != '--' AND [Longest Reception] NOT LIKE '%T';

-- Adds values to the column that I made but excludes '--' and numbers with T's

SELECT	[Longest Reception], LEN([Longest Reception]) AS 'String Length', 
		LEFT([Longest Reception], (LEN([Longest Reception])-1)) AS 'Clean Longest Reception'
FROM Career_Stats_Receiving_Analysis
WHERE [Longest Reception] LIKE '%T'
ORDER BY 'String Length' DESC;

--	Parse the longest reception yards and store in clean longest reception
UPDATE [dbo].[Career_Stats_Receiving_Analysis]
SET [Clean Longest Reception] =
		CAST(LEFT([Longest Reception], (LEN([Longest Reception]) -1)) AS int ) 
WHERE [Longest Reception] LIKE '%T';

--	Set [Clean Longest Reception as TD] = 1 when longest reception is a TD

UPDATE Career_Stats_Receiving_Analysis
SET [Clean Longest Reception was TD]=1
WHERE [Longest Reception] LIKE '%T';

--	Set [Clean Longest Reception as TD] = 0 when longest reception is NOT a TD


UPDATE Career_Stats_Receiving_Analysis
SET [Clean Longest Reception was TD]=0
WHERE [Longest Reception] != '--' AND [Longest Reception] NOT LIKE '%T';


SELECT [Longest Reception], [Clean Longest Reception], [Clean Longest Reception was TD]
FROM Career_Stats_Receiving_Analysis
WHERE [Longest Reception] LIKE '%T' OR [Longest Reception] != '--';

DROP TABLE Career_Stats_Receiving_Analysis;
