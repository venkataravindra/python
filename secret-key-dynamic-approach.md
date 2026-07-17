# **Making SECRET_KEY Dynamic - Multiple Approaches**

Here are several ways to make your SECRET_KEY dynamic instead of hardcoded:

---

## **Method 1: Environment Variables (Recommended for Production)**

### **Updated Code:**
```python
import bcrypt
import jwt
import os
import secrets
from datetime import datetime, timedelta, timezone

# Dynamic SECRET_KEY from environment variable with fallback
def get_secret_key():
    secret_key = os.getenv('JWT_SECRET_KEY')
    if not secret_key:
        # Generate a new secret key if not found in environment
        secret_key = secrets.token_hex(32)
        print(f"⚠️  Warning: Generated new SECRET_KEY: {secret_key}")
        print("💡 Set JWT_SECRET_KEY environment variable for production!")
    return secret_key

SECRET_KEY = get_secret_key()
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Rest of your functions remain the same
def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, hashed: str) -> bool:
    return bcrypt.checkpw(plain.encode(), hashed.encode())

def create_access_token(username: str) -> str:
    payload = {
        "sub": username,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
```

### **Setting Environment Variable:**

#### **Windows:**
```cmd
# Command Prompt
set JWT_SECRET_KEY=your_secret_key_here

# PowerShell
$env:JWT_SECRET_KEY="your_secret_key_here"

# Permanent (System Properties > Environment Variables)
setx JWT_SECRET_KEY "your_secret_key_here"
```

#### **Linux/Mac:**
```bash
# Temporary
export JWT_SECRET_KEY="your_secret_key_here"

# Permanent (add to ~/.bashrc or ~/.zshrc)
echo 'export JWT_SECRET_KEY="your_secret_key_here"' >> ~/.bashrc
source ~/.bashrc
```

---

## **Method 2: Configuration File Approach**

### **Create `config.py`:**
```python
import os
import secrets
import json
from pathlib import Path

class Config:
    def __init__(self):
        self.config_file = Path("app_config.json")
        self.load_config()
    
    def load_config(self):
        """Load configuration from file or create new one"""
        if self.config_file.exists():
            with open(self.config_file, 'r') as f:
                config_data = json.load(f)
                self.SECRET_KEY = config_data.get('SECRET_KEY')
        else:
            self.SECRET_KEY = None
        
        # Generate new key if not found
        if not self.SECRET_KEY:
            self.SECRET_KEY = secrets.token_hex(32)
            self.save_config()
    
    def save_config(self):
        """Save configuration to file"""
        config_data = {
            'SECRET_KEY': self.SECRET_KEY,
            'ALGORITHM': 'HS256',
            'ACCESS_TOKEN_EXPIRE_MINUTES': 30
        }
        with open(self.config_file, 'w') as f:
            json.dump(config_data, f, indent=2)
        
        # Set restrictive permissions (Unix-like systems)
        if hasattr(os, 'chmod'):
            os.chmod(self.config_file, 0o600)

# Create global config instance
config = Config()
```

### **Updated Main Code:**
```python
import bcrypt
import jwt
from datetime import datetime, timedelta, timezone
from config import config

# Use dynamic SECRET_KEY from config
SECRET_KEY = config.SECRET_KEY
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Your existing functions remain the same
def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, hashed: str) -> bool:
    return bcrypt.checkpw(plain.encode(), hashed.encode())

def create_access_token(username: str) -> str:
    payload = {
        "sub": username,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
```

---

## **Method 3: Database Storage (Advanced)**

### **Database Approach:**
```python
import bcrypt
import jwt
import secrets
import sqlite3
from datetime import datetime, timedelta, timezone
from pathlib import Path

class SecretKeyManager:
    def __init__(self, db_path="app_secrets.db"):
        self.db_path = db_path
        self.init_database()
        self.SECRET_KEY = self.get_or_create_secret_key()
    
    def init_database(self):
        """Initialize the secrets database"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS app_secrets (
                id INTEGER PRIMARY KEY,
                key_name TEXT UNIQUE,
                key_value TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        conn.commit()
        conn.close()
        
        # Set restrictive permissions
        if hasattr(os, 'chmod'):
            os.chmod(self.db_path, 0o600)
    
    def get_or_create_secret_key(self):
        """Get existing secret key or create new one"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # Try to get existing key
        cursor.execute("SELECT key_value FROM app_secrets WHERE key_name = ?", ("JWT_SECRET_KEY",))
        result = cursor.fetchone()
        
        if result:
            secret_key = result[0]
        else:
            # Create new secret key
            secret_key = secrets.token_hex(32)
            cursor.execute(
                "INSERT INTO app_secrets (key_name, key_value) VALUES (?, ?)",
                ("JWT_SECRET_KEY", secret_key)
            )
            conn.commit()
            print(f"🔑 Generated new SECRET_KEY and stored in database")
        
        conn.close()
        return secret_key
    
    def rotate_secret_key(self):
        """Generate and store a new secret key"""
        new_key = secrets.token_hex(32)
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute(
            "UPDATE app_secrets SET key_value = ?, updated_at = CURRENT_TIMESTAMP WHERE key_name = ?",
            (new_key, "JWT_SECRET_KEY")
        )
        conn.commit()
        conn.close()
        self.SECRET_KEY = new_key
        return new_key

# Initialize secret key manager
key_manager = SecretKeyManager()
SECRET_KEY = key_manager.SECRET_KEY
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Your existing functions
def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, hashed: str) -> bool:
    return bcrypt.checkpw(plain.encode(), hashed.encode())

def create_access_token(username: str) -> str:
    payload = {
        "sub": username,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

# Function to rotate secret key when needed
def rotate_secret_key():
    """Rotate the secret key (invalidates all existing tokens)"""
    global SECRET_KEY
    SECRET_KEY = key_manager.rotate_secret_key()
    print("🔄 Secret key rotated. All existing tokens are now invalid.")
    return SECRET_KEY
```

---

## **Method 4: Using .env File (Popular with FastAPI/Flask)**

### **Install python-dotenv:**
```bash
pip install python-dotenv
```

### **Create `.env` file:**
```env
# .env file (add to .gitignore!)
JWT_SECRET_KEY=d244bab78d771f55ae8aa86a2b2767fd1a94534b15659c2783dac5cff86e3242
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

### **Updated Code:**
```python
import bcrypt
import jwt
import secrets
import os
from datetime import datetime, timedelta, timezone
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

def get_secret_key():
    """Get secret key from environment or generate new one"""
    secret_key = os.getenv('JWT_SECRET_KEY')
    if not secret_key:
        secret_key = secrets.token_hex(32)
        print(f"⚠️  Generated new SECRET_KEY: {secret_key}")
        print("💡 Add this to your .env file: JWT_SECRET_KEY=" + secret_key)
    return secret_key

SECRET_KEY = get_secret_key()
ALGORITHM = os.getenv('JWT_ALGORITHM', 'HS256')
ACCESS_TOKEN_EXPIRE_MINUTES = int(os.getenv('ACCESS_TOKEN_EXPIRE_MINUTES', 30))

# Your existing functions remain the same
def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, hashed: str) -> bool:
    return bcrypt.checkpw(plain.encode(), hashed.encode())

def create_access_token(username: str) -> str:
    payload = {
        "sub": username,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
```

---

## **Method 5: Runtime Generation with Persistence**

### **Simple Runtime Generation:**
```python
import bcrypt
import jwt
import secrets
import os
from datetime import datetime, timedelta, timezone
from pathlib import Path

class DynamicSecretKey:
    def __init__(self):
        self.key_file = Path(".secret_key")
        self.SECRET_KEY = self.load_or_generate_key()
    
    def load_or_generate_key(self):
        """Load existing key or generate new one"""
        if self.key_file.exists():
            try:
                with open(self.key_file, 'r') as f:
                    key = f.read().strip()
                    if len(key) == 64:  # Valid hex key length
                        return key
            except Exception as e:
                print(f"Error reading key file: {e}")
        
        # Generate new key
        new_key = secrets.token_hex(32)
        self.save_key(new_key)
        return new_key
    
    def save_key(self, key):
        """Save key to file with proper permissions"""
        with open(self.key_file, 'w') as f:
            f.write(key)
        
        # Set restrictive permissions (Unix-like systems)
        if hasattr(os, 'chmod'):
            os.chmod(self.key_file, 0o600)
        
        print(f"🔑 Secret key saved to {self.key_file}")
    
    def regenerate_key(self):
        """Generate a new secret key"""
        self.SECRET_KEY = secrets.token_hex(32)
        self.save_key(self.SECRET_KEY)
        return self.SECRET_KEY

# Initialize dynamic secret key
secret_manager = DynamicSecretKey()
SECRET_KEY = secret_manager.SECRET_KEY
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Your existing functions
def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, hashed: str) -> bool:
    return bcrypt.checkpw(plain.encode(), hashed.encode())

def create_access_token(username: str) -> str:
    payload = {
        "sub": username,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

# Utility function to regenerate key
def regenerate_secret_key():
    """Regenerate secret key (invalidates all tokens)"""
    global SECRET_KEY
    SECRET_KEY = secret_manager.regenerate_key()
    print("🔄 Secret key regenerated!")
    return SECRET_KEY
```

---

## **Method 6: Cloud-based Secret Management**

### **AWS Secrets Manager Example:**
```python
import boto3
import json
import secrets
from botocore.exceptions import ClientError

class AWSSecretManager:
    def __init__(self, secret_name="myapp/jwt-secret", region="us-east-1"):
        self.secret_name = secret_name
        self.region = region
        self.client = boto3.client('secretsmanager', region_name=region)
        self.SECRET_KEY = self.get_or_create_secret()
    
    def get_or_create_secret(self):
        try:
            # Try to get existing secret
            response = self.client.get_secret_value(SecretId=self.secret_name)
            secret_data = json.loads(response['SecretString'])
            return secret_data['jwt_secret_key']
        except ClientError as e:
            if e.response['Error']['Code'] == 'ResourceNotFoundException':
                # Create new secret
                return self.create_new_secret()
            else:
                raise e
    
    def create_new_secret(self):
        new_key = secrets.token_hex(32)
        secret_data = {
            'jwt_secret_key': new_key,
            'algorithm': 'HS256'
        }
        
        self.client.create_secret(
            Name=self.secret_name,
            SecretString=json.dumps(secret_data),
            Description='JWT Secret Key for MyApp'
        )
        
        print(f"🔑 Created new secret in AWS Secrets Manager")
        return new_key

# Usage (requires AWS credentials configured)
# aws_secret_manager = AWSSecretManager()
# SECRET_KEY = aws_secret_manager.SECRET_KEY
```

---

## **Comparison of Methods**

| Method | Security | Complexity | Best For |
|--------|----------|------------|----------|
| Environment Variables | ⭐⭐⭐⭐ | ⭐⭐ | Production, Docker |
| Config File | ⭐⭐⭐ | ⭐⭐ | Development, Small apps |
| Database Storage | ⭐⭐⭐⭐ | ⭐⭐⭐ | Enterprise apps |
| .env File | ⭐⭐⭐⭐ | ⭐⭐ | Development, FastAPI |
| Runtime Generation | ⭐⭐⭐ | ⭐⭐ | Quick setup |
| Cloud Secrets | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Production, Scalable |

---

## **Security Best Practices**

### **🔒 Important Security Notes:**

1. **Never commit secrets to version control**
   ```bash
   # Add to .gitignore
   .env
