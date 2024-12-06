# Parse-8-Bank-Statements-using-AI-LLM-ML-CV
 Give you any US based bank statement PDF, you give us back a JSON file (see below) we can parse in our Python code. We want ALL info
Name of Bank, Account Number, Statement Date, Customer info such as name, address, city state zip code, all balance info, all transaction info, interest, account summary, overdraft, returned fees, any feeds, daily balance. All numbers relating to the customers US Based Bank Statement PDF file. No matter what the bank we get a standardized JSON from your code. We will used  your code to parse bank statements we collect via email attachments or file upload attachment. I have attached/uploaded a few bank statements for you to review.
======
To extract data from bank statement PDFs and return it in a standardized JSON format, you need to build a PDF parsing solution that can handle different bank statement layouts. This is challenging, as each bank may use its own format and structure for the statement.

For this task, we will utilize Python libraries that help in extracting text from PDFs, followed by structured parsing and regular expression-based methods to identify and organize the required fields.
Required Libraries:

    PyPDF2 or pdfplumber: To extract text from PDFs.
    re (Regular Expressions): To parse and match specific data (like bank name, account number, balance info, etc.).
    pandas: (optional) To handle tabular transaction data if needed.

Here's the Python code that can be used to extract the required information from any US-based bank statement PDF file.
Steps:

    Extract text from PDF.
    Use regex patterns to find specific fields (like name, address, account number, etc.).
    Format the parsed data into JSON.

Installation of Required Libraries:

pip install PyPDF2 pdfplumber pandas

Python Code:

import pdfplumber
import re
import json
from typing import Dict

def extract_bank_statement_data(pdf_path: str) -> Dict:
    # Open PDF using pdfplumber
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        for page in pdf.pages:
            text += page.extract_text()
    
    # Define regex patterns for each of the data fields
    patterns = {
        'bank_name': r"(?i)(?<=Bank\s)([A-Za-z\s]+)(?=\s|,)",
        'account_number': r"Account Number\s*[:\-]?\s*(\d{4,})",
        'statement_date': r"Statement Date\s*[:\-]?\s*([A-Za-z]+\s+\d{1,2},\s*\d{4})",
        'customer_name': r"Customer\s+Name\s*[:\-]?\s*([A-Za-z\s]+)",
        'customer_address': r"Address\s*[:\-]?\s*([\d\s\w\.,]+)",
        'customer_city_state_zip': r"([A-Za-z\s]+,\s+[A-Za-z]+)\s+(\d{5})",
        'balance_info': r"Balance\s*[:\-]?\s*\$([\d,]+\.\d{2})",
        'interest_info': r"Interest\s*[:\-]?\s*\$([\d,]+\.\d{2})",
        'overdraft_info': r"Overdraft\s*[:\-]?\s*\$([\d,]+\.\d{2})",
        'returned_fees': r"Returned\s+Fees\s*[:\-]?\s*\$([\d,]+\.\d{2})",
        'transactions': r"(?<=\n)([A-Za-z\s]+[\d{1,2},\s]+[A-Za-z\s]+(?:[\d{2}]+[\.\d]{2}|\$[\d,]{3,}))+"
    }

    # Initialize result dictionary
    result = {
        'bank_name': '',
        'account_number': '',
        'statement_date': '',
        'customer': {
            'name': '',
            'address': '',
            'city': '',
            'state': '',
            'zip_code': '',
        },
        'balance': {
            'current_balance': '',
            'interest': '',
            'overdraft': '',
            'returned_fees': ''
        },
        'transactions': []
    }

    # Extract data using regex patterns
    result['bank_name'] = re.search(patterns['bank_name'], text).group(1) if re.search(patterns['bank_name'], text) else 'N/A'
    result['account_number'] = re.search(patterns['account_number'], text).group(1) if re.search(patterns['account_number'], text) else 'N/A'
    result['statement_date'] = re.search(patterns['statement_date'], text).group(1) if re.search(patterns['statement_date'], text) else 'N/A'
    
    # Extract customer info
    result['customer']['name'] = re.search(patterns['customer_name'], text).group(1) if re.search(patterns['customer_name'], text) else 'N/A'
    result['customer']['address'] = re.search(patterns['customer_address'], text).group(1) if re.search(patterns['customer_address'], text) else 'N/A'
    
    city_state_zip = re.search(patterns['customer_city_state_zip'], text)
    if city_state_zip:
        result['customer']['city'] = city_state_zip.group(1).split(',')[0]
        result['customer']['state'] = city_state_zip.group(1).split(',')[1].strip()
        result['customer']['zip_code'] = city_state_zip.group(2)
    
    # Balance information
    result['balance']['current_balance'] = re.search(patterns['balance_info'], text).group(1) if re.search(patterns['balance_info'], text) else 'N/A'
    result['balance']['interest'] = re.search(patterns['interest_info'], text).group(1) if re.search(patterns['interest_info'], text) else 'N/A'
    result['balance']['overdraft'] = re.search(patterns['overdraft_info'], text).group(1) if re.search(patterns['overdraft_info'], text) else 'N/A'
    result['balance']['returned_fees'] = re.search(patterns['returned_fees'], text).group(1) if re.search(patterns['returned_fees'], text) else 'N/A'
    
    # Transaction information (simplified)
    transactions = re.findall(patterns['transactions'], text)
    for transaction in transactions:
        result['transactions'].append({
            'transaction_details': transaction.strip()
        })
    
    return result

def main():
    # Path to your bank statement PDF
    pdf_path = 'path_to_bank_statement.pdf'
    
    # Parse the bank statement and extract the data
    bank_statement_data = extract_bank_statement_data(pdf_path)
    
    # Convert the result to JSON
    print(json.dumps(bank_statement_data, indent=4))

if __name__ == "__main__":
    main()

Breakdown of Code:

    PDF Extraction: pdfplumber extracts the text from each page of the PDF. This is useful since most PDFs from banks are text-based (i.e., not scanned images).

    Regex Patterns: These patterns are designed to match:
        Bank Name (bank_name).
        Account Number (account_number).
        Statement Date (statement_date).
        Customer Name, Address, City, State, and Zip Code (customer_name, customer_address, customer_city_state_zip).
        Balance Information (balance_info).
        Interest, Overdraft, Returned Fees (interest_info, overdraft_info, returned_fees).
        Transaction Information (transactions).

    Structured JSON Output: The extracted data is structured in a dictionary, which is then printed as a JSON object using json.dumps.

Example Output (JSON):

{
    "bank_name": "Example Bank",
    "account_number": "1234567890",
    "statement_date": "January 1, 2024",
    "customer": {
        "name": "John Doe",
        "address": "1234 Elm St, Apt 56",
        "city": "Cityname",
        "state": "CA",
        "zip_code": "12345"
    },
    "balance": {
        "current_balance": "5000.00",
        "interest": "15.00",
        "overdraft": "0.00",
        "returned_fees": "35.00"
    },
    "transactions": [
        {
            "transaction_details": "Payment to ABC Store: -50.00"
        },
        {
            "transaction_details": "Deposit from XYZ Corp: +500.00"
        }
    ]
}

Next Steps:

    Testing: You’ll need to test the code with several bank statements from different institutions to ensure the regex patterns work correctly across various formats.
    Improvement: You can refine the regex patterns further if some specific data is not being captured properly, depending on the variability of the bank statements.

This code provides a robust starting point for extracting essential information from various US-based bank statement PDFs and returning it in a standardized JSON format.
