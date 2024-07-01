# Edmonton 311-Report

### Report Link : [Edmonton 311 Status Analysis Visualization.pdf](https://github.com/user-attachments/files/16042465/Edmonton.311.Status.Analysis.Visualization.pdf)

## Problem Statement

This report helps the City Manager keep track of the status of all contacts to the 311 responsive department, and better reposition the 311 team for prompt responses. It helps ascertain the number of complaints recieved by the team, the amount of complaints that are resolved and unresolved as well as time spent on resolving the complaints. Also helps determine the locations the complaints are from and the channels through different visuals.

### Steps followed 

- Step 1 : Load data into Power BI Desktop, dataset is a csv file.

- Step 2 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.

- Step 3 : Also since by default, profile will be opened only for 1000 rows so "column profiling based on entire dataset" was selected.

- Step 4 : It was observed that there were blank rows and columns with errors & empty values were present. Blank rows were removed and columns with empty or null string values were replaced with "Unspecified". The columns include "Status Detail", "Neighbourhood", "Community League", and "Ward". Null values in the "Date Closed" column were omitted as it represents tickets that have not been closed out.

- Step 5 : An index column ("TicketID") was added to attach an identity key to each ticket data (row).

-Step 6 :  A custom column was added ("Number of Days") to calculate the days difference between the "Date Created" and "Date Closed".

 Following M Query was written to find the Number of Days,
 
         Number of Days = Duration.Days([Date Closed]-[Date Created])+1 

- Step 7 : In the report view, under the view tab, theme was selected.

- Step 8 : Since the data contains various ratings, thus in order to represent ratings, a new visual was added using the three ellipses in the visualizations pane in report view.

- Step 9 : Visual filters (Slicers) were added for four (4) fields and one (1) parameter named "Year", "Month", "Day", "Status Detail" and "Range" respectively.

- Step 10 : Seven (7) card visuals,from calculated measures, were added to the canvas as mentioned below with written DAX expressions respectively,

  (a) Total Number of Tickets

                Number of Tickets = 
                var _Calc = COUNT('311_Explorer'[TicketID] )
                RETURN IF(ISBLANK(_Calc),0,_Calc)

  (b) Total Number of Open Tickets (i.e. Tickets without closed date)

        Number of Open Tickets = 
        VAR _Calc = CALCULATE('Calculate Measures'[Number of Tickets],'311_Explorer'[Request Status] = "Open") 
        RETURN IF(ISBLANK(_Calc),0,_Calc)
  
  (c) Total Number of Closed Tickets

             Number of Closed Tickets = 
             VAR _Calc = 
             CALCULATE('Calculate Measures'[Number of Tickets],'311_Explorer'[Request Status] = "Closed") 
             RETURN IF(ISBLANK(_Calc),0,_Calc)
  
  (d) Average Number of Tickets per Day

        Average Tickets per Day = AVERAGEX(VALUES('311_Explorer'[Date Created].[Day]),[Number of Tickets])
  
  (e) Average Number of Tickets per Month

        Average Tickets per Month = AVERAGEX(VALUES('311_Explorer'[Date Created].[Month]),[Number of Tickets])
  
  (f) Average Number of Tickets per Year

        Average Tickets per Year = AVERAGEX(VALUES('311_Explorer'[Date Created].[Year]),[Number of Tickets])

  (g) Average Number of Days to Close Ticket

        Average Number of Days = AVERAGE('311_Explorer'[Number of Days])
           
           In order to avoid the "total number of tickets" as well as 
           the "total number of open and closed tickets" from returning (Blank) values,
           ISBLANK function was used replace "(Blank)" with "0" in each of the DAX Queries.

![Cards Visuals](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/4660d626-e051-4926-bcf0-3bdad5779549)


- Step 11 : A bar chart was also added to the report design area representing the total number of tickets by service code. While creating this visual, the TOPN function was used in writing the DAX expression to get the top service code with the highest number of tickets with the aid of the range paramenter as a filter.

        Dynamic TOPN Ticket Count = 
                CALCULATE([Number of Tickets],
                KEEPFILTERS(
                    TOPN(
                            Range[Range Value],
                            ALLSELECTED('311_Explorer'[Service Code]),
                            [Number of Tickets],DESC
                        )
                    )
                )  
Also, to get a dynamic chart title to match with the selected range value a measure was created with the following written DAX expression;

        TOPN Title:Service Code Ticket Count = "Top " & 
        SELECTEDVALUE(Range[Range]," ") & " Ticket Count by Service Code"
        
- Step 12 : A scatter chart was added to the report representing the total number tickets, further dissected into total number of open and closed tickets respectively, in each neighbourhood.

- Step 13 : A pie chart was added to the report representing the percentage of total number tickets for each ticket source.

- Step 14 : A column chart was added to the report representing the total number tickets, with the ability to drill down, in each ward,then its community league then the neighbourhoods. An hierarchy was created to easily achieve this as seen below;

![ward hierarchy snapshot](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/e9d1bf27-db6f-441e-96cb-e96a57280786)

- Step 15 : In the report view, under the insert tab, using shapes option from elements group a rectangle was inserted, adjusted and positioned, and the report title and sutitle were inserted by creating each measures with the following DAX expressions;

        TITLE:
        Status Title = "Status Analysis For Year: " & SELECTEDVALUE('311_Explorer'[Date Created].[Year],"All") & ", 
        in Month: " & SELECTEDVALUE('311_Explorer'[Date Created].[Month],"All") & ", on Day: " & 
        SELECTEDVALUE('311_Explorer'[Date Created].[Day],"All")

        SUBTITLE:
        Status Subtitle = "In " & SELECTEDVALUE('311_Explorer'[Ward],"All Wards") & " Within " & 
        SELECTEDVALUE('311_Explorer'[Community League],"All Community Leagues") & " and in " & 
        SELECTEDVALUE('311_Explorer'[Neighbourhood],"All Neighourhoods") & ", For Service Code: " & 
        SELECTEDVALUE('311_Explorer'[Service Code],"All") & ", And Service Detail: " & 
        SELECTEDVALUE('311_Explorer'[Status Detail], "All") & ", 
        Via: " & SELECTEDVALUE('311_Explorer'[Ticket Source], "All Ticket Sources")

The DAX expressions were written to accommodate for selected values from the visuals and slicers.

- Step 16 : Calculated column was created in which tickets were grouped based on the time period of ticket closure (Early, Late, Very Late, Extremely Late, and Overdue).

for creating the new column following DAX expression was written;
       
         Period of Resolution Indicator = 
    VAR _Calc =
        IF('311_Explorer'[Number of Days] >= 1 && '311_Explorer'[Number of Days] <=31, "Early",
            IF('311_Explorer'[Number of Days] > 31 && '311_Explorer'[Number of Days] <= 180, "Late",
                IF('311_Explorer'[Number of Days] > 180 && '311_Explorer'[Number of Days] <= 365, "Very Late",
                    IF('311_Explorer'[Number of Days] > 365, "Extremely Late"
                        )
                    )
                )
            )
    RETURN IF(ISBLANK(_Calc),"Overdue",_Calc)
        
Snap of new calculated column,

![Period of resolution indicator snapshot](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/4106077c-e3b8-44b3-bcc6-8bfa0a8d5e52)

- Step 17 : A donut chart was added to the report representing the percentage of total number tickets for each resolution period indicator.

- Step 18 : A table was added to the report representing the date each ticket was created and the date closed, the difference  of both days, the period resolution indicator based on the difference, and the ward, community league, neighbourhood, service code and status detail.

![table visual](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/4af42257-f321-4f19-99e1-865571ce5bb1)


# Insights

A two-paged report, derived from Edmonton 311 five (5) years (2013 - 2017) data, was created on Power BI Desktop, then exported into pdf.

The following inferences can be drawn from the report;

### [1] Total Number of Tickets = 267,224

   Number of Open Tickets = 6,615
   
    2013: 23
    2014: 57
    2015: 64
    2016: 508
    2017: 5,963 

   Number of Closed Tickets = 260,609
           
### [2] Neighbourhood

Downtown has the most number of tickets at 5,467 (5,311 of these are closed and the remaining 156 tickets are still open), followed by Strathcona with 3,911 total tickets (3,713 closed and 198 open tickets). Then Oliver with the total tickets of 3,900 (3,802 closed and 98 open), Alberta Avenue follows with a total of 3,192 tickets (3,021 closed and 171 open tickets), and the fifth top neighbourhood with 2,816 tickets (2,711 closed and 105 open tickets) is Westmount.

    It is noticed that Strathcona has the most open tickets at 198 followed by Alberta Avenue at 171 then 
    Downtown at 156 despite having the most number of total tickets.

 ### [4] Some other insights
 
 ### Ticket Source
 
    a) Telephone Call - 216,025 (80.84%)
    b) MobileApp - 13,841 (5.18%)
    c) Internet Mail - 11,150 (4.17%)
    d) Web Form - 10,035 (3.76%) 
    e) Mobile App Auto - 8,036 (3.01%)
    f) Posse Auto - 4,274 (1.6%)
    g) Mobile App Agent - 3,863 (1.45%)

         thus, complaints are recieved the most via Telephone Calls to 311.
 
 ### Ticket Resolution Period Indicator
 
 1.1) 80.29% (214,547) tickets were resolved 'Early' i.e. between 1 to 31 days.
 
 1.2) 13.94% (37,253) tickets were resolved 'Late' i.e. between 32 to 180 days.
 
 1.3) 2.48% (6,6115) tickets have not been resolved i.e. 'Overdue'.
 
 1.4) 2.19% (5,860) tickets were resolved 'Very Late' i.e. between 181 to 365 days.

 1.5) 1.1% (2,949) tickets were resolved 'Extremely Late' i.e. more than 365.
 
         thus, over 80% of tickets were resolved between 1 to 31 days.
         

### Ward

2.1) Unspecified - 100,023

2.2) Ward 08 - 20,310

2.3) Ward 11 - 17,479

2.4) Ward 07 - 17,089

2.5) Ward 02 - 16,884

2.6) Ward 06 - 16,367

2.7) Ward 01 - 14,808

2.8) Ward 10 - 12,813

2.9) Ward 09 - 11,543

2.10) Ward 05 - 10,694

2.11) Ward 04 - 10,304

2.12) Ward 03 - 9,744

2.13) Ward 12 - 9,166

        thus, 37.43% (100,023) of the total tickets were gotten from unspecified wards.

### Status Detail

3.1) Assesment in Progress - 1,636 open tickets.

3.2) Citizen Contacted - 24 open tickets.

3.3) Investigation Ongoing - 6 open tickets.

3.4) Unspecified - 4,834 open tickets.

3.5) Warning Issued - 85 open tickets.

3.6) Work Planned-Route/Schedule Dependent - 28 open tickets.

3.7) Work Planned-Seasonal/Weather Dependent - 2 open tickets

        hence, these figures show the number of tickets that are unsettled using the aforementioned status detail as a filter.
