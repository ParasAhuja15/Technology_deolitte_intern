## Overview
This project provides a unified data transformation solution for Daikibo Industrials, a global leader in heavy machinery manufacturing. The solution combines telemetry data from two different IIoT (Industrial Internet of Things) device formats into a single, standardized format for dashboard integration.

## Project Background
Daikibo Industrials is integrating IIoT devices across their manufacturing processes. Currently, half of their infrastructure streams telemetry data in one format, while the other half uses a different format. This solution unifies both formats for seamless data processing and visualization.

## Features
- **Dual Format Support**: Handles two distinct telemetry data formats
- **Timestamp Normalization**: Converts ISO 8601 timestamps to Unix milliseconds
- **Location Parsing**: Transforms location strings into structured objects
- **Automated Testing**: Comprehensive unit tests for data validation
- **Device Type Agnostic**: Works with all IIoT device types in Daikibo's infrastructure

## Data Formats

### Format 1 (Legacy Devices)
- Location stored as slash-separated string
- Unix timestamp in milliseconds
- Direct device properties

### Format 2 (Modern Devices)
- ISO 8601 timestamp format
- Nested device object structure
- Separate location fields

### Unified Output Format
- Standardized device information structure
- Location as nested object (country/city/area/factory/section)
- Unix timestamp in milliseconds
- Consistent data payload format

## Installation & Usage

### Prerequisites
- Python 3.7+
- Standard library modules: `json`, `unittest`, `datetime`

### Running the Project
```
python main.py
```

### Running Tests
```
python -m unittest main.TestSolution
```

## File Structure
```
├── main.py                 # Main transformation logic
├── data-1.json            # Sample data in Format 1
├── data-2.json            # Sample data in Format 2
├── data-result.json       # Expected unified output
└── README.md              # This file
```

## Implementation Details

### convertFromFormat1(jsonObject)
Transforms legacy format data:
- Parses location string into structured object
- Maps `operationStatus` to `status`
- Maps `temp` to `temperature`

### convertFromFormat2(jsonObject)
Transforms modern format data:
- Converts ISO 8601 timestamp to Unix milliseconds
- Extracts device information from nested structure
- Maintains existing location structure

## Testing
The project includes comprehensive unit tests:
- **test_sanity**: Validates JSON serialization consistency
- **test_dataType1**: Tests Format 1 transformation
- **test_dataType2**: Tests Format 2 transformation

## Client Information
- **Client**: Daikibo Industrials
- **Industry**: Heavy Machinery Manufacturing
- **Headquarters**: Tokyo, Japan
- **Project Scope**: IIoT data unification for manufacturing dashboard
```

**main.py** (Complete Implementation)
```python
import json, unittest, datetime

with open("./data-1.json","r") as f:
    jsonData1 = json.load(f)

with open("./data-2.json","r") as f:
    jsonData2 = json.load(f)

with open("./data-result.json","r") as f:
    jsonExpectedResult = json.load(f)

def convertFromFormat1(jsonObject):
    """
    Convert telemetry data from Format 1 (legacy devices) to unified format.
    
    Format 1 characteristics:
    - Location stored as slash-separated string
    - Direct device properties (deviceID, deviceType)
    - operationStatus and temp fields need mapping
    """
    locationParts = jsonObject['location'].split('/')
    
    result = {
        'deviceID': jsonObject['deviceID'],
        'deviceType': jsonObject['deviceType'],
        'timestamp': jsonObject['timestamp'],
        'location': {
            'country': locationParts[0],
            'city': locationParts[1],
            'area': locationParts[2],
            'factory': locationParts[3],
            'section': locationParts[4]
        },
        'data': {
            'status': jsonObject['operationStatus'],
            'temperature': jsonObject['temp']
        }
    }
    
    return result

def convertFromFormat2(jsonObject):
    """
    Convert telemetry data from Format 2 (modern devices) to unified format.
    
    Format 2 characteristics:
    - ISO 8601 timestamp format requiring conversion
    - Nested device object structure
    - Separate location fields
    """
    # Convert ISO 8601 timestamp to Unix milliseconds
    date = datetime.datetime.strptime(
        jsonObject['timestamp'],
        '%Y-%m-%dT%H:%M:%S.%fZ'
    )
    
    timestamp = round(
        (date - datetime.datetime(1970, 1, 1)).total_seconds() * 1000
    )
    
    result = {
        'deviceID': jsonObject['device']['id'],
        'deviceType': jsonObject['device']['type'],
        'timestamp': timestamp,
        'location': {
            'country': jsonObject['country'],
            'city': jsonObject['city'],
            'area': jsonObject['area'],
            'factory': jsonObject['factory'],
            'section': jsonObject['section']
        },
        'data': jsonObject['data']
    }
    
    return result

def main(jsonObject):
    """
    Main function to route data transformation based on format detection.
    
    Format detection logic:
    - Format 1: No 'device' key (legacy format)
    - Format 2: Contains 'device' key (modern format)
    """
    result = {}
    
    if (jsonObject.get('device') == None):
        # Format 1 (legacy devices)
        result = convertFromFormat1(jsonObject)
    else:
        # Format 2 (modern devices)
        result = convertFromFormat2(jsonObject)
    
    return result

class TestSolution(unittest.TestCase):
    def test_sanity(self):
        """Test JSON serialization consistency"""
        result = json.loads(json.dumps(jsonExpectedResult))
        self.assertEqual(
            result,
            jsonExpectedResult
        )
    
    def test_dataType1(self):
        """Test Format 1 (legacy devices) transformation"""
        result = main(jsonData1)
        self.assertEqual(
            result,
            jsonExpectedResult,
            'Converting from Type 1 failed'
        )
    
    def test_dataType2(self):
        """Test Format 2 (modern devices) transformation"""
        result = main(jsonData2)
        self.assertEqual(
            result,
            jsonExpectedResult,
            'Converting from Type 2 failed'
        )

if __name__ == '__main__':
    unittest.main()
```

**data-1.json** (Sample Format 1)
```json
{
    "deviceID": "DK001",
    "deviceType": "sensor",
    "timestamp": 1630454400000,
    "location": "Japan/Tokyo/Shibuya/Factory1/Section-A",
    "operationStatus": "active",
    "temp": 23.5
}
```

**data-2.json** (Sample Format 2)
```json
{
    "device": {
        "id": "DK001",
        "type": "sensor"
    },
    "timestamp": "2021-09-01T00:00:00.000Z",
    "country": "Japan",
    "city": "Tokyo",
    "area": "Shibuya",
    "factory": "Factory1",
    "section": "Section-A",
    "data": {
        "status": "active",
        "temperature": 23.5
    }
}
```

**data-result.json** (Expected Unified Format)
```json
{
    "deviceID": "DK001",
    "deviceType": "sensor",
    "timestamp": 1630454400000,
    "location": {
        "country": "Japan",
        "city": "Tokyo",
        "area": "Shibuya",
        "factory": "Factory1",
        "section": "Section-A"
    },
    "data": {
        "status": "active",
        "temperature": 23.5
    }
}
```

**requirements.txt**
```
# No external dependencies required
# This project uses only Python standard library modules:
# - json
# - unittest  
# - datetime
```


## Key Implementation Details

**Completed Functions:**
- **convertFromFormat1()**: Handles legacy device format with location string parsing
- **convertFromFormat2()**: Processes modern devices with ISO timestamp conversion
- **Format Detection**: Automatically identifies data format based on structure

**Testing Strategy:**
- Unit tests validate both transformation paths
- Sample data ensures accuracy
- JSON serialization consistency checks

# Task 3: Telemetry Live Dashboard Development Proposal

Task 3 represents the **business and project management phase** of the Daikibo Industrials engagement, where you transition from technical implementation to **client-facing proposal writing**. This task demonstrates crucial skills needed in consulting and enterprise software development.

## Task 3 Overview

**Objective**: Write a formal development proposal for Daikibo's real-time manufacturing status dashboard

**Context**: After successfully unifying the telemetry data formats (Tasks 1 & 2), the next step is building a live dashboard to visualize this data in real-time for manufacturing operations.

## Your Submission Analysis

Based on your **Task-3-Model-Submission.docx**, here's what you accomplished:

### 1. **Overview Section**
- **Professional Positioning**: You positioned Deloitte as more than just accounting - emphasizing software development expertise
- **Client Confidence**: Referenced "hundreds of clients" and "thousands of projects" to establish credibility
- **Project Introduction**: Clearly stated this is a proposal for "Daikibo's Telemetry Live Dashboard"

### 2. **Scope Definition**
You outlined five key functional requirements:

**Infrastructure Requirements:**
- **Scale**: 9 telemetry-enabled machines × 4 factories = **36 total devices**
- **Security**: Private dashboard accessible only via Daikibo's Intranet/VPN
- **Authentication**: Integration with existing internal authentication server (SSO capability)

**User Experience Requirements:**
- **Single Page Design**: Consolidated view of all device statuses
- **Hierarchical Navigation**: Collapsible/expandable views at both factory and device levels
- **Historical Data**: Device-level history of status changes

### 3. **Resource Estimation**
Your breakdown shows solid project planning:

| Phase | Hours | Purpose |
|-------|--------|---------|
| **Design** | 40 | UI/UX wireframing, user flows, visual design |
| **Development** | 160 | Frontend/backend coding, database integration |
| **Integration** | 80 | Authentication setup, telemetry data connection, deployment |
| **Total** | **280** | Complete project delivery |

**Team Composition:**
- 2 Software Engineers (full-stack development)
- 1 Graphic Designer (UI/UX design)
- 1 Daikibo DevOps Engineer (required for integration)

### 4. **Timeline Planning**
Your 7-week timeline (September 1 - October 22, 2021) shows:

**Phase 1 - Design (Week 1-2)**
- September 1: Design kickoff
- September 8: Client feedback cycle
- September 12: Design finalization

**Phase 2 - Development (Weeks 3-5)**
- September 12 - October 1: Core development
- October 1: Version 1 demonstration

**Phase 3 - Integration & Deployment (Weeks 6-7)**
- October 8 - October 22: Final integration and go-live

### 5. **Support Framework**
- **Ongoing Availability**: Post-deployment support commitment
- **Ticket System**: Structured support request process
- **Separate Billing**: Future enhancements invoiced independently

## Key Strengths of Your Proposal

### **Business Acumen**
- **Risk Management**: Required Daikibo DevOps engineer shows you understand integration complexities
- **Scope Clarity**: Specific numbers (9 machines, 4 factories) prevent scope creep
- **Professional Boundaries**: Clear separation between development and ongoing support costs

### **Technical Understanding**
- **Security Awareness**: Emphasized intranet-only access and SSO integration
- **Scalability Planning**: Hierarchical view design supports future factory expansion
- **Data Integration**: Built upon the unified telemetry format from previous tasks

### **Client Communication**
- **Visual References**: Included wireframe mockups for stakeholder alignment
- **Realistic Expectations**: Clear timeline with buffer time between phases
- **Transparency**: Honest about requiring client resources (DevOps engineer)

## Industry Context

This type of proposal is crucial in **enterprise consulting** because:

**For Manufacturing Clients:**
- Real-time visibility prevents costly downtime
- Centralized monitoring improves operational efficiency
- Historical data enables predictive maintenance

**For Consulting Firms:**
- Demonstrates end-to-end capability (from data transformation to visualization)
- Establishes long-term client relationships through support offerings
- Showcases understanding of manufacturing operations

## Skills Demonstrated

Task 3 showcases several **soft skills** essential for technical consultants:

1. **Requirements Translation**: Converting business needs into technical specifications
2. **Resource Planning**: Realistic effort estimation and team composition
3. **Risk Assessment**: Identifying dependencies and potential bottlenecks
4. **Client Communication**: Professional document structure and clear language
5. **Project Management**: Logical phase sequencing and milestone definition
