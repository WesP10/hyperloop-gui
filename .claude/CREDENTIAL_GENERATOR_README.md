# User Credential Generator - Quick Start

## Setup

1. **Install required packages** (if not already installed):
   ```bash
   pip install pandas openpyxl bcrypt
   ```

2. **Place your Excel file** in the workspace root:
   - Name it `users.xlsx` OR update `EXCEL_FILE` variable in `generate_users.py`
   - Ensure it has these columns: `NetID`, `Email`, `Name`, plus security questions

3. **Update email configuration** in `generate_users.py`:
   - Change `SENDER_EMAIL = "your-email@gmail.com"` to your actual Gmail address
   - Gmail App Password is already set: `bnux xbft wgrz loue`

## Excel Column Mapping

Expected columns:
- `NetID` → username (required)
- `Email` → user email (required)
- `Name` → full name (required)
- `Car you have/like?` → password ingredient
- `Pets Name` → password ingredient
- `Favorite City` → password ingredient
- `Favorite Number` → password ingredient
- `Worst Sub-team` → password ingredient
- `Coolest Invention` → password ingredient
- `Select a Symbol` → password ingredient

## Usage

```bash
python generate_users.py
```

The script will:
1. Read the Excel file
2. Generate memorable passwords (e.g., `Tesla-Buddy-Austin42!`)
3. Hash them with bcrypt
4. Save to `generated_users.json`
5. Ask if you want to send emails
6. Email credentials to each user

## Output

**generated_users.json** contains:
```json
{
  "MOCK_USERS": {
    "jsmith": {
      "username": "jsmith",
      "email": "jsmith@example.com",
      "full_name": "John Smith",
      "hashed_password": "$2b$12$..."
    }
  }
}
```

## Next Steps

1. Copy the `MOCK_USERS` dictionary from `generated_users.json`
2. Replace the `MOCK_USERS` in `cloud-services/src/auth/auth_service.py`
3. **DELETE** `generate_users.py`, `generated_users.json`, and your Excel file
   - These files are in `.gitignore` but delete them for safety

## Security Notes

- Passwords are readable but use personal info (memorable for users)
- Format: `Answer1-Answer2-Answer3##!` where ## is random 2-digit number
- All passwords are hashed with bcrypt (cost factor 12)
- Files are in `.gitignore` to prevent accidental commits
- **Always delete these files after use**

## Local Script Ran

'''
"""
Local script to generate user credentials from Excel spreadsheet.
WARNING: This file contains sensitive information and should NEVER be committed to version control.
Add to .gitignore immediately after use.
"""

import pandas as pd
import bcrypt
import json
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import random
import re
import argparse

# ===== CONFIGURATION =====
EXCEL_FILE = "example.xlsx"  # Change to your Excel file path
OUTPUT_JSON = "generated_users.json"  # Will contain MOCK_USERS format
SENDER_EMAIL = "example@gmail.com"  # Change to your Gmail address
GMAIL_APP_PASSWORD = "example"
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
TEST_EMAIL = "example@gmail.com"  # Test mode sends here

# ===== PASSWORD GENERATION =====
def clean_answer(answer):
    """Clean security question answer for password use."""
    if pd.isna(answer) or answer == "":
        return None
    # Remove special characters, keep alphanumeric
    cleaned = re.sub(r'[^a-zA-Z0-9]', '', str(answer))
    return cleaned if cleaned else None

def generate_password(row):
    """
    Generate memorable password from security questions.
    Format: Answer1SymbolAnswer2SymbolNumberSymbol (e.g., Tesla$Buddy$42$)
    """
    # Get user's selected symbol
    symbol = row.get('Select a Symbol', '$')
    if pd.isna(symbol) or symbol == "":
        symbol = '$'  # Default fallback
    else:
        symbol = str(symbol).strip()
    
    # Collect all available security answers (excluding the symbol field)
    answers = []
    
    fields = [
        'example1',
        'example2'
    ]
    
    for field in fields:
        if field in row:
            cleaned = clean_answer(row[field])
            if cleaned:
                answers.append(cleaned)
    
    # Select 2-3 random answers
    if len(answers) >= 3:
        selected = random.sample(answers, 3)
    elif len(answers) >= 2:
        selected = random.sample(answers, 2)
    elif len(answers) >= 1:
        selected = answers
    else:
        # Fallback: use name + random number
        name = clean_answer(row.get('Name', 'User'))
        selected = [name, str(random.randint(1000, 9999))]
    
    # Add random number suffix for security
    suffix = str(random.randint(10, 99))
    
    # Join with user's symbol and add suffix with symbol
    password = symbol.join(selected) + suffix + symbol
    
    return password

# ===== MAIN PROCESSING =====
def main():
    # Parse command-line arguments
    parser = argparse.ArgumentParser(description='Generate user credentials from Excel')
    parser.add_argument('--prod', action='store_true', 
                       help='Production mode: send emails to all users. Without this flag, only test email is sent.')
    args = parser.parse_args()
    
    is_production = args.prod
    
    print("=" * 60)
    print("User Credential Generator")
    print(f"Mode: {'PRODUCTION' if is_production else 'TEST (dry-run)'}")
    print("=" * 60)
    
    # Read Excel file
    print(f"\n[1/5] Reading Excel file: {EXCEL_FILE}")
    try:
        df = pd.read_excel(EXCEL_FILE)
        print(f"✓ Found {len(df)} users")
    except FileNotFoundError:
        print(f"✗ Error: File '{EXCEL_FILE}' not found!")
        print("Please update EXCEL_FILE path in the script.")
        return
    except Exception as e:
        print(f"✗ Error reading Excel: {e}")
        return
    
    # Display column names for verification
    print(f"\nColumns found: {list(df.columns)}")
    
    # Validate required columns
    required_cols = ['NetID', 'Email', 'Name']
    missing_cols = [col for col in required_cols if col not in df.columns]
    if missing_cols:
        print(f"\n✗ Error: Missing required columns: {missing_cols}")
        return
    
    # Generate users
    print(f"\n[2/5] Generating passwords and hashing...")
    users_dict = {}
    credentials_list = []  # For emailing
    
    for idx, row in df.iterrows():
        netid = str(row['NetID']).strip()
        email = str(row['Email']).strip()
        full_name = str(row['Name']).strip()
        
        if not netid or netid == 'nan':
            print(f"  ⚠ Skipping row {idx}: No NetID")
            continue
        
        # Generate password
        plain_password = generate_password(row)
        
        # Hash password with bcrypt (cost factor 12, default)
        hashed_password = bcrypt.hashpw(
            plain_password.encode('utf-8'), 
            bcrypt.gensalt()
        ).decode('utf-8')
        
        # Create user entry
        users_dict[netid] = {
            "username": netid,
            "email": email,
            "full_name": full_name,
            "hashed_password": hashed_password,
        }
        
        # Store credentials for emailing
        credentials_list.append({
            "username": netid,
            "password": plain_password,
            "email": email,
            "full_name": full_name
        })
        
        print(f"  ✓ {netid}: {plain_password}")
    
    print(f"\n✓ Generated {len(users_dict)} users")
    
    # Save to JSON
    print(f"\n[3/5] Saving to {OUTPUT_JSON}...")
    output_data = {
        "MOCK_USERS": users_dict,
        "note": "Copy the MOCK_USERS dictionary into auth_service.py"
    }
    
    with open(OUTPUT_JSON, 'w') as f:
        json.dump(output_data, f, indent=4)
    
    print(f"✓ Saved to {OUTPUT_JSON}")
    
    # Send emails
    print(f"\n[4/5] Sending emails...")
    
    if is_production:
        # Production mode: send to all users
        send_confirmation = input("PRODUCTION MODE: Send emails to ALL users? (yes/no): ").strip().lower()
        
        if send_confirmation == 'yes':
            if SENDER_EMAIL == "your-email@gmail.com":
                print("✗ Error: Please update SENDER_EMAIL in the script configuration!")
                return
            
            success_count = 0
            for cred in credentials_list:
                try:
                    send_credentials_email(
                        cred['email'],
                        cred['username'],
                        cred['password'],
                        cred['full_name']
                    )
                    print(f"  ✓ Sent to {cred['email']}")
                    success_count += 1
                except Exception as e:
                    print(f"  ✗ Failed to send to {cred['email']}: {e}")
            
            print(f"\n✓ Sent {success_count}/{len(credentials_list)} emails")
        else:
            print("Skipped email sending.")
    else:
        # Test mode: send mock email to test address
        print("TEST MODE: Sending sample email to test address...")
        if SENDER_EMAIL == "your-email@gmail.com":
            print("✗ Error: Please update SENDER_EMAIL in the script configuration!")
            return
        
        try:
            # Use first user's credentials as example
            if credentials_list:
                sample = credentials_list[0]
                send_test_email(
                    TEST_EMAIL,
                    sample['username'],
                    sample['password'],
                    sample['full_name'],
                    len(credentials_list)
                )
                print(f"  ✓ Test email sent to {TEST_EMAIL}")
                print(f"  Sample credentials: {sample['username']} / {sample['password']}")
            else:
                print("  ✗ No users to send test email")
        except Exception as e:
            print(f"  ✗ Failed to send test email: {e}")
    
    # Final instructions
    print(f"\n[5/5] Next Steps:")
    if is_production:
        print(f"  1. Review {OUTPUT_JSON}")
        print(f"  2. Copy MOCK_USERS content to cloud-services/src/auth/auth_service.py")
        print(f"  3. DELETE this script and {OUTPUT_JSON} (or add to .gitignore)")
        print(f"  4. Verify users can log in")
    else:
        print(f"  1. Review {OUTPUT_JSON} to verify output")
        print(f"  2. Check test email at {TEST_EMAIL}")
        print(f"  3. Run with --prod flag when ready for production")
    print("\n" + "=" * 60)

# ===== EMAIL SENDING =====
def send_test_email(to_email, sample_username, sample_password, sample_name, total_users):
    """Send test email with sample credentials."""
    
    # Create message
    msg = MIMEMultipart('alternative')
    msg['Subject'] = '[TEST] User Credential Generation Complete'
    msg['From'] = SENDER_EMAIL
    msg['To'] = to_email
    
    # Email body
    text = f"""
TEST MODE - Credential Generation Complete

Generated {total_users} user accounts successfully.

Sample credentials (first user):
Username: {sample_username}
Password: {sample_password}
Full Name: {sample_name}

Review the generated_users.json file and run with --prod flag to send emails to all users.
"""
    
    html = f"""
<html>
  <body>
    <h2 style="color: #ff9800;">TEST MODE - Credential Generation Complete</h2>
    
    <p>Generated <strong>{total_users}</strong> user accounts successfully.</p>
    
    <h3>Sample credentials (first user):</h3>
    <table style="border: 1px solid #ddd; padding: 10px; margin: 10px 0;">
      <tr>
        <td style="padding: 5px;"><strong>Username:</strong></td>
        <td style="padding: 5px;"><code>{sample_username}</code></td>
      </tr>
      <tr>
        <td style="padding: 5px;"><strong>Password:</strong></td>
        <td style="padding: 5px;"><code>{sample_password}</code></td>
      </tr>
      <tr>
        <td style="padding: 5px;"><strong>Full Name:</strong></td>
        <td style="padding: 5px;">{sample_name}</td>
      </tr>
    </table>
    
    <p>Review the <code>generated_users.json</code> file and run with <code>--prod</code> flag to send emails to all users.</p>
  </body>
</html>
"""
    
    # Attach both plain text and HTML versions
    part1 = MIMEText(text, 'plain')
    part2 = MIMEText(html, 'html')
    msg.attach(part1)
    msg.attach(part2)
    
    # Send email
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(SENDER_EMAIL, GMAIL_APP_PASSWORD)
        server.send_message(msg)

def send_credentials_email(to_email, username, password, full_name):
    """Send credentials email to a user."""
    
    # Create message
    msg = MIMEMultipart('alternative')
    msg['Subject'] = 'Your Account Credentials'
    msg['From'] = SENDER_EMAIL
    msg['To'] = to_email
    
    # Email body
    text = f"""
Hi {full_name},

Your account has been created! Here are your login credentials:

Username: {username}
Password: {password}

Please keep these credentials secure and change your password after your first login.

If you have any questions, please contact the administrator.

Best regards,
The Team
"""
    
    html = f"""
<html>
  <body>
    <p>Hi {full_name},</p>
    
    <p>Your account has been created! Here are your login credentials:</p>
    
    <table style="border: 1px solid #ddd; padding: 10px; margin: 10px 0;">
      <tr>
        <td style="padding: 5px;"><strong>Username:</strong></td>
        <td style="padding: 5px;"><code>{username}</code></td>
      </tr>
      <tr>
        <td style="padding: 5px;"><strong>Password:</strong></td>
        <td style="padding: 5px;"><code>{password}</code></td>
      </tr>
    </table>
    
    <p>Please keep these credentials secure and change your password after your first login.</p>
    
    <p>If you have any questions, please contact the administrator.</p>
    
    <p>Best regards,<br>The Team</p>
  </body>
</html>
"""
    
    # Attach both plain text and HTML versions
    part1 = MIMEText(text, 'plain')
    part2 = MIMEText(html, 'html')
    msg.attach(part1)
    msg.attach(part2)
    
    # Send email
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(SENDER_EMAIL, GMAIL_APP_PASSWORD)
        server.send_message(msg)

if __name__ == "__main__":
    main()

'''