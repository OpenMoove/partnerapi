# Python Examples

Complete Python implementations for the OpenMoove Partner API, including a full-featured SDK and practical examples.

## 🚀 Quick Start

### Install Dependencies
```bash
pip install requests python-dotenv
```

### Environment Setup
Create `.env` file:
```bash
OPENMOOVE_API_KEY=om_your_key_here
OPENMOOVE_ENVIRONMENT=staging  # or 'production'
```

### Basic Usage
```python
from openmoove_sdk import OpenMooveAPI

# Initialize client
api = OpenMooveAPI()

# Create property
result = api.create_client({
    'first_name': 'John',
    'last_name': 'Smith',
    'email': 'john.smith@example.com',
    'phone_number': '+447700900123',
    'property_address': '123 Example Street, London',
    'property_postcode': 'SW1A 1AA',
    'estate_agent_email': 'agent@youragency.com',
    'products': ['moove-ready-pack']
})

print(f"Created property: {result['property']['transactionId']}")
```

---

## 📦 Complete SDK Implementation

### openmoove_sdk.py
```python
"""
OpenMoove Partner API Python SDK

A complete Python client for the OpenMoove Partner API with comprehensive
error handling, retry logic, and convenience methods.
"""

import os
import time
import random
import logging
from typing import Dict, List, Optional, Any, Union
from dataclasses import dataclass
from datetime import datetime, timedelta
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


@dataclass
class APIConfig:
    """Configuration for OpenMoove API client"""
    api_key: str
    environment: str = 'staging'
    timeout: int = 30
    max_retries: int = 3
    backoff_factor: float = 0.3
    
    @property
    def base_url(self) -> str:
        """Get base URL for the environment"""
        if self.environment == 'production':
            return 'https://api.openmoove.com/api/v1/partners'
        return 'https://staging.openmoove.com/api/v1/partners'


class OpenMooveError(Exception):
    """Base exception for OpenMoove API errors"""
    pass


class ValidationError(OpenMooveError):
    """Raised when API returns validation errors"""
    def __init__(self, message: str, field_errors: Optional[Dict] = None):
        super().__init__(message)
        self.field_errors = field_errors or {}


class AuthenticationError(OpenMooveError):
    """Raised when API key is invalid"""
    pass


class RateLimitError(OpenMooveError):
    """Raised when rate limit is exceeded"""
    def __init__(self, retry_after: Optional[int] = None):
        super().__init__("Rate limit exceeded")
        self.retry_after = retry_after


class OpenMooveAPI:
    """
    OpenMoove Partner API Client
    
    Provides a complete interface to the OpenMoove Partner API with automatic
    retry logic, comprehensive error handling, and convenience methods.
    """
    
    def __init__(self, api_key: Optional[str] = None, environment: str = 'staging'):
        """
        Initialize OpenMoove API client
        
        Args:
            api_key: API key (if not provided, loaded from OPENMOOVE_API_KEY env var)
            environment: 'staging' or 'production'
        """
        self.config = APIConfig(
            api_key=api_key or os.getenv('OPENMOOVE_API_KEY'),
            environment=environment or os.getenv('OPENMOOVE_ENVIRONMENT', 'staging')
        )
        
        if not self.config.api_key:
            raise ValueError("API key is required. Set OPENMOOVE_API_KEY or pass api_key parameter")
        
        # Configure session with retry strategy
        self.session = requests.Session()
        self.session.headers.update({
            'X-API-KEY': self.config.api_key,
            'Content-Type': 'application/json',
            'User-Agent': 'OpenMoove-Python-SDK/1.0.0'
        })
        
        # Configure retry strategy
        retry_strategy = Retry(
            total=self.config.max_retries,
            backoff_factor=self.config.backoff_factor,
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=["HEAD", "GET", "PUT", "DELETE", "OPTIONS", "TRACE", "POST"]
        )
        
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)
        
        logger.info(f"Initialized OpenMoove API client for {self.config.environment}")
    
    def _make_request(self, method: str, endpoint: str, **kwargs) -> requests.Response:
        """Make HTTP request with error handling"""
        
        url = f"{self.config.base_url.rstrip('/')}/{endpoint.lstrip('/')}"
        
        try:
            response = self.session.request(
                method, 
                url, 
                timeout=self.config.timeout,
                **kwargs
            )
            
            # Handle specific status codes
            if response.status_code == 401:
                raise AuthenticationError("Invalid API key")
            elif response.status_code == 429:
                retry_after = int(response.headers.get('Retry-After', 60))
                raise RateLimitError(retry_after)
            elif response.status_code == 400:
                error_data = response.json()
                raise ValidationError(
                    self._format_validation_errors(error_data),
                    error_data
                )
            
            response.raise_for_status()
            return response
            
        except requests.exceptions.RequestException as e:
            logger.error(f"Request failed: {e}")
            raise OpenMooveError(f"Request failed: {e}")
    
    def _format_validation_errors(self, error_data: Dict) -> str:
        """Format validation errors for user display"""
        errors = []
        
        # Field-specific errors
        for field, field_errors in error_data.items():
            if field == 'non_field_errors':
                errors.extend(field_errors)
            elif isinstance(field_errors, list):
                errors.append(f"{field}: {'; '.join(field_errors)}")
        
        return "; ".join(errors) if errors else "Validation error"
    
    # Products API
    def list_products(self) -> List[Dict]:
        """
        List all available products
        
        Returns:
            List of product dictionaries
        """
        response = self._make_request('GET', '/products/')
        data = response.json()
        return data.get('results', [])
    
    # Clients & Properties API
    def create_client(self, client_data: Dict[str, Any]) -> Dict[str, Any]:
        """
        Create a new client and property
        
        Args:
            client_data: Dictionary containing client and property information
            
        Required fields:
            - first_name (str)
            - last_name (str)
            - email (str)
            - phone_number (str)
            - property_address (str)
            - property_postcode (str)
            - At least one of: estate_agent_email, conveyancer_email, 
              mortgage_broker_email, auctioneer_email
              
        Optional fields:
            - sale_method (str): 'standard', 'modern_auction', 'secure_sale'
            - products (list): Product slugs to purchase
            
        Returns:
            Dictionary containing property_id, mover_id, and full property details
            
        Raises:
            ValidationError: If required fields are missing or invalid
        """
        response = self._make_request('POST', '/clients/', json=client_data)
        return response.json()
    
    def create_standard_property(self, client_data: Dict[str, Any]) -> Dict[str, Any]:
        """
        Convenience method to create a standard property sale
        
        Args:
            client_data: Client information (sale_method will be set to 'standard')
            
        Returns:
            Property creation response
        """
        data = {**client_data, 'sale_method': 'standard'}
        if 'products' not in data:
            data['products'] = ['moove-ready-pack']
        
        return self.create_client(data)
    
    def create_auction_property(self, client_data: Dict[str, Any]) -> Dict[str, Any]:
        """
        Convenience method to create a modern auction property
        
        Args:
            client_data: Client information (must include auctioneer_email)
            
        Returns:
            Property creation response
            
        Raises:
            ValidationError: If auctioneer_email is missing
        """
        if 'auctioneer_email' not in client_data:
            raise ValidationError("auctioneer_email is required for auction properties")
        
        data = {**client_data, 'sale_method': 'modern_auction'}
        if 'products' not in data:
            data['products'] = ['moove-ready-pack']
        
        return self.create_client(data)
    
    # Properties API
    def list_properties(self, page: int = 1, page_size: int = 20) -> Dict[str, Any]:
        """
        List properties where you're assigned as a professional
        
        Args:
            page: Page number (1-based)
            page_size: Number of results per page (max 100)
            
        Returns:
            Dictionary with count, next, previous, and results
        """
        params = {'page': page, 'page_size': min(page_size, 100)}
        response = self._make_request('GET', '/properties/', params=params)
        return response.json()
    
    def get_property(self, transaction_id: str) -> Dict[str, Any]:
        """
        Get detailed property information
        
        Args:
            transaction_id: Property transaction UUID
            
        Returns:
            Property details with PDTF-compliant structure
        """
        response = self._make_request('GET', f'/properties/{transaction_id}/')
        return response.json()
    
    def get_all_properties(self) -> List[Dict[str, Any]]:
        """
        Get all properties (handles pagination automatically)
        
        Returns:
            List of all property dictionaries
        """
        all_properties = []
        page = 1
        
        while True:
            response = self.list_properties(page=page, page_size=100)
            all_properties.extend(response['results'])
            
            if not response.get('next'):
                break
            page += 1
        
        return all_properties
    
    # Milestones API
    def get_milestones(self, transaction_id: str) -> List[Dict[str, Any]]:
        """
        Get all milestones for a property
        
        Args:
            transaction_id: Property transaction UUID
            
        Returns:
            List of milestone dictionaries
        """
        response = self._make_request('GET', f'/properties/{transaction_id}/milestones/')
        return response.json()
    
    def get_current_milestone(self, transaction_id: str) -> Optional[Dict[str, Any]]:
        """
        Get the current active milestone for a property
        
        Args:
            transaction_id: Property transaction UUID
            
        Returns:
            Current milestone dictionary or None if all completed
        """
        try:
            response = self._make_request('GET', f'/properties/{transaction_id}/milestones/current/')
            return response.json()
        except OpenMooveError as e:
            if "not found" in str(e).lower():
                return None
            raise
    
    def get_milestone_progress(self, transaction_id: str) -> Dict[str, Any]:
        """
        Get milestone progress summary for a property
        
        Args:
            transaction_id: Property transaction UUID
            
        Returns:
            Dictionary with progress statistics
        """
        milestones = self.get_milestones(transaction_id)
        
        total = len(milestones)
        completed = sum(1 for m in milestones if m['completed'])
        overdue = sum(1 for m in milestones if m.get('is_overdue', False))
        
        current_milestone = None
        for milestone in milestones:
            if not milestone['completed']:
                current_milestone = milestone
                break
        
        return {
            'total_milestones': total,
            'completed_milestones': completed,
            'overdue_milestones': overdue,
            'completion_percentage': (completed / total * 100) if total > 0 else 0,
            'current_milestone': current_milestone
        }
    
    # Chat API (Future)
    def get_chat_history(self, transaction_id: str, page: int = 1, page_size: int = 20) -> Dict[str, Any]:
        """
        Get chat history for a property (currently returns mock data)
        
        Args:
            transaction_id: Property transaction UUID
            page: Page number
            page_size: Messages per page
            
        Returns:
            Chat history response
        """
        params = {'page': page, 'page_size': page_size}
        response = self._make_request('GET', f'/properties/{transaction_id}/chat/', params=params)
        return response.json()
    
    def get_chat_summary(self, transaction_id: str) -> Dict[str, Any]:
        """
        Get AI-generated chat summary for a property (currently returns mock data)
        
        Args:
            transaction_id: Property transaction UUID
            
        Returns:
            Chat summary response
        """
        response = self._make_request('GET', f'/properties/{transaction_id}/chat/summary/')
        return response.json()
    
    # Utility Methods
    def validate_client_data(self, client_data: Dict[str, Any]) -> List[str]:
        """
        Validate client data before API call
        
        Args:
            client_data: Client data dictionary
            
        Returns:
            List of validation error messages (empty if valid)
        """
        errors = []
        
        # Required fields
        required_fields = [
            'first_name', 'last_name', 'email', 'phone_number',
            'property_address', 'property_postcode'
        ]
        
        for field in required_fields:
            if not client_data.get(field):
                errors.append(f"Missing required field: {field}")
        
        # Professional email requirement
        professional_emails = [
            'estate_agent_email', 'conveyancer_email', 
            'mortgage_broker_email', 'auctioneer_email'
        ]
        
        if not any(client_data.get(email) for email in professional_emails):
            errors.append("At least one professional email is required")
        
        # Auction-specific validation
        sale_method = client_data.get('sale_method')
        if sale_method in ['modern_auction', 'secure_sale']:
            if not client_data.get('auctioneer_email'):
                errors.append("auctioneer_email is required for auction sale methods")
        
        return errors
    
    def health_check(self) -> bool:
        """
        Check if API is responding
        
        Returns:
            True if API is healthy, False otherwise
        """
        try:
            self.list_properties(page_size=1)
            return True
        except Exception as e:
            logger.error(f"Health check failed: {e}")
            return False


# Convenience Functions
def create_api_client() -> OpenMooveAPI:
    """Create API client with environment variables"""
    return OpenMooveAPI()


def quick_property_creation(
    first_name: str,
    last_name: str,
    email: str,
    phone_number: str,
    property_address: str,
    property_postcode: str,
    estate_agent_email: str,
    **kwargs
) -> Dict[str, Any]:
    """
    Quick property creation with minimal parameters
    
    Args:
        first_name: Client first name
        last_name: Client last name
        email: Client email
        phone_number: Client phone (international format)
        property_address: Property address
        property_postcode: Property postcode
        estate_agent_email: Estate agent email
        **kwargs: Additional fields (sale_method, products, etc.)
        
    Returns:
        Property creation response
    """
    api = create_api_client()
    
    client_data = {
        'first_name': first_name,
        'last_name': last_name,
        'email': email,
        'phone_number': phone_number,
        'property_address': property_address,
        'property_postcode': property_postcode,
        'estate_agent_email': estate_agent_email,
        **kwargs
    }
    
    return api.create_client(client_data)


if __name__ == "__main__":
    # Example usage
    api = OpenMooveAPI()
    
    print("OpenMoove API Client initialized")
    print(f"Environment: {api.config.environment}")
    print(f"Base URL: {api.config.base_url}")
    
    # Health check
    if api.health_check():
        print("✅ API is healthy")
    else:
        print("❌ API health check failed")
```

---

## 🎯 Practical Examples

### example_standard_property.py
```python
"""
Example: Create a standard property sale
"""

from openmoove_sdk import OpenMooveAPI
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def create_standard_property_example():
    """Create a standard property sale with full professional team"""
    
    # Initialize API client
    api = OpenMooveAPI()
    
    # Client data
    client_data = {
        'first_name': 'Alice',
        'last_name': 'Johnson',
        'email': 'alice.johnson@example.com',
        'phone_number': '+447700900111',
        'property_address': '15 Oakwood Drive, Leeds',
        'property_postcode': 'LS1 1AA',
        'sale_method': 'standard',
        'estate_agent_email': 'agent@premierproperties.com',
        'conveyancer_email': 'solicitor@legalservices.com',
        'mortgage_broker_email': 'advisor@mortgageexperts.com',
        'products': ['moove-ready-pack']
    }
    
    try:
        # Validate data before sending
        validation_errors = api.validate_client_data(client_data)
        if validation_errors:
            logger.error(f"Validation errors: {validation_errors}")
            return
        
        # Create property
        logger.info("Creating standard property...")
        result = api.create_standard_property(client_data)
        
        # Extract key information
        property_id = result['property_id']
        transaction_id = result['property']['transactionId']
        
        logger.info(f"✅ Property created successfully!")
        logger.info(f"Property ID: {property_id}")
        logger.info(f"Transaction ID: {transaction_id}")
        logger.info(f"Property Address: {result['property']['propertyPack']['address']['line1']}")
        
        # Get milestone progress
        logger.info("Checking milestone progress...")
        progress = api.get_milestone_progress(transaction_id)
        
        logger.info(f"Milestones: {progress['completed_milestones']}/{progress['total_milestones']} completed")
        logger.info(f"Progress: {progress['completion_percentage']:.1f}%")
        
        if progress['current_milestone']:
            current = progress['current_milestone']
            logger.info(f"Current milestone: {current['name']}")
            if current.get('due_date'):
                logger.info(f"Due date: {current['due_date']}")
        
        return result
        
    except Exception as e:
        logger.error(f"❌ Failed to create property: {e}")
        raise

if __name__ == "__main__":
    create_standard_property_example()
```

### example_auction_property.py
```python
"""
Example: Create a modern auction property
"""

from openmoove_sdk import OpenMooveAPI, ValidationError
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def create_auction_property_example():
    """Create a modern method auction property"""
    
    api = OpenMooveAPI()
    
    client_data = {
        'first_name': 'Robert',
        'last_name': 'Wilson',
        'email': 'robert.wilson@example.com',
        'phone_number': '+447700900222',
        'property_address': '42 Auction House Lane, Manchester',
        'property_postcode': 'M1 2AB',
        'estate_agent_email': 'agent@auctionproperties.com',
        'auctioneer_email': 'auctioneer@biddinghouse.com',
        'products': ['moove-ready-pack']
    }
    
    try:
        logger.info("Creating modern auction property...")
        result = api.create_auction_property(client_data)
        
        transaction_id = result['property']['transactionId']
        property_pack = result['property']['propertyPack']
        
        logger.info(f"✅ Auction property created!")
        logger.info(f"Transaction ID: {transaction_id}")
        logger.info(f"Sale Method: {property_pack['priceInformation']['disposal']}")
        logger.info(f"Address: {property_pack['address']['line1']}")
        
        # Check that auction-specific professionals are assigned
        if 'auctioneer_id' in result:
            logger.info(f"Auctioneer assigned: ID {result['auctioneer_id']}")
        
        # Get milestones (should be auction-specific)
        milestones = api.get_milestones(transaction_id)
        logger.info(f"Auction milestones created: {len(milestones)} total")
        
        for milestone in milestones[:3]:  # Show first 3
            status = "✅" if milestone['completed'] else "⏳"
            logger.info(f"  {status} {milestone['name']}")
        
        return result
        
    except ValidationError as e:
        logger.error(f"❌ Validation error: {e}")
        if hasattr(e, 'field_errors'):
            for field, errors in e.field_errors.items():
                logger.error(f"  {field}: {errors}")
    except Exception as e:
        logger.error(f"❌ Failed to create auction property: {e}")
        raise

if __name__ == "__main__":
    create_auction_property_example()
```

### example_bulk_import.py
```python
"""
Example: Bulk import properties from CSV or database
"""

import csv
import time
from typing import List, Dict
from openmoove_sdk import OpenMooveAPI, ValidationError
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def bulk_import_from_csv(csv_file: str, batch_size: int = 5) -> Dict:
    """
    Import properties from CSV file
    
    CSV should have columns: first_name, last_name, email, phone_number,
    property_address, property_postcode, estate_agent_email, sale_method
    """
    
    api = OpenMooveAPI()
    results = {'successful': 0, 'failed': 0, 'errors': []}
    
    logger.info(f"Starting bulk import from {csv_file}")
    
    with open(csv_file, 'r') as file:
        reader = csv.DictReader(file)
        properties = list(reader)
    
    logger.info(f"Found {len(properties)} properties to import")
    
    # Process in batches to respect rate limits
    for i in range(0, len(properties), batch_size):
        batch = properties[i:i + batch_size]
        logger.info(f"Processing batch {i//batch_size + 1}/{(len(properties)-1)//batch_size + 1}")
        
        for property_data in batch:
            try:
                # Add unique timestamp to email to prevent duplicates
                property_data['email'] = f"{property_data['email'].split('@')[0]}+{int(time.time())}@{property_data['email'].split('@')[1]}"
                
                # Add default product if not specified
                if 'products' not in property_data:
                    property_data['products'] = ['moove-ready-pack']
                
                # Create property
                result = api.create_client(property_data)
                results['successful'] += 1
                
                logger.info(f"✅ Created: {property_data['property_address']} - {result['property']['transactionId']}")
                
                # Brief pause to avoid rate limiting
                time.sleep(0.2)
                
            except ValidationError as e:
                results['failed'] += 1
                error_info = {
                    'property_address': property_data.get('property_address', 'Unknown'),
                    'error': str(e),
                    'field_errors': getattr(e, 'field_errors', {})
                }
                results['errors'].append(error_info)
                logger.error(f"❌ Failed: {property_data.get('property_address', 'Unknown')} - {e}")
                
            except Exception as e:
                results['failed'] += 1
                error_info = {
                    'property_address': property_data.get('property_address', 'Unknown'),
                    'error': str(e)
                }
                results['errors'].append(error_info)
                logger.error(f"❌ Unexpected error: {property_data.get('property_address', 'Unknown')} - {e}")
        
        # Pause between batches
        if i + batch_size < len(properties):
            logger.info("Pausing between batches...")
            time.sleep(2)
    
    # Summary
    logger.info(f"Bulk import completed!")
    logger.info(f"Successful: {results['successful']}")
    logger.info(f"Failed: {results['failed']}")
    
    if results['errors']:
        logger.info("Errors:")
        for error in results['errors'][:5]:  # Show first 5 errors
            logger.info(f"  {error['property_address']}: {error['error']}")
    
    return results

def create_sample_csv():
    """Create a sample CSV file for testing"""
    
    sample_data = [
        {
            'first_name': 'John',
            'last_name': 'Smith',
            'email': 'john.smith@example.com',
            'phone_number': '+447700900301',
            'property_address': '123 Sample Street, London',
            'property_postcode': 'SW1A 1AA',
            'estate_agent_email': 'agent@sampleagency.com',
            'sale_method': 'standard'
        },
        {
            'first_name': 'Jane',
            'last_name': 'Auction',
            'email': 'jane.auction@example.com',
            'phone_number': '+447700900302',
            'property_address': '456 Auction Avenue, Manchester',
            'property_postcode': 'M1 2AB',
            'estate_agent_email': 'agent@auctionhouse.com',
            'auctioneer_email': 'auctioneer@bidding.com',
            'sale_method': 'modern_auction'
        },
        {
            'first_name': 'Bob',
            'last_name': 'Complete',
            'email': 'bob.complete@example.com',
            'phone_number': '+447700900303',
            'property_address': '789 Complete Road, Birmingham',
            'property_postcode': 'B1 1AA',
            'estate_agent_email': 'agent@complete.com',
            'conveyancer_email': 'lawyer@legal.com',
            'mortgage_broker_email': 'broker@mortgages.com',
            'sale_method': 'standard'
        }
    ]
    
    with open('sample_properties.csv', 'w', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=sample_data[0].keys())
        writer.writeheader()
        writer.writerows(sample_data)
    
    logger.info("Created sample_properties.csv")

if __name__ == "__main__":
    # Create sample CSV
    create_sample_csv()
    
    # Import from CSV
    results = bulk_import_from_csv('sample_properties.csv')
    
    print(f"\nFinal Results:")
    print(f"Successful imports: {results['successful']}")
    print(f"Failed imports: {results['failed']}")
```

---

## 🔍 Monitoring Example

### example_monitoring.py
```python
"""
Example: Monitor multiple properties and track progress
"""

from openmoove_sdk import OpenMooveAPI
from typing import List, Dict
import time
from datetime import datetime, timedelta
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class PropertyMonitor:
    """Monitor multiple properties for progress and issues"""
    
    def __init__(self):
        self.api = OpenMooveAPI()
        self.properties = []
    
    def add_properties_to_monitor(self, transaction_ids: List[str]):
        """Add properties to monitoring list"""
        for transaction_id in transaction_ids:
            try:
                property_data = self.api.get_property(transaction_id)
                self.properties.append({
                    'transaction_id': transaction_id,
                    'address': property_data['propertyPack']['address']['line1'],
                    'status': property_data['status'],
                    'last_checked': datetime.now()
                })
                logger.info(f"Added to monitoring: {property_data['propertyPack']['address']['line1']}")
            except Exception as e:
                logger.error(f"Failed to add {transaction_id}: {e}")
    
    def check_all_properties(self) -> Dict:
        """Check all monitored properties for issues"""
        
        summary = {
            'total_properties': len(self.properties),
            'on_track': 0,
            'at_risk': 0,
            'overdue': 0,
            'issues': []
        }
        
        for property_info in self.properties:
            transaction_id = property_info['transaction_id']
            
            try:
                # Get milestone progress
                progress = self.api.get_milestone_progress(transaction_id)
                
                # Categorize property status
                if progress['overdue_milestones'] > 0:
                    summary['overdue'] += 1
                    summary['issues'].append({
                        'transaction_id': transaction_id,
                        'address': property_info['address'],
                        'type': 'overdue',
                        'overdue_count': progress['overdue_milestones'],
                        'completion_percentage': progress['completion_percentage']
                    })
                    
                elif progress['current_milestone']:
                    current = progress['current_milestone']
                    if current.get('due_date'):
                        due_date = datetime.fromisoformat(current['due_date'].replace('Z', '+00:00'))
                        days_until_due = (due_date - datetime.now()).days
                        
                        if days_until_due <= 2:  # At risk if due in 2 days
                            summary['at_risk'] += 1
                            summary['issues'].append({
                                'transaction_id': transaction_id,
                                'address': property_info['address'],
                                'type': 'at_risk',
                                'days_until_due': days_until_due,
                                'current_milestone': current['name']
                            })
                        else:
                            summary['on_track'] += 1
                else:
                    summary['on_track'] += 1
                
                # Update last checked
                property_info['last_checked'] = datetime.now()
                
            except Exception as e:
                logger.error(f"Failed to check {transaction_id}: {e}")
        
        return summary
    
    def generate_report(self) -> str:
        """Generate a monitoring report"""
        
        summary = self.check_all_properties()
        
        report = f"""
Property Monitoring Report - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
{'='*60}

Summary:
  Total Properties: {summary['total_properties']}
  On Track: {summary['on_track']} ({summary['on_track']/summary['total_properties']*100:.1f}%)
  At Risk: {summary['at_risk']} ({summary['at_risk']/summary['total_properties']*100:.1f}%)
  Overdue: {summary['overdue']} ({summary['overdue']/summary['total_properties']*100:.1f}%)

"""
        
        if summary['issues']:
            report += "Issues Requiring Attention:\n"
            report += "-" * 30 + "\n"
            
            for issue in summary['issues']:
                if issue['type'] == 'overdue':
                    report += f"❌ OVERDUE: {issue['address']}\n"
                    report += f"   {issue['overdue_count']} overdue milestones\n"
                    report += f"   Progress: {issue['completion_percentage']:.1f}%\n\n"
                    
                elif issue['type'] == 'at_risk':
                    report += f"⚠️  AT RISK: {issue['address']}\n"
                    report += f"   Current: {issue['current_milestone']}\n"
                    report += f"   Due in {issue['days_until_due']} days\n\n"
        else:
            report += "✅ All properties are on track!\n"
        
        return report
    
    def send_daily_report(self):
        """Generate and log daily report"""
        report = self.generate_report()
        logger.info(report)
        
        # Here you could email the report, send to Slack, etc.
        return report

def demo_monitoring():
    """Demonstrate property monitoring"""
    
    monitor = PropertyMonitor()
    api = OpenMooveAPI()
    
    # First, create some demo properties to monitor
    logger.info("Creating demo properties for monitoring...")
    
    demo_properties = [
        {
            'first_name': 'Monitor',
            'last_name': 'Property1',
            'email': 'monitor1@example.com',
            'phone_number': '+447700900401',
            'property_address': 'Monitor Property 1, London',
            'property_postcode': 'SW1A 1AA',
            'estate_agent_email': 'agent@monitor.com'
        },
        {
            'first_name': 'Monitor',
            'last_name': 'Property2',
            'email': 'monitor2@example.com',
            'phone_number': '+447700900402',
            'property_address': 'Monitor Property 2, Manchester',
            'property_postcode': 'M1 2AB',
            'estate_agent_email': 'agent@monitor.com',
            'sale_method': 'modern_auction',
            'auctioneer_email': 'auctioneer@monitor.com'
        }
    ]
    
    transaction_ids = []
    
    for property_data in demo_properties:
        try:
            result = api.create_client(property_data)
            transaction_ids.append(result['property']['transactionId'])
            logger.info(f"Created demo property: {result['property']['transactionId']}")
            time.sleep(1)  # Brief pause
        except Exception as e:
            logger.error(f"Failed to create demo property: {e}")
    
    if transaction_ids:
        # Add to monitoring
        monitor.add_properties_to_monitor(transaction_ids)
        
        # Generate report
        logger.info("Generating monitoring report...")
        report = monitor.send_daily_report()
        
        return monitor
    else:
        logger.error("No properties created for monitoring")

if __name__ == "__main__":
    demo_monitoring()
```

---

## 🚀 Installation & Setup

### requirements.txt
```
requests>=2.31.0
python-dotenv>=1.0.0
urllib3>=2.0.0
```

### Install Script
```bash
#!/bin/bash
# install.sh

echo "Installing OpenMoove Python SDK dependencies..."
pip install -r requirements.txt

echo "Creating .env template..."
cat > .env.example << 'EOF'
# OpenMoove API Configuration
OPENMOOVE_API_KEY=om_your_key_here
OPENMOOVE_ENVIRONMENT=staging

# Optional: Logging level
LOG_LEVEL=INFO
EOF

echo "Setup complete!"
echo "1. Copy .env.example to .env"
echo "2. Add your API key from staging.openmoove.com/profile/integrations/"
echo "3. Run examples: python example_standard_property.py"
```

---

## 🧪 Testing

### test_sdk.py
```python
"""
Tests for OpenMoove SDK
"""

import unittest
from unittest.mock import Mock, patch
from openmoove_sdk import OpenMooveAPI, ValidationError, AuthenticationError
import requests

class TestOpenMooveAPI(unittest.TestCase):
    
    def setUp(self):
        self.api = OpenMooveAPI(api_key='test_key', environment='staging')
    
    def test_init_without_api_key(self):
        """Test initialization without API key raises error"""
        with self.assertRaises(ValueError):
            OpenMooveAPI(api_key='')
    
    def test_validate_client_data_missing_fields(self):
        """Test validation catches missing required fields"""
        data = {'first_name': 'John'}
        errors = self.api.validate_client_data(data)
        
        self.assertGreater(len(errors), 0)
        self.assertTrue(any('last_name' in error for error in errors))
    
    def test_validate_client_data_missing_professional_email(self):
        """Test validation catches missing professional email"""
        data = {
            'first_name': 'John',
            'last_name': 'Smith',
            'email': 'john@test.com',
            'phone_number': '+447700900123',
            'property_address': '123 Test St',
            'property_postcode': 'SW1A 1AA'
        }
        errors = self.api.validate_client_data(data)
        
        self.assertTrue(any('professional email' in error for error in errors))
    
    def test_validate_auction_without_auctioneer(self):
        """Test auction validation requires auctioneer email"""
        data = {
            'first_name': 'John',
            'last_name': 'Smith',
            'email': 'john@test.com',
            'phone_number': '+447700900123',
            'property_address': '123 Test St',
            'property_postcode': 'SW1A 1AA',
            'sale_method': 'modern_auction',
            'estate_agent_email': 'agent@test.com'
        }
        errors = self.api.validate_client_data(data)
        
        self.assertTrue(any('auctioneer_email' in error for error in errors))
    
    @patch('requests.Session.request')
    def test_authentication_error(self, mock_request):
        """Test authentication error handling"""
        mock_response = Mock()
        mock_response.status_code = 401
        mock_response.raise_for_status.side_effect = requests.HTTPError()
        mock_request.return_value = mock_response
        
        with self.assertRaises(AuthenticationError):
            self.api.list_properties()
    
    @patch('requests.Session.request')
    def test_validation_error(self, mock_request):
        """Test validation error handling"""
        mock_response = Mock()
        mock_response.status_code = 400
        mock_response.json.return_value = {
            'first_name': ['This field is required.']
        }
        mock_request.return_value = mock_response
        
        with self.assertRaises(ValidationError) as cm:
            self.api.create_client({})
        
        self.assertIn('first_name', str(cm.exception))

if __name__ == '__main__':
    unittest.main()
```

Run tests:
```bash
python -m pytest test_sdk.py -v
```

---

## 🔗 Next Steps

- **[JavaScript Examples](../javascript/)** - Node.js and browser implementations
- **[cURL Examples](../curl/)** - Command-line testing
- **[Integration Patterns](../../guides/integration-patterns.md)** - Advanced workflows
- **[API Reference](../../api-reference/endpoints.md)** - Complete documentation