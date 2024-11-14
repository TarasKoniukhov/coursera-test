# coursera-test

-- Dynamic Formatting (K, M, B) 
VAR _Pnl = SELECTEDMEASURE()

VAR _Result = SWITCH( TRUE,
                        ABS(_Pnl)<1000,"0;(0)",
                        ABS(_Pnl)<1000000,"#,.0K;(#,.0K)",                        
                        ABS(_Pnl)<1000000000,"#,,.0M;(#,,.0M)",
                         ABS(_Pnl)>=1000000000,"#,,,.0B;(#,,,.0B)")                       
RETURN _Result
