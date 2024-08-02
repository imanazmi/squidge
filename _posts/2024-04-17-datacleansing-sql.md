---
layout: post
title:  Data Cleansing in SQL
categories: [SQL,Code, Project]
---

Project description: I used SQL to clean data using Nashville Housing data 


References: [AlexTheAnalyst](https://www.youtube.com/watch?v=qfyynHBFOsM&list=PLUaB-1hjhk8H48Pj32z4GZgGWyylqv85f)

Below is the raw snippet of code including the comments.

1) Select all to view overall data.
```sql	

select *
from PortfolioProject..NashvilleHousing
```


2) Standardise date format
```sql
select saleDateConverted, convert (date, saledate) as newSaleDate
from PortfolioProject..NashvilleHousing

update NashvilleHousing
set SaleDate = CONVERT(date, saledate)

alter table nashvillehousing
add saleDateConverted date;

update NashvilleHousing
set saleDateConverted = CONVERT(date, saledate)

select *
from PortfolioProject..NashvilleHousing
```


3) Populate property address
```sql
select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, isnull(a.PropertyAddress, b.PropertyAddress)
from PortfolioProject.dbo.NashvilleHousing a
join PortfolioProject.dbo.NashvilleHousing b
on a.ParcelID = b.ParcelID
and a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

update a
set PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
from PortfolioProject.dbo.NashvilleHousing a
join PortfolioProject.dbo.NashvilleHousing b
on a.ParcelID = b.ParcelID
and a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null
```


4) Break out the address into individual columns  
There are two ways to do the split

a) Long way
```sql
select PropertyAddress
from PortfolioProject..NashvilleHousing
--where PropertyAddress is null
--order by ParcelID

select 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', propertyaddress)-1) as address,
SUBSTRING(propertyaddress, CHARINDEX(',', propertyaddress) + 1 , LEN(PropertyAddress)) as address
from PortfolioProject.dbo.NashvilleHousing

alter table nashvillehousing
add propertySplitAddress nvarchar(255);

update NashvilleHousing
set propertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', propertyaddress)-1)

alter table nashvillehousing
add propertySplitCity nvarchar(255);

update NashvilleHousing
set propertySplitCity = SUBSTRING(propertyaddress, CHARINDEX(',', propertyaddress) + 1 , LEN(PropertyAddress))

select *
from PortfolioProject.dbo.NashvilleHousing
```


b) Short way
```sql
select OwnerAddress
from PortfolioProject..NashvilleHousing

select 
PARSENAME(replace(owneraddress, ',', '.'), 3),
PARSENAME(replace(owneraddress, ',', '.'), 2),
PARSENAME(replace(owneraddress, ',', '.'), 1)
from PortfolioProject..NashvilleHousing
```


i. Split parsename 3
```sql
alter table nashvillehousing
add ownerSplitAddress nvarchar(255);

update NashvilleHousing
set ownerSplitAddress = PARSENAME(replace(owneraddress, ',', '.'), 3)
```


ii. Split parsename 2
```sql
alter table nashvillehousing
add ownerSplitCity nvarchar(255);

update NashvilleHousing
set ownerSplitCity = PARSENAME(replace(owneraddress, ',', '.'), 2)
```


iii. Split parsename 1
```sql
alter table nashvillehousing
add ownerSplitState nvarchar(255);

update NashvilleHousing
set ownerSplitState = PARSENAME(replace(owneraddress, ',', '.'), 1)

select *
from PortfolioProject..NashvilleHousing
```


5) Change 'Y' and 'N' to 'Yes' and 'No' in SoldAsVacant field
```sql
select distinct(SoldAsVacant), count(soldasvacant)
from PortfolioProject..NashvilleHousing
group by SoldAsVacant
order by 2

select SoldAsVacant,
case	when SoldAsVacant = 'Y' then 'Yes'
		when SoldAsVacant = 'N' then 'No'
		else SoldAsVacant
		end
from PortfolioProject..NashvilleHousing
```


a) Use update to count the number of 'Yes' and 'No'
```sql
update NashvilleHousing
set SoldAsVacant = case when SoldAsVacant = 'Y' then 'Yes'
		when SoldAsVacant = 'N' then 'No'
		else SoldAsVacant
		end
```


5) Remove duplicates
```sql
with rowNumCTE as(
select *, 
	ROW_NUMBER() 
	over (
	partition by parcelID,
				 propertyAddress,
				 salePrice,
				 saleDate,
				 legalReference
				 ORDER BY
					UniqueID
					) row_num

from PortfolioProject..NashvilleHousing
--order by ParcelID
)
/*
delete
from rowNumCTE
where row_num > 1
*/

select *
from rowNumCTE
where row_num > 1
--order by PropertyAddress

select *
from PortfolioProject..NashvilleHousing
```

6) Delete unused columns
```sql
select *
from PortfolioProject..NashvilleHousing

alter table PortfolioProject..NashvilleHousing
drop column owneraddress, taxdistrict, propertyaddress

alter table PortfolioProject..NashvilleHousing
drop column saledate
```

