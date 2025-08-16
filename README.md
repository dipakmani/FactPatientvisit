import pandas as pd
import numpy as np
from faker import Faker
import random
from datetime import timedelta

fake = Faker()

# -----------------------------
# Configuration
# -----------------------------
num_records_fact = 500_000   # Fact table records
num_records_dim = 500        # Dimension table records

# -----------------------------
# Generate Dimension Tables (6 columns each)
# -----------------------------
def generate_dim_patient(n):
    df = pd.DataFrame({
        'PatientID': range(1, n+1),
        'PatientName': [fake.name() for _ in range(n)],
        'Gender': [random.choice(['Male','Female']) for _ in range(n)],
        'DOB': [fake.date_of_birth(minimum_age=0, maximum_age=90) for _ in range(n)],
        'City': [fake.city() for _ in range(n)],
        'State': [fake.state() for _ in range(n)]
    })
    return df

def generate_dim_doctor(n):
    return pd.DataFrame({
        'DoctorID': range(1, n+1),
        'DoctorName': [fake.name() for _ in range(n)],
        'Specialty': [random.choice(['Cardiology','Orthopedics','Neurology','General']) for _ in range(n)],
        'ExperienceYears': [random.randint(1,40) for _ in range(n)],
        'Department': [random.choice(['Surgery','Cardiology','Neurology','Orthopedics']) for _ in range(n)],
        'ContactNumber': [fake.phone_number() for _ in range(n)]
    })

def generate_dim_hospital(n):
    return pd.DataFrame({
        'HospitalID': range(1, n+1),
        'HospitalName': [fake.company() for _ in range(n)],
        'City': [fake.city() for _ in range(n)],
        'State': [fake.state() for _ in range(n)],
        'Capacity': [random.randint(50,500) for _ in range(n)],
        'AccreditationLevel': [random.choice(['A','B','C']) for _ in range(n)]
    })

def generate_dim_department(n):
    return pd.DataFrame({
        'DepartmentID': range(1, n+1),
        'DepartmentName': [random.choice(['Surgery','Cardiology','Neurology','Orthopedics']) for _ in range(n)],
        'Floor': [random.randint(1,10) for _ in range(n)],
        'HeadDoctorID': [random.randint(1,n) for _ in range(n)],
        'RoomCount': [random.randint(5,50) for _ in range(n)],
        'Budget': [random.randint(10000,500000) for _ in range(n)]
    })

def generate_dim_admission_type(n):
    return pd.DataFrame({
        'AdmissionTypeID': range(1, n+1),
        'Type': [random.choice(['Emergency','Routine','Transfer']) for _ in range(n)],
        'Description': [fake.sentence() for _ in range(n)],
        'Department': [random.choice(['Surgery','Cardiology','Neurology','Orthopedics']) for _ in range(n)],
        'CostEstimate': [random.randint(1000,50000) for _ in range(n)],
        'Notes': [fake.sentence() for _ in range(n)]
    })

def generate_dim_insurance(n):
    return pd.DataFrame({
        'InsuranceID': range(1, n+1),
        'ProviderName': [fake.company() for _ in range(n)],
        'PolicyType': [random.choice(['Basic','Premium','Gold','Silver']) for _ in range(n)],
        'CoveragePercent': [random.randint(50,100) for _ in range(n)],
        'ContactNumber': [fake.phone_number() for _ in range(n)],
        'Notes': [fake.sentence() for _ in range(n)]
    })

def generate_dim_payment_method(n):
    return pd.DataFrame({
        'PaymentMethodID': range(1, n+1),
        'Method': [random.choice(['Cash','Credit Card','Insurance','UPI']) for _ in range(n)],
        'Bank': [fake.company() for _ in range(n)],
        'AccountNumber': [fake.bothify(text='##########') for _ in range(n)],
        'Notes': [fake.sentence() for _ in range(n)],
        'TransactionFee': [random.randint(0,100) for _ in range(n)]
    })

def generate_dim_room(n):
    return pd.DataFrame({
        'RoomID': range(1, n+1),
        'RoomNumber': [random.randint(100,500) for _ in range(n)],
        'RoomType': [random.choice(['ICU','General','Private']) for _ in range(n)],
        'Floor': [random.randint(1,10) for _ in range(n)],
        'Capacity': [random.randint(1,5) for _ in range(n)],
        'AvailabilityStatus': [random.choice(['Available','Occupied','Maintenance']) for _ in range(n)]
    })

def generate_dim_medical_equipment(n):
    return pd.DataFrame({
        'EquipmentID': range(1, n+1),
        'EquipmentName': [fake.word().title() for _ in range(n)],
        'Category': [random.choice(['Monitoring','Surgical','Support','Imaging']) for _ in range(n)],
        'PurchaseDate': [fake.date_between(start_date='-10y', end_date='today') for _ in range(n)],
        'Cost': [random.randint(1000,50000) for _ in range(n)],
        'MaintenanceSchedule': [random.choice(['Monthly','Quarterly','Yearly']) for _ in range(n)]
    })

# -----------------------------
# Generate Fact_Visits Table
# -----------------------------
def generate_fact_visits(fact_records, dim_records):
    # Create dimension tables
    Dim_Patient_df = generate_dim_patient(dim_records)
    Dim_Doctor_df = generate_dim_doctor(dim_records)
    Dim_Hospital_df = generate_dim_hospital(dim_records)
    Dim_Department_df = generate_dim_department(dim_records)
    Dim_AdmissionType_df = generate_dim_admission_type(dim_records)
    Dim_Insurance_df = generate_dim_insurance(dim_records)
    Dim_PaymentMethod_df = generate_dim_payment_method(dim_records)
    Dim_Room_df = generate_dim_room(dim_records)
    Dim_Equipment_df = generate_dim_medical_equipment(dim_records)

    # Random foreign keys
    patient_ids = np.random.randint(1, dim_records+1, fact_records)
    doctor_ids = np.random.randint(1, dim_records+1, fact_records)
    hospital_ids = np.random.randint(1, dim_records+1, fact_records)
    department_ids = np.random.randint(1, dim_records+1, fact_records)
    admissiontype_ids = np.random.randint(1, dim_records+1, fact_records)
    insurance_ids = np.random.randint(1, dim_records+1, fact_records)
    paymentmethod_ids = np.random.randint(1, dim_records+1, fact_records)
    room_ids = np.random.randint(1, dim_records+1, fact_records)
    equipment_ids = np.random.randint(1, dim_records+1, fact_records)

    # Admission and Discharge dates
    admission_dates = [fake.date_between(start_date='-5y', end_date='today') for _ in range(fact_records)]
    discharge_dates = [ad + timedelta(days=random.randint(1,15)) for ad in admission_dates]

    # Age & AgeGroup
    ages = [random.randint(0,90) for _ in range(fact_records)]
    age_groups = ['Child' if a<18 else 'Adult' if a<60 else 'Senior' for a in ages]

    # Base fact table
    fact = pd.DataFrame({
        'VisitID': range(1, fact_records+1),
        'PatientID': patient_ids,
        'DoctorID': doctor_ids,
        'HospitalID': hospital_ids,
        'DepartmentID': department_ids,
        'AdmissionTypeID': admissiontype_ids,
        'InsuranceID': insurance_ids,
        'PaymentMethodID': paymentmethod_ids,
        'RoomID': room_ids,
        'EquipmentID': equipment_ids,
        'AdmissionDate': admission_dates,
        'DischargeDate': discharge_dates,
        'Age': ages,
        'AgeGroup': age_groups,
        'VisitCost': np.random.randint(1000,50000, fact_records)
    })

    # Merge dimension attributes beside IDs
    fact = fact.merge(Dim_Patient_df, on='PatientID', how='left', suffixes=('','_Patient'))
    fact = fact.merge(Dim_Doctor_df, on='DoctorID', how='left', suffixes=('','_Doctor'))
    fact = fact.merge(Dim_Hospital_df, on='HospitalID', how='left', suffixes=('','_Hospital'))
    fact = fact.merge(Dim_Department_df, on='DepartmentID', how='left', suffixes=('','_Department'))
    fact = fact.merge(Dim_AdmissionType_df, on='AdmissionTypeID', how='left', suffixes=('','_AdmissionType'))
    fact = fact.merge(Dim_Insurance_df, on='InsuranceID', how='left', suffixes=('','_Insurance'))
    fact = fact.merge(Dim_PaymentMethod_df, on='PaymentMethodID', how='left', suffixes=('','_PaymentMethod'))
    fact = fact.merge(Dim_Room_df, on='RoomID', how='left', suffixes=('','_Room'))
    fact = fact.merge(Dim_Equipment_df, on='EquipmentID', how='left', suffixes=('','_Equipment'))

    return fact

# -----------------------------
# Generate data and save CSV
# -----------------------------
Fact_Visits = generate_fact_visits(num_records_fact, num_records_dim)
Fact_Visits.to_csv('Fact_Visits_Denormalized.csv', index=False)

print("Fact_Visits CSV generated with 500,000 records and 9 dimensions with related attributes!")
