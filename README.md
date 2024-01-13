import csv
import statistics

# read data 
def read_file(file_path):
    data = []
    with open(file_path, 'r') as file:
        reader = csv.reader(file)
        for row in reader:
            data.append(row)
    return data

# Function to calculate project utilization rate
def calculate_utilization(timesheet_data):
    utilization_data = []
    for entry in timesheet_data[1:]:  
        employee_id, hours_worked = map(int, entry)
        utilization = min(1.0, hours_worked / 2000)  # Utilization capped at 100%
        utilization_data.append((employee_id, utilization))
    return utilization_data

# Function to calculate qualitative evaluation metric for Consultants
def calculate_evaluation_metric(evaluation_data):
    evaluation_metric_data = []
    for entry in evaluation_data[1:]: 
        employee_id, comments = entry[0], entry[1:]
        positive_keywords = ['excellent', 'good', 'prompt', 'dependable']
        negative_keywords = ['poor', 'error', 'unreliable', 'late']

        total_keywords = len(comments)
        positive_count = sum(keyword in ' '.join(comments) for keyword in positive_keywords)
        negative_count = sum(keyword in ' '.join(comments) for keyword in negative_keywords)

        if total_keywords == 0:
            evaluation_score = 50 
        else:
            evaluation_score = round((positive_count / total_keywords) * 100)

        evaluation_metric_data.append((employee_id, evaluation_score))
    return evaluation_metric_data

# Function to calculate bonuses
def calculate_bonus(eligible_employees, bonus_rate):
    bonus_data = []
    for eid, job_code, base_pay, util, eval_metric, _, sales in eligible_employees:
        if job_code == 'C':  # Consultant
            if util > 0.6 and eval_metric >= 75.0 / 100:  # Fix: Convert 75 to percentage
                bonus = min(base_pay * bonus_rate, 50000)  # Cap bonus at $50,000
            else:
                bonus = 0
        elif job_code == 'D':  # Director
            if util > 0.6 and sales is not None:
                bonus = min(sales * bonus_rate, 150000)  # Cap bonus at $150,000
            else:
                bonus = 0
        else:
            bonus = 0

        bonus_data.append((eid, bonus))

    return bonus_data

# Function to store results in emp_end_yr.txt
def store_results(employee_data, utilization_data, evaluation_metric_data, bonus_data, file_path):
    with open(file_path, 'w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(['ID', 'LastName', 'FirstName', 'JobCode', 'BasePay', 'Utilization', 'Evaluation/Sales', 'Bonus'])

        for employee in employee_data[1:]: 
            eid, last_name, first_name, job_code, base_pay, *_ = employee
            utilization = next((util for emp_id, util in utilization_data if emp_id == int(eid)), 0)
            evaluation_metric = next((score for emp_id, score in evaluation_metric_data if emp_id == int(eid)), 0)
            bonus = next((b for emp_id, b in bonus_data if emp_id == int(eid)), 0)

            writer.writerow([int(eid), last_name, first_name, job_code, float(base_pay), utilization, evaluation_metric, bonus])

# Function to search for employee information by ID
def search_employee_info(employee_id, employee_data):
    employee_id = int(employee_id)
    employee = next((emp for emp in employee_data[1:] if int(emp[0]) == employee_id), None)

    if employee:
        print(f'ID: {employee[0]}')
        print(f'{job_title(employee[3])}: {employee[2]} {employee[1]}')

    
        if len(employee) > 5:
            try:
                utilization = float(employee[5]) * 100
                print(f'Utilization: {utilization:.2f}%')
            except ValueError:
                print('Utilization information is not a valid number.')

        if len(employee) > 6:
            try:
                evaluation_score = int(employee[6])
                print(f'Evaluation score: {evaluation_score}')
            except ValueError:
                print('Evaluation score is not a valid integer.')

        if len(employee) > 4:
            try:
                base_pay = float(employee[4])
                print(f'Base pay: ${base_pay:,.2f}')
            except ValueError:
                print('Base pay is not a valid number.')

        if len(employee) > 7:
            try:
                bonus = float(employee[7])
                print(f'Bonus: ${bonus:,.2f}')
            except ValueError:
                print('Bonus is not a valid number.')

    else:
        print(f'Employee with ID {employee_id} not found.')

# Function to provide basic descriptive analytics
def descriptive_analytics(utilization_data, sales_data, evaluation_metric_data, bonus_data):
    
    utilization_values = [util for _, util in utilization_data]
    print("Utilization Statistics:")
    print(f"Number of data points: {len(utilization_values)}")
    print(f"Minimum: {min(utilization_values):.2%}")
    print(f"Maximum: {max(utilization_values):.2%}")
    print(f"Mean: {statistics.mean(utilization_values):.2%}")
    print(f"Median: {statistics.median(utilization_values):.2%}")
    print(f"Standard Deviation: {statistics.stdev(utilization_values):.2%}\n")

    # Sales (for Directors only)
    if sales_data:
        sales_values = [float(sales) for _, sales in sales_data[1:]]  # Skip the header row
        print("Sales Statistics:")
        print(f"Number of data points: {len(sales_values)}")
        print(f"Minimum: ${min(sales_values):,.2f}")
        print(f"Maximum: ${max(sales_values):,.2f}")
        print(f"Mean: ${statistics.mean(sales_values):,.2f}")
        print(f"Median: ${statistics.median(sales_values):,.2f}")
        print(f"Standard Deviation: ${statistics.stdev(sales_values):,.2f}\n")

    # Evaluation Metric (for Consultants only)
    if evaluation_metric_data:
        eval_metric_values = [score for _, score in evaluation_metric_data[1:]]  # Skip the header row
        print("Evaluation Metric Statistics:")
        print(f"Number of data points: {len(eval_metric_values)}")
        print(f"Minimum: {min(eval_metric_values)}")
        print(f"Maximum: {max(eval_metric_values)}")
        print(f"Mean: {statistics.mean(eval_metric_values)}")
        print(f"Median: {statistics.median(eval_metric_values)}")
        print(f"Standard Deviation: {statistics.stdev(eval_metric_values)}\n")

    # Bonus Amount (for those who qualified for a bonus)
    if bonus_data:
        bonus_values = [bonus for _, bonus in bonus_data[1:]]  
        print("Bonus Amount Statistics:")
        print(f"Number of data points: {len(bonus_values)}")
        print(f"Minimum: ${min(bonus_values):,.2f}")
        print(f"Maximum: ${max(bonus_values):,.2f}")
        print(f"Mean: ${statistics.mean(bonus_values):,.2f}")
        print(f"Median: ${statistics.median(bonus_values):,.2f}")
        print(f"Standard Deviation: ${statistics.stdev(bonus_values):,.2f}\n")

# Function to identify Consultants with poor performance
def identify_consultants_with_poor_performance(utilization_data, evaluation_metric_data):
    utilization_values = [util for _, util in utilization_data[1:]]  # Skip the header row
    mean_utilization = sum(utilization_values) / len(utilization_values)
    std_dev_utilization = statistics.stdev(utilization_values)

    poor_consultants = [
        (employee_id, utilization)
        for employee_id, utilization in utilization_data[1:]
        if utilization < mean_utilization - std_dev_utilization and
        next((score for eid, score in evaluation_metric_data[1:] if eid == employee_id), 0) == 0
    ]

    return poor_consultants




# Helper function to determine job title
def job_title(job_code):
    return 'Consultant' if job_code == 'C' else 'Director'

# Main script
# Main script
evaluation_data = read_file('evaluation.txt')
timesheet_data = read_file('timesheet.txt')
sales_data = read_file('sales.txt')
employee_data = read_file('emp_beg_yr.txt')

utilization_data = calculate_utilization(timesheet_data)
evaluation_metric_data = calculate_evaluation_metric(evaluation_data)

bonus_rate = float(input("Enter the bonus percentage rate: ")) / 100

eligible_employees = []

# Debug: Print headers for clarity
print("\nDebug: Headers for eligible_employees")
print("ID, JobCode, BasePay, Utilization, EvaluationMetric, None, None")

for employee in employee_data[1:]:  
    eid, _, _, job_code, base_pay, *_ = employee
    # Calculate utilization
    util = next((util for emp_id, util in utilization_data if emp_id == int(eid)), 0)

    # Calculate evaluation metric
    eval_metric = next((score for emp_id, score in evaluation_metric_data if emp_id == int(eid)), 0)

    # Debug: Print each eligible employee's information
    print(f"{eid}, {job_code}, {float(base_pay)}, {util}, {eval_metric}, None, None")

    # Append the tuple to eligible_employees
    eligible_employees.append((int(eid), job_code, float(base_pay), util, eval_metric, None, None)) 

bonus_data = calculate_bonus(eligible_employees, bonus_rate)

# Store results in emp_end_yr.txt
store_results(employee_data, utilization_data, evaluation_metric_data, bonus_data, 'emp_end_yr.txt')

# Example: Search for employee information
search_employee_info(101, employee_data)

# Example: Descriptive analytics
descriptive_analytics(utilization_data, sales_data, evaluation_metric_data, bonus_data)

# Example: Identify Consultants with poor performance
poor_consultants = identify_consultants_with_poor_performance(utilization_data, evaluation_metric_data)
print("Consultants with poor performance:")
for eid, utilization in poor_consultants:
    print(f'ID: {eid}, Utilization: {utilization * 100:.2f}%')
