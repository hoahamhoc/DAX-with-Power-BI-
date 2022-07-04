# DAX-with-Power-BI-
DAX from basic to advanced
DAX from basic to advanced

                          CASE 1: Số lượng ProductName có Doanh Thu lớn hơn Doanh Thu trung bình của 1 Product Name              

VAR DS_ProductName = 
    SUMMARIZE(Sales,'Product'[Product Name])
VAR ADD_DT = 
    ADDCOLUMNS(
        DS_ProductName,
        "@DT",[Doanh Thu]
    )
VAR DTTB = 
    AVERAGEX(ADD_DT,[@DT])
VAR Filter_DT = 
    FILTER(
        ADD_DT,
        [@DT] > DTTB
    )
Return
    COUNTROWS(Filter_DT)
    

                           CASE 2: Top 5 Product Name có Doanh Thu lớn nhất chiếm bao nhiêu % Tổng Doanh Thu
                           
 VAR PN=
    SUMMARIZE('Product','Product'[Product Name])
VAR ADD_DOANHTHU=
    ADDCOLUMNS(PN,"@DT",[Doanh Thu])
Var TOP_5= 
    SUMX(TOPN(5,ADD_DOANHTHU,[@DT],DESC),[@DT])
RETURN
DIVIDE(TOP_5,[Doanh Thu])
    
    
                            
                           CASE 3: Top 5 Product Name có Doanh Thu nhỏ nhất chiếm bao nhiêu % Tổng Doanh Thu
                           
 VAR PN=
    SUMMARIZE('Product','Product'[Product Name])
VAR ADD_DOANHTHU=
    ADDCOLUMNS(PN,"@DT",[Doanh Thu])
Var TOP_5= 
    SUMX(TOPN(5,ADD_DOANHTHU,[@DT],ASC),[@DT])
RETURN
DIVIDE(TOP_5,[Doanh Thu])

    
                           CASE 4: Có bao nhiêu Product không được mua qua các ngày, tháng, năm hoặc các product không có doanh thu
                           
                           
 VAR ALL_DSSP_PR=
SUMMARIZE('Product','Product'[Product Name])
VAR ALL_DSSP_SALES=
SUMMARIZE(Sales,'Product'[Product Name])
VAR EXCEPT_=
EXCEPT(ALL_DSSP_PR,ALL_DSSP_SALES)
VAR KQ_COUNT_SP=
COUNTROWS(EXCEPT_)
VAR MIN_DATE_SALE=
CALCULATE(MIN(Sales[Order Date]),ALL())
VAR MAX_DATE_SALES=
CALCULATE(MAX(Sales[Order Date]),ALL())
VAR CURRENT_DATE=
MAX('Date'[Date])
return
IF(AND(
    CURRENT_DATE>=MIN_DATE_SALE,
    CURRENT_DATE<=MAX_DATE_SALES),KQ_COUNT_SP)

                              CASE 5:Running Total và cách giới hạn dữ liệu theo dòng thời gian thực
                              
                             

var maxsaledate = CALCULATE(MAX(Sales[Order Date]),ALL())
var minsaledate = CALCULATE(min(Sales[Order Date]),ALL())
var currentdate = max('Date'[Date])
var runningtotal = 
    if(
        currentdate<=maxsaledate && 
            currentdate>= minsaledate,
        CALCULATE([Doanh Thu], 
            FILTER( 
                ALL('Date'), 
                [Date]<=max('Date'[Date])
            )
        )
    )
Return
SWITCH(
    true(),
    ISFILTERED('Date'[Date]) || ISFILTERED('Date'[Calendar Year]) || ISFILTERED('Date'[Month]), runningtotal,
    [Doanh Thu]
)
                           
                  CASE 6: Tính doanh thu của các sản phẩm trong năm đầu tiên bán, sau đó qua các năm coi doanh thu của các sản phẩm đó như thế nào
                  
  VAR MIN_SALES_YEAR=
   YEAR(
       CALCULATE(MIN(Sales[Order Date]),
       ALL()
            )
              )
VAR PRODUCT_FIRSTSALEDATE=
    CALCULATETABLE(
        VALUES(Sales[ProductKey]),
        ALL(),
     YEAR(Sales[Order Date])= MIN_SALES_YEAR)
VAR FAKE_RLATIONSHIP=
    TREATAS(PRODUCT_FIRSTSALEDATE,Sales[ProductKey])
    //TU COT NAO DEN COT NAO (TRETAST)
RETURN
 CALCULATE([Doanh Thu],
 KEEPFILTERS(FAKE_RLATIONSHIP))
 
 
 
                                   CASE 7: Pareto Analysis (Brand). Xem các brand nào đóng góp đến 80% Doanh thu của công ty
    
    
    VAR DT
     [Doanh Thu]
VAR ALLSALES = 
    CALCULATE([Doanh Thu],ALLSELECTED(Sales))     
RETURN
DIVIDE(
    SUMX( 
        FILTER( SUMMARIZE(ALLSELECTED(Sales),'Product'[Brand],
            "DOANHTHU",[Doanh Thu]),
            [DOANHTHU]>=DT),[DOANHTHU]),
 ALLSALES,0)
                     
                     
                     
                     
                              CASE 8:RANKING WITH HIERARCHY (Brand -> Category -> Subcategory)
 VAR RANK_BRAND=
  VAR DT_BRAND=
   ADDCOLUMNS(
       CALCULATETABLE(
           VALUES('Product'[Brand]),
           ALLSELECTED()),
           "@DT",[Doanh Thu])
    
   VAR ADD_RANK=
       ADDCOLUMNS(
           DT_BRAND,"@RANK",
           VAR CURRENT_DT= [@DT]
           RETURN
           RANKX(DT_BRAND,
              [@DT],
                 CURRENT_DT,
                   DESC,
                      Dense)
                            )
VAR FILTER_BRAND=
    FILTER(ADD_RANK,
            [Brand] IN VALUES('Product'[Brand]))
RETURN
    SUMX(FILTER_BRAND,[@RANK])
    // ......................................................

VAR RANK_CATEGORY=
     VAR DT_CATEGORY=
          ADDCOLUMNS(
              CALCULATETABLE(VALUES('Product'[Category]),
              ALLSELECTED(),
              VALUES('Product'[Brand])),
              "@DT",
              [Doanh Thu])
    VAR ADD_RANK=
            ADDCOLUMNS( 
                DT_CATEGORY,
                "@RANK",
                      VAR CURRENT_DT=[@DT]
                   RETURN
            RANKX(DT_CATEGORY,[@DT],CURRENT_DT,DESC,Dense))
VAR FILTER_CATEGORY=
      FILTER(ADD_RANK,
              [Category] IN VALUES('Product'[Category]))
    RETURN
    SUMX(FILTER_CATEGORY,[@RANK])
    // .............................................................
VAR RANK_SUBCATEGORY=
    VAR DT_SUBCATEGORY=
        ADDCOLUMNS(
            CALCULATETABLE(
                  VALUES('Product'[Subcategory]),
                  ALLSELECTED(),
                  VALUES('Product'[Brand]),
                  VALUES('Product'[Category])),
                "@DT",
                 [Doanh Thu])
    VAR ADD_RANK=
            ADDCOLUMNS(
                    DT_SUBCATEGORY,
                    "RANK",
                    VAR CURRENT_DT= [@DT]
                    RETURN
                    RANKX(DT_SUBCATEGORY,[@DT],CURRENT_DT,DESC,Dense))
VAR FILTER_SUB=
     FILTER(ADD_RANK,
          [Subcategory] IN VALUES('Product'[Subcategory]))
        RETURN 
            SUMX(FILTER_SUB,[RANK])
RETURN 
 SWITCH(TRUE(),
     ISINSCOPE('Product'[Subcategory]),RANK_SUBCATEGORY,
     ISINSCOPE('Product'[Category]),RANK_CATEGORY,
     ISINSCOPE('Product'[Brand]),RANK_BRAND,"HOA"
  )
  
  
                                                          CASE 9: NEW CUSTOMER         
                                                          
VAR DS_KH =
    CALCULATETABLE(
        VALUES(Sales[CustomerKey]),
        ALLSELECTED()
    )
VAR ADD_MinDate = 
    ADDCOLUMNS(
        DS_KH,
        "@MinDate",
        CALCULATE(
            MIN(Sales[Order Date]),
            ALLSELECTED('Date')
        )
    )
VAR Filter_Date =
    FILTER(
        ADD_MinDate,
        [@MinDate] in VALUES('Date'[Date])
    )
Return
    COUNTROWS(Filter_Date)
    
                                                      CASE 10: Returning Customer
                                                      
   VAR CURRENT_=
          VALUES(Sales[CustomerKey]) 
 VAR CURRENT_DATE=
         MIN(Sales[Order Date])
 VAR PASTCUSTOMER= 
        CALCULATETABLE(
            VALUES(Sales[CustomerKey]),
        ALL('Date'[Date]),
        'Date'[Date]<CURRENT_DATE)
 VAR NEWCUSTOMER_= 
        INTERSECT(CURRENT_,PASTCUSTOMER)
 RETURN
        COUNTROWS(NEWCUSTOMER_)
                                                       
                                                       
                                                         CASE 11: Loyal Customer
                                                         
   Loyal Customer = 
VAR DSKH_WithFirstDate = 
    ADDCOLUMNS(
        CALCULATETABLE(
            VALUES(Customer[CustomerKey]),
            ALLSELECTED()
        ),
        "@MinDate",
        CALCULATE(
            MIN(Sales[Order Date]),
            ALL('Date')
        )    
    )
VAR AddCountrows = 
    ADDCOLUMNS(
        DSKH_WithFirstDate,
        "@CountRows",
        CALCULATE(
            DISTINCTCOUNT(Sales[Order Date]),
          	All('Date'),
          	Sales[Order Date] <= MAX('Date'[Date])
        ),
        "@DiffDate",
        DATEDIFF([@MinDate],MAX('Date'[Date]),DAY)
    )
REturn
    COUNTROWS(
        FILTER(
            AddCountrows,
            [@CountRows] = [@DiffDate] && NOT(ISBLANK([@MinDate]))&&NOT(ISBLANK([@CountRows]))
        )
    )
                                                       

                  
  
   
