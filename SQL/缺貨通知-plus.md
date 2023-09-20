[[SQL]]

select * from StoreProd 
where not barcode in (Select barcode from Product)

Insert into StoreProd(Barcode, State)
Select Barcode, '0' as State from Product 
where not barcode in (Select barcode from StoreProd)