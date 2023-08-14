# SQL-Data-Cleaning
------This is a data cleaning project for Nashville Housing using SQL
----we will be working on Nashville Housing dataset

SELECT *
FROM Portfolioproject.dbo.NashvilleHousing 

--------------------------------------------------------------------------

--Standardize Date Format
---The SaleDate was converted from DateTime format to Date format and was given a new name which is SalesDateConverted

SELECT SalesDateConverted, CONVERT(Date,SaleDate)
FROM Portfolioproject.dbo.NashvilleHousing 


UPDATE Portfolioproject.dbo.NashvilleHousing
SET SaleDate = CONVERT(Date,SaleDate)

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
ADD SalesDateConverted Date

---the SaleDateConverted was updated on the server
UPDATE Portfolioproject.dbo.NashvilleHousing
SET SalesDateConverted = CONVERT(Date,SaleDate)

------------------------------------------------

--Populate Property Address Data
---some of the PropertyAddress were populated with NuLL values
SELECT *
FROM Portfolioproject.dbo.NashvilleHousing
WHERE PropertyAddress is NULL
ORDER BY ParcelID

---The PropertyAddress were populated using the ParcelID since some of the PropertyAddreess were recorded twice and this was done by using a self join.The table was
---joined to itself were the ParcelId are the same and the UniqueID arent the same

SELECT  a.PropertyAddress , a.ParcelID, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress )
FROM Portfolioproject.dbo.NashvilleHousing a
JOIN Portfolioproject.dbo.NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.UniqueID <> b.UniqueID
WHERE  a.PropertyAddress is NULL

-----PropertyAddress was updated on the sql server

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress )
FROM Portfolioproject.dbo.NashvilleHousing a
JOIN Portfolioproject.dbo.NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.UniqueID <> b.UniqueID
WHERE  a.PropertyAddress is NULL


---------------------------------------------------------------------
---Breaking out Address into Individual Columns (Address, City, State)
-----The Address and State were obtained from the PropertyAddress column using the delimeter(,) in this instance.

SELECT
   SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) as Address
  ,SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) as Address
FROM Portfolioproject.dbo.NashvilleHousing

---The new column "PropertySplitAdddress" was gotten from PropertyAddress column which has been added to the server and updated

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
ADD PropertySplitAddress Nvarchar(255)

UPDATE Portfolioproject.dbo.NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)

---The new column "PropertySplitCity" was gotten from PropertyAddress column which has been added to the server and updated

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
ADD PropertySplitCity Nvarchar(255)

UPDATE Portfolioproject.dbo.NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))



SELECT OwnerAddress
FROM Portfolioproject.dbo.NashvilleHousing

---Parsenmae was used for the delimited value
SELECT 
  PARSENAME(REPLACE(OwnerAddress,',','.') ,3) 
,PARSENAME(REPLACE(OwnerAddress,',','.') ,2) 
,PARSENAME(REPLACE(OwnerAddress,',','.') ,1)
FROM Portfolioproject.dbo.NashvilleHousing


---The new column "OwnerSplitAddress" was gotten from "OwnerAddress" column which has been added to the server and updated

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
ADD OwnerSplitAddress Nvarchar(255)

UPDATE Portfolioproject.dbo.NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress,',','.') ,3)


---The new column "OwnerSplitCity" was gotten from "OwnerAddress" column which has been added to the server and updated

ALTER TABLE  Portfolioproject.dbo.NashvilleHousing
ADD OwnerSplitCity Nvarchar(255)

UPDATE Portfolioproject.dbo.NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress,',','.') ,2)


---The new column "OwnerSplitState" was gotten from "OwnerAddress" column which has been added to the server and updated

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
ADD OwnerSplitState Nvarchar(255)

UPDATE Portfolioproject.dbo.NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress,',','.') ,1)

-------------------------------------------------------------------
 --Change Y and N to YES and NO in 'Sold as Vacant' field


SELECT DISTINCT(SoldAsVacant), COUNT(SoldAsVacant)
FROM Portfolioproject.dbo.NashvilleHousing
GROUP BY SoldAsVacant

----The values of SoldAsVacant that were previously "Y" and "N" were changed to "Yes" and "No," respectively, whereas SoldAsVacant is used to denote values that are empty.
SELECT SoldAsVacant, 
CASE when SoldAsVacant = 'Y' THEN 'Yes'
     when SoldAsVacant = 'N' THEN 'No'
	 Else SoldAsVacant
	 End
FROM Portfolioproject.dbo.NashvilleHousing

----the SoldAsVacant column was updated on the sql server

UPDATE Portfolioproject.dbo.NashvilleHousing
SET SoldAsVacant = CASE when SoldAsVacant = 'Y' THEN 'Yes'
     when SoldAsVacant = 'N' THEN 'No'
	 Else SoldAsVacant
	 End

----------------------------------------------------------------------

--Remove Duplicates

----here, duplicates and unused columns are removed by using CTE . The row_number enables us identify the duplicate values 

WITH RowNumCTE as(
SELECT *, ROW_NUMBER() OVER(
PARTITION BY ParcelID,
PropertyAddress,
SalePrice,SaleDate,LegalReference
ORDER BY UniqueID) row_num
FROM Portfolioproject.dbo.NashvilleHousing
)

SELECT*
FROM RowNumCTE 
WHERE row_num > 1

-----the duplicate values are deleted

WITH RowNumCTE as(
SELECT *, ROW_NUMBER() OVER(
PARTITION BY ParcelID,
PropertyAddress,
SalePrice,SaleDate,LegalReference
ORDER BY UniqueID) row_num
FROM Portfolioproject.dbo.NashvilleHousing
) 

DELETE
FROM RowNumCTE 
WHERE row_num > 1



---DELETE UNUSED COLUMNS

----the unused columns are deleted from the server

SELECT *
FROM Portfolioproject.dbo.NashvilleHousing

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
DROP COLUMN PropertyAddress, OwnerAddress, TaxDistrict

ALTER TABLE Portfolioproject.dbo.NashvilleHousing
DROP COLUMN SaleDate
