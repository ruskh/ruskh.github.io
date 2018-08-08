---
layout: post
title: "Using Sql Server File Tables and Json Functions to work with GSTR2A Files "
date: 2018-08-08
---

GSTR2A.json contains details of invoices uploaded by Suppliers. One file exists per month. The data contained in these files keeps changing as and when suppliers file their returns, so these files need to be redownloaded. Storing these files in a  MS Sql Server File Table, makes it easy to :

1. Consider only the latest file, by File Creation Date and ignore older duplicates.
2. Store files for multiple periods and Dealers in the same folder.
3. Give any names to the files, without affecting the results.

		 SELECT name as Filename , creation_time,
      		 Gstr2A.OurGST,Gstr2A.GSTMonth,Supplier.GSTIN,Invoice.Inv_Date,Invoice.Inv_Num,      		 
		 Invoice.Inv_Total,Item.Taxable_Value,Item.Tax_Rate,Item.CGST,Item.SGST,Item.IGST,Item.Cess,
                 Invoice.Inv_Type,Invoice.Point_of_Supply,Invoice.RCM,
                 Item.Num,Invoice.chksum,Supplier.cfs,Supplier.cname from dbo.GSTJSON 

			CROSS APPLY openjson(cast((cast(file_stream as xml)) as nvarchar(max))) 
			WITH (OurGst nvarchar(15) '$.gstin',  GSTMonth nvarchar(10) '$.fp', 
		        b2b nvarchar(max) AS JSON) As Gstr2A

	                CROSS APPLY OPENJSON(Gstr2A.b2b)
	                WITH (GSTIN nvarchar(15) '$.ctin' ,cfs nvarchar(5),cname nvarchar(50),
	      	        inv nvarchar(max) AS JSON) AS Supplier
		   

		        CROSS APPLY OPENJSON(Supplier.inv)
		        WITH( itms nvarchar(max) AS JSON,
		        Inv_Total float '$.val',Inv_Type nvarchar(5) '$.inv_typ',
		        Point_of_Supply smallint '$.pos' , Inv_Date nvarchar(50)'$.idt',
		        RCM nvarchar(5)'$.rchrg' ,Inv_Num nvarchar(20)'$.inum' ,
		        chksum nvarchar(100)) AS Invoice

		        CROSS APPLY OPENJSON(Invoice.itms)
		        WITH (num int ,Taxable_Value float '$.itm_det.txval',
		        Tax_Rate float '$.itm_det.rt',Sgst float '$.itm_det.samt',
		        Cgst float '$.itm_det.camt',Igst float '$.itm_det.iamt',Cess float '$.itm_det.csamt') As Item


