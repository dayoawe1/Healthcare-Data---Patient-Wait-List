
## Healthcare Data-Patient Wait List Dashboard

#### Dashboard Link : https://app.powerbi.com/groups/me/reports/43a8a849-d4b3-439c-8eae-052e20ef0e8a/ReportSection?experience=power-bi

The healthcare Data-Patient Wait List captures detailed information about patient waitlists in a healthcare setting, focusing on both inpatient and outpatient categories. 'Inpatient' refers to a patient who stays in a hospital while under treatment, and 'outpatient' refers to a patient who receives medical treatment without being admitted to a hospital.

### Dataset Purpose
The dataset is used to analyze patient wait times across different specialties and demographics, providing insights into areas where healthcare services may need improvement. This involves identifying trends in patient wait times, evaluating the effectiveness of patient flow management, and informing decisions on resource allocation to reduce waiting periods for specific treatments or demographics.

# Overall Objectives
### Project Goals
1. Track Current Status of Patient waiting list
2. Analyze historical monthly trends of waiting list in Inpatient and Outpatient categories.
3. detailed specialty level and age profile analysis.

### Data Scope
The data covers the years 2018 to 2021. Each category, 'Inpatient' and 'Outpatient', has four CSV files corresponding to each year within this range. The files have the following key data columns: 

1. Archive_Date: The specific date when the data was recorded or archived.
2. Specialty_Name: The medical specialty involved, such as Neurology, Ophthalmology, or Orthopedics.
3. Specialty_HIPE: A numeric code corresponding to the medical specialty as per the HIPE (Hospital In-Patient Enquiry).
4. Case_Type: Indicates whether the case is classified as "Day Case" or "Inpatient" or "Outpatient".
5. Adult_Child: Specifies if the patient is an adult or a child.
6. Age_Profile: The age range of the patient.
7. Time_Bands: The time patients have been on the waitlist, categorized into bands.
8. Total: The number of patients within the given specialty, case type, age profile, and time band.

### Metrics Required
1. Current Total wait list.
2. Average and Median waiting list.

### Views Required
1. Summary page
2. Detailed page for Granular Analysis

# Steps Followed
### Data Collection.
We utilized the folder data connector to import our data into Power BI. The data is organized into two folders—'Inpatient' and 'Outpatient'—each containing four files. Before importing, we ensured that all files across both folders had consistent column counts and identical headers. In Power BI, start by clicking Get Data and selecting the Folder connector. Browse to and locate the Inpatient folder, then click Combine and Load to merge the four files into a single dataset. The same was done for Outpatient data.

Snap of files comination for Inpatient data

![Snapshot of file combination](https://github.com/user-attachments/assets/fd4c4eb6-3a43-4825-9d65-00235cc4c7eb)


### Data Transformation and Modeling
- In the Inpatient data, all columns were assigned the correct data type except for the "Archive_Date" column, which was incorrectly captured as text instead of a date. To transform this, follow these steps in Power Query:

a. Click on the Add Column tab.

b. Select Custom Column.

c. Assign a new column name, such as Archive_Date_New.

d. Use the following M query formula to format the new column correctly (custom column formula)

            try Date.FromText(Text.Start([Archive_Date],2) & "-" 
            & Text.Middle([Archive_Date],3,2) & "-"
            & Text.End([Archive_Date],4))  
            
            otherwise
             Date.FromText(Text.Middle([Archive_Date],3,2) & "-" 
            & Text.Start([Archive_Date],2) & "-" 
            & Text.End([Archive_Date],4))

Snap of corrected Archive_Date datatype

![Archive Date New](https://github.com/user-attachments/assets/0f1d7442-82ea-4555-ace3-b3565d84f6b1)


- In the outpatient data, three transformations were applied:

a. The Archive_Date column's data type was corrected using the same steps as in the Inpatient data.

b. The Speciality column was renamed to Specialty_Name to match the corresponding column in the Inpatient data.

c. Since the outpatient data does not have a Case_Type column, a new custom column named Case_Type was created using the M query formula "Outpatient". This column was then positioned between Specialty_Name and Adult_Child to align with the layout in the Inpatient data.

Snap of renamed "Speciality" column to "Specialty_Name" and added "Case_type" column

![Specialty and case type](https://github.com/user-attachments/assets/5890ea3e-5b9c-4764-815f-c717c6c4b207)


- The "Inpatient" and "Outpatient" datasets were then merged by using the Append as New Query option, and renamed to All_Data

-Due to the large number of specialties in the Specialty_Name column, I decided to categorize them into broader groups. To achieve this, I manually created a mapping table that associates each specialty with its corresponding Specialty_Group. This effort was aimed at enhancing the visualization and analysis of the specialties in the next step. The mapping table was then loaded into Power BI.

Snap of manually created mapping table

![Mapping table](https://github.com/user-attachments/assets/cb455515-c6d2-48ad-abf4-a84dc7ad408b)


- For the Modeling, a one-to-many (1:M) relationship was created between the Specialty_Name in the main data table and the Specialty in the mapping table. 

Snap of the Modeling view

![Modeling](https://github.com/user-attachments/assets/b985d60a-1e8b-46bf-b2cc-c90624aded87)


### Data Visualization
- Step 1: New measures were created to find current month wait list versus same month previous year wait list.

The following DAX expressions was written for the same

    Latest Month Wait List = CALCULATE(SUM(All_Data[Total]),
    All_Data[Archive_Date New] = MAX(All_Data[Archive_Date New])) + 0

    PY Latest Month Wait List = CALCULATE(SUM(All_Data[Total]),
    All_Data[Archive_Date New] = EDATE(MAX(All_Data[Archive_Date 
    New]),-12)) + 0

Two card visuals were added to the report view to represent the Latest Month Wait List and same month Previous Year Wait List

![PY](https://github.com/user-attachments/assets/9d9541c1-70b2-4e5e-8b71-c23dab2ef9fb)

- Step 2: We created a dummy table (Calc Method) with values 'Average' and 'Median' to enable dynamic switching between these metrics, allowing us to effectively address outliers in the waitlist data.

- Step 3: Visual filter (Slicers) was added to the report view for the "Average" and "Median" to serves as a toggle / selector for the user to switch between the two metrics.

![Avgmed](https://github.com/user-attachments/assets/514e0b85-3dae-421f-b6d9-9bcad594da30)


- Step 4: Measures for Average and Median Waitlist was created using the following DAX expressions 

      Average Wait List = AVERAGE(All_Data[Total])

      Median Wait List = MEDIAN(All_Data[Total])

- Step 5:  A new DAX measure that switches between the Average Wait list and Median Wait list based on the slicer selection was created using the following DAX expression

      Avg/Med Wait List = SWITCH(VALUES('Calculation Method'
      [Calc Method]),"Average",[Average Wait List],
       "Median",[Median Wait List])

- Step 6: A donut chart was also included in the report design to display the "Average/Median Wait list" by "Case Type" (Day Case, Inpatient, and Outpatient).

![Screenshot 2024-08-12 110254](https://github.com/user-attachments/assets/86f37d4d-d042-405c-ba6b-022fc9cea263)


- Step 7: A stacked column chart was also added to the report design area to show the relationship between "Time_Bands" and "Age_Profile".

- Step 8: "The Time_Band" column in the 'All_Data' dataset contains repeated values that are displayed in the stacked column chart. To resolve this, go to Transform Data, select the Time_Band column, and replace the duplicate values to ensure consistency in the chart.

- Step 9: Blank values in the Time_Band and Age_Profile columns of the All_Data dataset were replaced with "No Input." Subsequently, the "No Input" values were filtered out from both columns using the Filter pane.

Snap of the stacked column chart

![Screenshot 2024-08-12 110307](https://github.com/user-attachments/assets/36d13403-8ad7-41e7-8824-3963b0b42333)


- Step 10: A multi-row card was also added to the report design area. The card was configured with "Specialty_Name" and "Avg/Med Waitlist" as the measure. A filter was then applied by choosing "Top N" in the filter pane, setting it to display the top five specialties by entering "5" as the limit based on the "Avg/Med wait list" value.

![Screenshot 2024-08-12 110403](https://github.com/user-attachments/assets/7aafcd72-d900-488a-9ae8-10f0fa2ae955)


- Step 11: Two line charts were also included in the report design area (Summary page), each displaying the sum total for Case_Type over Archive_Date. One of the line charts was filtered to show only Day Case and Inpatient, while the other was filtered to display only Outpatient.

![Screenshot 2024-08-12 110425](https://github.com/user-attachments/assets/ef78aca9-3554-40be-ad6c-9a55cfebfc90)


- Step 12: Three visual filters (slicers) were also added to the report design area: one for Archive_Date, another for Case_Type, and the last for Specialty_Name. Both slicers for Case_Type and Specialty_Name were formatted to dropdown

![Three visuals](https://github.com/user-attachments/assets/f73d1fe5-d6bc-466a-b425-73133e3a680b)


- Step 13. Another report page (Detail) was created for the visuals. On the detail report design area, five visual slicers were added for Archive_Date, Case_Type, Specialty_Name, Age_Profile and Time_Bands respectively.

![Screenshot 2024-08-12 111103](https://github.com/user-attachments/assets/2d7f48bc-d744-4c14-bcf1-d3e3be35769a)


- Step 14. A matrix visual was added to the detail report design area. Archive_Date, Specialty_Name, Age_Profile and Time_Bands were added to the rows of the visuals, Case_Type to the column and Total to the legend. 

![Screenshot 2024-08-12 111207](https://github.com/user-attachments/assets/82a2c2b4-9ef6-4898-b4f1-117447a6ec79)


- Step 15. To enhance the aesthetic appeal of the dashboard, we used https://stock.adobe.com/search?k=dashboard&search_type=recentsearch to find a dashboard background sample. We then utilized https://color.adobe.com/create/image to extract the themes and colors from the chosen sample. Afterward, the background was designed in PowerPoint and uploaded to Power BI. Next, formatting was applied to all visuals on both the summary and detail pages of the dashboard.

- Step 16. A new measure was created to dynamically display the selected metric either average or median using the following DAX expression.

      Dynamic Title = SWITCH(VALUES('Calculation Method'[Calc Method]),
      "Average","Key Indicators - Patient Wait List (Average)",
       "Median","Key Indicators - Patient Wait List (Median)")

A card visual for this dynamic title was then added to the report design area of the summary page.

![Average](https://github.com/user-attachments/assets/b9ca13be-deef-4a93-b112-92ace91eaf7c)

and

![Median](https://github.com/user-attachments/assets/5d4a5084-aeaf-4af5-ba9b-151c3d8af4fa)



#### Adding Interactivity or Navigation to Dashboard
- Step 17. New measures were created to account for blank when Case_Type and specialty filters are selected and formatted to be 'No data for selected criteria'  using the following DAX expressions

      NoDataLeft = IF(ISBLANK(CALCULATE(SUM(All_Data[Total]), 
      All_Data[Case_Type]<>"Outpatient")),"No data for selected 
      criteria","")

      NoDataRight = IF(ISBLANK(CALCULATE(SUM(All_Data[Total]), 
      All_Data[Case_Type]="Outpatient")),"No data for selected 
      criteria","")     


Snap of blank when Specialty_Name was filtered to Accident and Emergency

![NoDataLeft](https://github.com/user-attachments/assets/0e0ba8d6-13fb-444d-880e-c184416d97f0)

Snap of blank when Specialty_Name was filtered to Histopathology

![NoDataRight](https://github.com/user-attachments/assets/6d8b3962-81c4-4c96-a601-338d0e555be6)

- Step 18. We added a page Navigation button for creating a link between the summary page and detail page by inserting buttons on both pages
- Step 19. We incorporated a stacked bar chart and a visual card to design a custom tooltip, enhancing the dashboard's interactivity. 

Snap of Interactive design when we hover the line charts

![Screenshot 2024-08-12 112239](https://github.com/user-attachments/assets/ad5b374a-7f9a-42f7-bbd3-2c12eed9acc1)


## Snapshot of Dashboard (Power BI Service)
Summary Page

![Screenshot 2024-08-12 115241](https://github.com/user-attachments/assets/814f9948-2502-4294-960e-2be207ba8c4a)

Detail Page

![Screenshot 2024-08-12 115436](https://github.com/user-attachments/assets/bb1d342b-a8f6-4994-80d2-1645d2cda3e0)



# Report Snapshot (Power BI DESKTOP)
Summary View

![Screenshot 2024-08-12 120141](https://github.com/user-attachments/assets/0303f3ff-1745-41b6-ac42-b5ffcfc84aed)

Detailed View

![Screenshot 2024-08-12 120050](https://github.com/user-attachments/assets/ed3cfc52-3801-42e8-808c-f304ee42772a)


# Insights

Two pages of report were created on Power BI Desktop & it was then published to Power BI Service.

Following inferences can be drawn from the dashboard;
Insights and Summary from the Patient Wait List Dashboard:

#### 1. Overview:
 Total Wait List Comparison:

- The dashboard shows a comparison of the total patient wait list between the latest month and the previous year’s corresponding month.

- 709K patients are on the wait list for the latest month, compared to 640K patients in the previous year’s latest month. This indicates an increase in the patient wait list by 69K.

#### 2. Wait List Bifurcation by Case Type: 
The donut chart highlights the distribution of the Average and Median wait list by case type:
- Outpatient: Average of 80 cases (72.49%), and a median of 29 cases (74.36%) 
- Day Case: Average of 19 cases (16.89%), and a median of 6 cases (15.38%)
- Inpatient: Average of  12 cases (10.62%), and a median of 4 cases (10.26%)
Outpatients represent the majority of the wait list, followed by day cases and inpatients.
#### 3. Wait List Analysis by Time Band vs. Age Profile:
The bar chart provides an analysis of the wait list segmented by time bands and age profiles:

- The 18+ months time band has the highest number of patients waiting, predominantly within the 16-64 age group (136 patients).
- The 0-3 months and 3-6 months time bands also have a significant number of patients, especially in the 16-64 and 0-15 age groups.
- Patients aged 65+ have fewer wait times across all time bands.
#### 4. Top 5 Wait Lists by Specialty:
The top 5 specialties with the highest wait list counts:
- Paediatric Dermatology: 168 patients
- Paediatric ENT: 148 patients
- Paed Orthopaedic: 115 patients
- Accident & Emergency: 111 patients
- Paed Cardiology: 102 patients
Paediatric Dermatology has the highest wait list, suggesting a significant demand in this specialty.
#### 5. Monthly Trend Analysis:
Day Case/Inpatient vs. Outpatients:
- The line chart shows trends in patient counts over time for day cases, inpatients, and outpatients.
- Day Case and Inpatient numbers remain relatively stable, with some fluctuations.
- Outpatient numbers show a general upward trend, especially after January 2020, reaching 0.63M in January 2021.
#### 6. Detailed Grid View:
The detailed view breaks down the data by Archive Date, Case Type, Age Profile, and Time Band.
- Orthopaedics specialty and the 16-64 age group have the highest wait list counts.
- The Outpatient case type dominates the wait list in most age profiles and time bands.
#### 7. Key Insights:
- The overall patient wait list has increased, indicating growing demand or delays in patient processing.
- Outpatients make up the majority of the wait list, with particular pressure on Paediatric Dermatology and other paediatric specialties.
- The longest wait times are seen in the 18+ months time band, primarily affecting the 16-64 age group.
- The rising trend in outpatient numbers could indicate a shift towards more outpatient care, or delays in outpatient services leading to a backlog.
#### 8. Recommendations:
- Resource Allocation: Focus additional resources on reducing wait times in the high-demand specialties, particularly in paediatrics.
- Operational Efficiency: Investigate the reasons behind the increased wait list and take steps to streamline patient processing and care delivery.
- Monitoring and Adjustment: Regularly monitor the wait list trends and adjust strategies to manage patient load effectively.











