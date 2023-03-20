# Using Jupyter Notebooks to Track School Performance

# SUMMARY
In this project, I was expected to merge two .csv files with school data and student data from one school district, then sort schools and students by various metrics, and finally to compare schools by the scores their students earned and the percentage of students who passed. The results of this analysis can be found in the PyCitySchools folder, in the Jupyter Notebook file. 

Schools were compared by the scores they earned in reading and math classes; the percentage of students who passed reading and math; and the percentage of students who passed both. Students were considered to pass these subjects if they earned a score of 70 or greater.

Much of the code was guided by starter code, provided in the assignment folder. The starter code can be found alongside the .csv files in this repository. 

# DISTRICT SUMMARY
To find the total number of schools and students, I used the ‘nunique()’ function in pandas. This function counts the number of unique values in a given column. However, I had difficulty using the “student_name” column to find the total number of unique students; the value printed in the Jupyter Notebook was different from the solution provided with the assignment. Therefore, I chose to use the “Student ID” column instead. This suggests there may be repeated names in the “student_name” column, which would not be counted in the ‘nunique()’ function, as they are not indeed unique. To find the total budget of all schools in the district, I had the program track all the unique budget values in the district, then used the ‘sum()’ function to add them together. To find the average reading and math scores, I simply used the ‘mean()’ function.

To find the percentage of students in the district who passed reading, math, and both subjects, I had the program find the number of students in the “math_score” and “reading_score” columns who received scores of 70 or more, then divided that number by the total number of students. To find the percentage of students who passed overall for example, I used the following code:

passing_math_reading_count = school_data_complete[
    (school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)
].count()["student_name"]
overall_passing_rate = passing_math_reading_count /  float(student_count) * 100

Each of these values were then placed into a DataFrame and displayed below the district summary section.




#SCHOOL SUMMARY
To find how each individual school performed, I made heavy use of the ‘groupby’ function. The program was told to group the data by school name, then to find the data connected to each school name. For example, school types were found by writing this code:

#Determine each school's type
school_types = school_data_complete.groupby("school_name")["type"].unique()

Although it required additional calculation, the passage rates at each school was found in a similar way, like this:

passing_math_reading = school_data_complete[(school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)]
#group passing math scores by their connected school name, find the number at each school that are passing scores, then divide by number of students to find percentage of passing students.
passing_math_reading = passing_math_reading.groupby(["school_name"]).size() / students_per_school * 100
passing_math_reading = passing_math_reading.__round__(2)


Each of these grouped data sets, were displayed as DataFrames beneath their respective sections of the Jupyter Notebook, then later merged like this into a larger school summary.

per_school_summary = school_types_df.join(students_per_school, on="School Name")
per_school_summary = pd. merge(per_school_summary, per_school_budget, on="School Name")
per_school_summary = pd.merge(per_school_summary, per_capita_spending, on="School Name")
per_school_summary = pd.merge(per_school_summary, per_school_math, on="School Name")
per_school_summary = pd.merge(per_school_summary, per_school_reading, on="School Name")
per_school_summary = pd.merge(per_school_summary, passing_math, on="School Name")
per_school_summary = pd.merge(per_school_summary, passing_reading, on="School Name")
per_school_summary = pd.merge(per_school_summary, passing_math_reading, on="School Name")

per_school_summary = per_school_summary.rename(columns={"Student ID": "Number of Students"})

To prevent an error the first column, tracking students per school, uses a ‘join’ function rather than a ‘merge’ function. Since this solution marks itself as “Student ID,” since that is the value it tracks to find the number of schools, this column is renamed under the merged DataFrames.



#RANKING SCHOOL PERFORMANCE
To find the top five and bottom five performing schools, I used a simple ‘sort_values’ function that referenced the school summary DataFrame, with descending values for the top five and ascending values for the bottom five. The code for both is below: 

# Sort the schools by `% Overall Passing` in descending order and display the top 5 rows.
highest_performing = per_school_summary.sort_values(["% Passing Overall"], ascending=False)

# Sort the schools by `% Overall Passing` in ascending order and display the top 5 rows.
bottom_performing = per_school_summary.sort_values(["% Passing Overall"], ascending=True)

These DataFrames were printed to show only ‘head(),’ or the top five values listed in each DataFrame.



#AVERAGE SCORES BY GRADE, SORTED BY SCHOOL
To find the average scores in reading and math by grade, I had the program reference the original DataFrame in the Jupyter Notebook, listing all data by individual students. Then, I told it to track each student by their grade level, to group them by school, and finally to find the mean of their scores in each subject. This sorted data was then placed together in separate DataFrames for reading and math and displayed beneath respective sections. For reference, the code for the math section – which is identical to reading – is displayed below:
ninth_graders_scores = school_data_complete[school_data_complete["grade"] == "9th"].groupby("school_name").mean()["math_score"]
tenth_graders_scores = school_data_complete[school_data_complete["grade"] == "10th"].groupby("school_name").mean()["math_score"]
eleventh_graders_scores = school_data_complete[school_data_complete["grade"] == "11th"].groupby("school_name").mean()["math_score"]
twelfth_graders_scores = school_data_complete[school_data_complete["grade"] == "12th"].groupby("school_name").mean()["math_score"]

#round values to 2 decimal places
ninth_graders_scores = ninth_graders_scores.__round__(2)
tenth_graders_scores = tenth_graders_scores.__round__(2)
eleventh_graders_scores = eleventh_graders_scores.__round__(2)
twelfth_graders_scores = twelfth_graders_scores.__round__(2)

# Combine each of the scores above into single DataFrame called `math_scores_by_grade`
math_scores_by_grade = pd.DataFrame({
    "9th": ninth_graders_scores,
    "10th": tenth_graders_scores,
    "11th": eleventh_graders_scores,
    "12th": twelfth_graders_scores
})

# Minor data wrangling
math_scores_by_grade.index.name = None

# Display the DataFrame
math_scores_by_grade



#SORTING SCORES BY SPENDING AND SIZE
To sort data by spending and size, I sorted the data into bins that were outlined by the starter code, based on the school summary DataFrame. After making a new copy of that DataFrame in the respective sections for spending and size, I used ‘pd.cut’ to extract the columns that were pertinent to both sections. For spending, this was the “Per Capita Spending ($)” column, and for size, it was the “Number of Students” column. Then, I grouped the original DataFrame’s data by spending or size, depending upon which was pertinent, and found the average scores and percentages at schools within the bin limits of spending and size. The code for spending is shown below, with dashes representing the cells of the Jupyter Notebook:

# Establish the bins 
spending_bins = [0, 585, 630, 645, 680]
labels = ["<$585", "$585-630", "$630-645", "$645-680"]


# Create a copy of the school summary since it has the "Per Student Budget" 
school_spending_df = per_school_summary.copy()



# Use `pd.cut` to categorize spending based on the bins.
school_spending_df["Spending Ranges (Per Student)"] = pd.cut(school_spending_df["Per Capita Spending ($)"], bins=spending_bins, labels=labels)


#  Calculate averages for the desired columns. 
spending_math_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]
spending_reading_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]
spending_passing_math = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Students Passing Math"]
spending_passing_reading = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Students Passing Reading"]
overall_passing_spending = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Overall"]


# Assemble into DataFrame
spending_summary = spending_summary_df = pd.DataFrame({
    "Average Math Score": spending_math_scores.__round__(2),
    "Average Reading Score": spending_reading_scores.__round__(2),
    "% Passing Math": spending_passing_math.__round__(2),
    "% Passing Reading": spending_passing_reading.__round__(2),
    "% Overall Passing": overall_passing_spending.__round__(2)
})

# Display results
spending_summary



#SORTING SCORES BY SCHOOL TYPE
In many ways, this section was similar to those above, grouping scores in the school summary DataFrame by each school’s type – district or charter – then finding the average scores and percentages of students who passed. However, due to a repeated type error caused by the school type data, which the program was attempting to average, I added the following line of code:
#convert school type column values to strings to prevent type errors
per_school_summary["School Type"] = per_school_summary["School Type"].astype(str)

This would prevent the program from attempting to find the average of the words “district” and “charter,” establishing them clearly as strings rather than numbers.

The following code was used to create a DataFrame with five columns, which listed average scores and percentages as follows:
# Assemble the new data by type into a DataFrame called `type_summary`
# Create the type_summary DataFrame
type_summary = pd.DataFrame({
    "Average Math Score": average_math_score_by_type.__round__(2),
    "Average Reading Score": average_reading_score_by_type.__round__(2),
    "% Passing Math": average_percent_passing_math_by_type.__round__(2),
    "% Passing Reading": average_percent_passing_reading_by_type.__round__(2),
    "% Overall Passing": average_percent_overall_passing_by_type.__round__(2)
})




#Authors:
Daniel Adamson





