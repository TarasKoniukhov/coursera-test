# coursera-test

-- Dynamic Formatting (K, M, B) 
VAR _Pnl = SELECTEDMEASURE()

VAR _Result = SWITCH( TRUE,
                        ABS(_Pnl)<1000,"0;(0)",
                        ABS(_Pnl)<1000000,"#,.0K;(#,.0K)",                        
                        ABS(_Pnl)<1000000000,"#,,.0M;(#,,.0M)",
                         ABS(_Pnl)>=1000000000,"#,,,.0B;(#,,,.0B)")                       
RETURN _Result



--/////////////////////////SVG Line Chart /////////////////////////



//https://kerrykolosko.com/portfolio/gradient-sparklines/
// line and area colour - use %23 instead of # for Firefox compatibility (Eldersveld)
//VAR Defs = 
//        "<defs>
//            <linearGradient id='grad' x1='0' y1='25' x2='0' y2='50' gradientUnits='userSpaceOnUse'>
//                <stop stop-color='green' offset='0' />
//                <stop stop-color='red' offset='1' />
//            </linearGradient>
//        </defs>"
// "Date" field used in this example along the X axis
VAR XMinDate =
    MINX( ALL( POBackTest[Date] ), POBackTest[Date] )
VAR XMaxDate =
    MAXX( ALL( POBackTest[Date] ), POBackTest[Date] )

// Obtain overall min and overall max measure values when evaluated for each date
VAR YMinValue =
    MINX(
        VALUES( POBackTest[Date] ),
        CALCULATE( [Portfolio Value, USD] )
    )
VAR YMaxValue =
    MAXX(
        VALUES( POBackTest[Date] ),
        CALCULATE( [Portfolio Value, USD] )
    )

// Build table of X & Y coordinates and fit to 50 x 150 viewbox
VAR SparklineTable =
    ADDCOLUMNS(
        SUMMARIZE( POBackTest, POBackTest[Date] ),
        "X",
            INT(
                150
                    * DIVIDE( POBackTest[Date] - XMinDate, XMaxDate - XMinDate )
            ),
        "Y",
            INT(
                50
                    * DIVIDE(
                        [Portfolio Value, USD] - YMinValue,
                        YMaxValue - YMinValue
                    )
            )
    )

// Concatenate X & Y coordinates to build the sparkline
VAR Lines =
    CONCATENATEX(
        SparklineTable,
        [X] & "," & 50 - [Y],
        " ",
        POBackTest[Date]
    )

VAR __SortValue =
    FORMAT( AVERAGE(POMatrixStrategies[PnLRank]), REPT( 0, 15 ) )

// Add to SVG, and verify Data Category is set to Image URL for this measure
VAR SVGImageURL =
    IF(
        HASONEVALUE( POBackTest[version] ),
        "data:image/svg+xml;utf8,"
            & "<svg xmlns='http://www.w3.org/2000/svg' 
            x='0px' y='0px' viewBox='0 0 150 50'
                    sort='"
           & __SortValue
            & "'
 
            >"
            & // Defs &
            "<polyline fill='none' stroke='#3454BE' 
                stroke-width='1' points='"
                & Lines
            & "'/></svg>",
        BLANK( )
    )
RETURN
    SVGImageURL

---////////////////////////SVG Text ////////////////////////////////////...

// Bonus: add a sort value that we can use for the table to sort

VAR __DataTable =
  //ADDCOLUMNS(
  //          SUMMARIZE(
  //              DimMetals,
  //              DimMetals[metal_id] 
  //          ),
  //          "@Val", SUM(POMatrixMetals[Cumulative P&L]),
  //          "@Color",IF( SUM(POMatrixMetals[Cumulative P&L]) > 0, "#38a169", "#e53e3e" )
  //  )

    SUMMARIZECOLUMNS(
			DimMetals[metal_id],
        "@Val", SUM(POMatrixMetalsPQ[Cumulative P&L]),
        "@Color",IF( SUM(POMatrixMetalsPQ[Cumulative P&L]) > 0, "#38a169", IF( SUM(POMatrixMetalsPQ[Cumulative P&L]) < 0,"#e53e3e" ))
    )
    
    
VAR __Txt =
    CONCATENATEX(
        __DataTable,
        "<tspan dy='1em' x='2' 
        font-family='Segoe UI Semibold, wf_segoe-ui_normal, helvetica, arial, sans-serif'
        font-size='0.7em' fill='" & [@Color] & "'>"
            & DimMetals[metal_id]
            & ": "
            & FORMAT( [@Val], "#,0;(#,0)" )
            & " </tspan>",
        "",
        [@Val], DESC
    )
VAR __SortValue =
    FORMAT( AVERAGE(POMatrixStrategies[PnLRank]), REPT( 0, 15 ) )
VAR __xValueScale = SUM(POMatrixMetalsPQ[Cumulative P&L])
VAR __svgBase =
    "data:image/svg+xml;utf8,
    <svg 
        xmlns='http://www.w3.org/2000/svg'
        width='100%' height='100%'
  
        sort='"
        & __SortValue
        & "'
        scale='"
        & __xValueScale
        & "'
    >    

   USHWF-SCIEH-AVIOO-QIBCJ-NDFAS
           <text>"
                &  __Txt  &
          "</text> 
     </svg>"
RETURN
    __svgBase
